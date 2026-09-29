# What will you learn?

- In this guide, you'll learn how to:

  - Containerize and run a Node.js application using Docker.

  - Set up a local development environment using containers.

  - Run tests inside a Docker container.
  
  - Use pnpm as package manager

Start by containerizing a Node.js application.

***

## Main Concept : Multi-stage Build

بدلاً من وضع كل شيء في Image واحد (مما يجعله ضخماً لأنه سيحتوي على أكواد الـ TypeScript، وأدوات التطوير devDependencies، وغيرها)، قمنا بتقسيم العملية إلى 3 مراحل:

1- builder: لبناء الكود (TypeScript to JavaScript).
2- deps: لتحميل الـ Dependencies الخاصة بالـ Production فقط.
3- runner: الـ Image النهائي الذي سيتم تشغيله (يحتوي فقط على ما يلزم للتشغيل).

## builder Stage

> الهدف هنا: تحويل كود الـ TypeScript إلى JavaScript.

- تحديد مجلد /app كمجلد العمل الأساسي داخل الـ Container.

```dockerfile
    FROM node:22-alpine AS builder

    WORKDIR /app
```

- تفعيل corepack (أداة مدمجة في Node.js) لتجهيز وتفعيل مدير الحزم pnpm بأحدث إصدار.

```dockerfile
RUN corepack enable && corepack prepare pnpm@latest --activate
```

- Use BuildKit from Docker:
  - --mount=type=cache: عمل Cache لمجلد الـ pnpm-store لتسريع عملية تحميل الـ Packages في المرات القادمة.
  - --mount=type=bind: نحن نقوم بـ "ربط" ملفات الـ package.json و lock و workspace فقط داخل الـ Container مؤقتاً أثناء الـ Install.
  - الفائدة: إذا قمت بتعديل كود الـ Source Code ولم تعدل الـ Packages، فإن Docker لن يعيد تحميل الـ Packages من جديد (لأن ملفات الـ JSON لم تتغير)، مما يوفر وقتاً هائلاً.
    --frozen-lockfile: يضمن تثبيت النسخ المطابقة تماماً للـ Lock file بدون أي تعديلات.

```dockerfile
    RUN --mount=type=cache,target=/root/.pnpm-store \
    --mount=type=bind,source=package.json,target=/app/package.json \
    --mount=type=bind,source=pnpm-lock.yaml,target=/app/pnpm-lock.yaml \
    --mount=type=bind,source=pnpm-workspace.yaml,target=/app/pnpm-workspace.yaml \
    pnpm install --frozen-lockfile
```

- الشرح: الآن ننسخ باقي ملفات المشروع (الكود المصدري)، ونقوم بتشغيل أمر البناء (pnpm run build) والذي سيقوم بتحويل الـ TypeScript إلى JavaScript وإخراجها في مجلد dist.

```dockerfile
COPY . .
RUN pnpm run build
```

---

## Dependencies Stage

> الهدف هنا: تحميل الـ Dependencies الخاصة بالـ Production فقط (بدون أدوات التطوير مثل TypeScript, Jest, etc).

- نبدأ مرحلة جديدة تماماً وصورة جديدة نظيفة.

- نفس منطق الـ Cache والـ Bind Mounts لتسريع العملية، ولكن مع إضافة علم --prod. هذا العلم يخبر pnpm بتحميل الـ dependencies فقط وتجاهل الـ devDependencies.

```dockerfile

FROM node:22-alpine AS deps

WORKDIR /app

RUN corepack enable && corepack prepare pnpm@latest --activate

RUN --mount=type=cache,target=/root/.pnpm-store \
    --mount=type=bind,source=package.json,target=/app/package.json \
    --mount=type=bind,source=pnpm-lock.yaml,target=/app/pnpm-lock.yaml \
    --mount=type=bind,source=pnpm-workspace.yaml,target=/app/pnpm-workspace.yaml \
    pnpm install --frozen-lockfile --prod

```

---

## Runner Stage

مرحلة التشغيل - الـ Image النهائي

> الهدف هنا: بناء الـ Image النهائي الذي سيتم رفعه وتشغيله في الـ Production. سيكون صغيراً جداً وآمناً.

- صورة جديدة ونظيفة. نضبط متغير البيئة NODE_ENV إلى production (مهم جداً لـ Express و Node لتحسين الأداء والأمان).

```dockerfile
FROM node:22-alpine AS runner
WORKDIR /app
ENV NODE_ENV=production

```

- نحن لا ننسخ كل المشروع! بل ننسخ فقط:
  - مجلد node_modules من مرحلة deps (والذي يحتوي على الـ Production packages فقط).
  - مجلد dist من مرحلة builder (والذي يحتوي على كود الـ JavaScript المترجم).
  - ملف package.json.

> --chown=node:node: يغير ملكية الملفات لتكون للمستخدم node بدلاً من root (لأمان أعلى).

- النتيجة: الـ Source Code الأصلي (TypeScript)، وأدوات التطوير، والـ devDependencies لم يتم نسخها للـ Image النهائي، مما جعل حجمه صغيراً جداً!

```dockerfile
COPY --from=deps --chown=node:node /app/node_modules ./node_modules
COPY --from=builder --chown=node:node /app/dist ./dist
COPY --from=builder --chown=node:node /app/package.json ./package.json

```

- أمان عالي 🔒. بدلاً من تشغيل التطبيق بصلاحيات root (وهو الخطأ الشائع)، نقوم بالتبديل إلى المستخدم node المحدد مسبقاً في صورة Alpine. إذا تم اختراق التطبيق، لن يمتلك المهاجم صلاحيات الـ Root في الـ Container.

- EXPOSE 3000: إخبار Docker أن التطبيق سيستمع على البورت 3000 (لأغراض التوثيق والشبكات).

- CMD: الأمر الذي سيتم تنفيذه عند تشغيل الـ Container (تشغيل ملف الـ JavaScript الرئيسي).

```dockerfile
USER node

EXPOSE 3000

CMD ["node", "dist/index.js"]
```

---

🏆 ملخص لماذا هذا الـ Dockerfile ممتاز جداً (تضيفها في ملاحظاتك):

- حجم صغير جداً (Small Size): بسبب الـ Multi-stage والـ Alpine، الـ Image النهائي لا يحتوي إلا على الـ JS المترجم والـ Prod Dependencies.

- سرعة بناء خيالية (Fast Builds): بسبب استخدام --mount=type=cache و bind لملفات الـ pnpm.

- أمان عالي (Secure): بسبب تشغيل التطبيق بـ USER node (Non-root user).

- استخدام pnpm: والذي يعتبر أسرع ويوفر في المساحة مقارنة بـ npm أو yarn.
