# functions

## **What Is Function ? **

<div dir="rtl">

- تخيلها كأنها "مصنع صغير" جوه قاعدة البيانات بتاعتك. بتاخد مدخلات (Inputs)، بتشتغل عليهم (ممكن عمليات حسابية، استعلامات معقدة، إلخ)، وبتطلعلك نتيجة (Output). أحسن حاجة فيها إنك بتكتب الكود مرة واحدة، وبتحفظه جوه الـ Database، وتقدر تناديه (Call) من أي مكان في تطبيقك أو حتى من جوه استعلامات SQL تانية.

**الفرق الجوهري بين الـ Function والـ Stored Procedure:**
_(ملاحظة: الـ Procedures ظهرت في PostgreSQL من النسخة 11، وقبل كده كان كل حاجة Functions)_

1. **القيمة المرجعة (Return Value):**
   - **Function**:

     > **لازم** ترجع قيمة (حتى لو كانت قيمة فاضية `VOID` أو جدول فاضي).

   - **Procedure:**
     > مش شرط ترجع أي قيمة.

2. **الاستخدام في الـ SQL:**
   - **Function:**
     > تقدر تناديها جوه استعلام `SELECT` عادي
     >
     > > `SELECT calculate_tax(100);`
   - **Procedure:**
     > مينفعش تحطها جوه `SELECT`. بتتنادي بأمر خاص اسمه `CALL`.
     >
     > > `CALL update_inventory();`

3. **التحكم في الـ Transactions (أهم فرق عملي):**
   - **Function:**
     > **ممنوع** تكتب جواها `COMMIT` أو `ROLLBACK`. هي بتشتغل تحت مظلة الـ Transaction اللي ناداها.
   - **Procedure:**
     > **مسموح** تكتب جواها `COMMIT` و `ROLLBACK` عشان تتحكم في الـ Transactions بنفسها.

---

### 🟢 النقطة 2: الهيكل الأساسي (Syntax) وشرح الـ `$$`

عشان تعمل Function، بنستخدم الأمر `CREATE OR REPLACE FUNCTION`.
ليه `OR REPLACE`؟ عشان لو عملت تعديل في الكود، وتعمل Run تاني، هو هيعمل تحديث للـ Function الموجودة من غير ما يرمي خطأ، وده بيوفر عليك إنك تعمل `DROP` (حذف) للـ Function القديمة الأول.

**شكل الهيكل الأساسي :**

```sql
CREATE OR REPLACE FUNCTION function_name(parameter_name datatype)
RETURNS return_datatype AS
$$
BEGIN

    -- هنا هتكتب الأكواد بتاعتك
    -- (Body)

    RETURN some_value;
END;
$$
LANGUAGE plpgsql;
```

**🔍 شرح الـ `$$` (Dollar Quoting) - ركز معايا هنا:**
في PostgreSQL، الـ `$$` (أو `$body$` أو `$func$`) هي عبارة عن **علامات تنصيص مخصصة للـ Strings**.

**ليه بنستخدمها في الـ Functions ؟**

> عشان الكود اللي جوه الـ Function هو في الأصل عبارة عن `String` كبير بيتخزن في الـ Database.

> لو استخدمنا علامات التنصيص العادية `' '`، هنضطر نعمل Escape لكل علامة تنصيص جوه الكود، وده هيخلي الكود شكله وحش جداً ومقروءش.

**مثال يوضح المشكلة والحل:**

> لو عايز تكتب جملة SQL جوه الـ Function بتقول: `SELECT 'Hello'`

- **لو استخدمنا تنصيص عادي:** `'SELECT ''Hello'''`
  > (شكله وحش ومقروءش).
- **لو استخدمنا `$$`:** `$$SELECT 'Hello'$$`
  > (نظيف ومقروء جداً).

💡 **نصيحة برمجية:** بدل ما تكتب `$$` بس، الأفضل تكتب `$body$` في البداية و `$body$` في النهاية. ده بيسهل عليك القراءة، ومهم جداً لو جيت تعمل Function جوه Function (Nested Functions).

```sql
CREATE OR REPLACE FUNCTION my_first_function()
RETURNS TEXT AS
$body$
BEGIN
    RETURN 'Hello PostgreSQL!';
END;
$body$
LANGUAGE plpgsql; -- هنا بنحدد إن اللغة اللي بنكتب بيها هي PL/pgSQL
```

ممتاز! يلا بينا نكمل المشوار، وهندخل دلوقتي في "مخزن الأدوات" بتاع الـ Function (المتغيرات) وإزاي ندخل ونطلع منها البيانات.

---

## 🟢 النقطة 3: المتغيرات (Variables) وقسم الـ `DECLARE`

قبل ما نبدأ نكتب الكود الفعلي (جوه الـ `BEGIN`)، محتاجين أحياناً نحفظ بيانات مؤقتة عشان نستخدمها. هنا بييجي دور قسم الـ `DECLARE` اللي بييجي **قبل** الـ `BEGIN`.

**طريقة كتابة المتغيرات:**

```sql
DECLARE
    -- اسم_المتغير  نوع_البيانات;
    user_age INTEGER;
    user_name TEXT;

    -- ممكن نديله قيمة افتراضية (Default Value) باستخدام :=
    counter INTEGER := 0;
    is_active BOOLEAN := TRUE;
```

> بدل ما تكتب نوع البيانات يدوي، تقدر تخلي المتغير ياخد نفس نوع بيانات عمود في جدول!
> ده بيحميك من الأخطاء لو غيرت نوع العمود في الجدول.

- **`%TYPE`**:

```sql

DECLARE my_user_name users.username%TYPE; --ياخد نفس نوع عمود
```

- نفس نوع عمود username في جدول users

- **`%ROWTYPE`**:
  > ياخد شكل "صف كامل" (Row) من جدول معين.
  ```sql
  DECLARE full_user users%ROWTYPE; -- متغير يقدر يخزن صف كامل من جدول users
  ```

---

## 🟢 النقطة 4: المدخلات والمخرجات (Parameters & Returns)

إزاي الـ Function بتتكلم مع العالم الخارجي؟

### أولاً: المدخلات (Parameters)

لما بتعرف Function، بتحدد اللي بيدخلها:

```sql
CREATE FUNCTION calculate_tax(price NUMERIC, tax_rate NUMERIC) ...
```

هنا `price` و `tax_rate` هما مدخلات من نوع `IN` (وهو النوع الافتراضي).

لكن في PostgreSQL عندنا 3 أنواع للمدخلات:

1. **`IN`**: (الافتراضي) بياخد قيمة من اللي نادى الـ Function، ومش بيقدر يغيرها ويرجعها.
2. **`OUT`**: بيحدد إن الـ Function هترجع قيمة من نوع معين، وبيتم التعامل معاه كأنه متغير جوه الـ Function تقدر تكتب فيه.
3. **`INOUT`**: بياخد قيمة من بره، وممكن تتعدل جوه الـ Function وترجع تاني.

#### ثانياً: المخرجات (RETURNS)

بعد كلمة `RETURNS` بنحدد شكل اللي هيطلع من الـ Function:

1. **قيمة مفردة (Scalar):** زي `INTEGER`, `TEXT`, `NUMERIC`.
   ```sql
   RETURNS INTEGER
   ```
2. **مفيش مخرجات (VOID):** لو الـ Function بتعمل تحديث (Update/Insert) ومش هترجع حاجة.
   ```sql
   RETURNS VOID
   ```
3. **مجموعة صفوف (SETOF):** لو عايز الـ Function ترجع جدول كامل (أو مجموعة صفوف من جدول موجود).
   ```sql
   RETURNS SETOF users -- هترجع كل صفوف جدول users
   ```
4. **جدول مخصص (TABLE):** لو عايز ترجع أعمدة معينة أنت محددها.
   ```sql
   RETURNS TABLE(user_id INTEGER, full_name TEXT)
   ```

---

### 💡 مثال يجمع النقطة 3 و 4 معاً:

تخيل عايزين Function تاخد `user_id`، وتجيب اسم المستخدم وعمره، وترجعهم كـ TABLE.

```sql
CREATE OR REPLACE FUNCTION get_user_details(p_user_id INTEGER)
RETURNS TABLE(user_name TEXT, user_age INTEGER) AS
$body$
DECLARE
    -- متغير مؤقت نحفظ فيه الـ ID عشان نستخدمه (مثال بسيط)
    v_target_id INTEGER := p_user_id;
BEGIN
    -- (هنشرح إزاي نكتب الكود الجوا في الدرس الجاي)
    RETURN QUERY
    SELECT u.username, u.age
    FROM users u
    WHERE u.id = v_target_id;
END;
$body$
LANGUAGE plpgsql;
```

_(لاحظ استخدام `RETURN QUERY` لأنه بيرجع `TABLE`، وده هنشرحه بالتفصيل في الدرس الجاي)._

---

###

ممتاز! يلا بينا ندخل دلوقتي في "قلب" الـ Function، وهو المكان اللي بيحصل فيه الشغل الفعلي.

### 🟢 النقطة 5: لغة PL/pgSQL (كتابة الكود جوه `BEGIN ... END;`)

لما بتكتب جوه الـ `BEGIN` و `END;`، أنت مش بتكتب SQL عادي، أنت بتكتب بلغة **PL/pgSQL**. في لغة SQL العادية، لو كتبت `SELECT age FROM users;` النتيجة هتظهر على الشاشة. لكن جوه الـ Function، **ممنوع تسيب أي نتيجة تطلع في الهواء**، لازم تحطها في مكان (متغير) أو ترجعها.

إليك أهم 3 أدوات هتستخدمها جوه الـ `BEGIN`:

#### 1. تعيين القيم (Assignment)

عشان تحط قيمة في متغير، بنستخدم الرمز `:=` (أو `=` في النسخ الجديدة، لكن `:=` هو الأشهر والأضمن في PL/pgSQL).

```sql
v_total := v_price + v_tax;
v_message := 'Hello ' || v_name; -- دمج النصوص باستخدام ||
```

#### 2. جلب البيانات من الجداول (`SELECT ... INTO`)

دي **أهم نقطة** ومبتدئين كتير بيقعوا فيها! لو عايز تجيب قيمة من جدول وتحطها في متغير، لازم تستخدم كلمة `INTO`.

```sql
DECLARE
    v_user_age INTEGER;
    v_user_name TEXT;
BEGIN
    -- نجيب العمر والاسم ونحطهم في المتغيرات
    SELECT age, username
    INTO v_user_age, v_user_name
    FROM users
    WHERE id = 1;

    -- لو عايز نجيب صف كامل (استخدام %ROWTYPE اللي شرحناه قبل كده)
    -- SELECT * INTO v_full_user FROM users WHERE id = 1;
END;
```

⚠️ **تحذير:** لو كتبت `SELECT` عادي جوه الـ Function من غير `INTO` أو `RETURN`، الـ PostgreSQL هيرمي خطأ اسمه: `query has no destination for result data`.

#### 3. تنفيذ أوامر بدون إرجاع (`PERFORM`)

أحياناً بتعمل `UPDATE`، `DELETE`، أو حتى `SELECT` عشان تتأكد من وجود بيانات، لكنك **مش عايز ترجع النتيجة دي**. هنا بنستخدم `PERFORM` بدل `SELECT`.

```sql
BEGIN
    -- مثال: عايز أعمل Update ومش هرجع حاجة
    UPDATE users SET last_login = NOW() WHERE id = 1; -- الـ UPDATE مش محتاج PERFORM

    -- مثال: عايز أعمل SELECT عشان أتأكد إن المستخدم موجود، بس مش هرجع البيانات
    PERFORM 1 FROM users WHERE id = 1 AND is_active = TRUE;

    -- مثال: نداء Function تانية مش هرجع قيمتها
    PERFORM send_welcome_email(v_user_name);
END;
```

---

### 💡 مثال عملي يجمع كل اللي اتعلمناه (النقاط 2، 3، 4، 5):

Function بتاخد `user_id` و `new_age`، بتعمل Update للعمر، وبتحسب عدد السنوات اللي هتضاف للعمر الحالي، وبترجع رسالة نصية.

```sql
CREATE OR REPLACE FUNCTION update_user_age(p_user_id INTEGER, p_new_age INTEGER)
RETURNS TEXT AS
$body$
DECLARE
    v_current_age INTEGER;
    v_diff INTEGER;
    v_result_message TEXT;
BEGIN
    -- 1. نجيب العمر الحالي ونحطه في المتغير (SELECT INTO)
    SELECT age INTO v_current_age
    FROM users
    WHERE id = p_user_id;

    -- 2. لو المستخدم مش موجود، نرجع رسالة (Early Return)
    IF v_current_age IS NULL THEN
        RETURN 'Error: User not found!';
    END IF;

    -- 3. نعمل الحسابات (Assignment)
    v_diff := p_new_age - v_current_age;

    -- 4. نعمل الـ Update في الجدول (مش محتاج PERFORM لأن مفيش SELECT)
    UPDATE users SET age = p_new_age WHERE id = p_user_id;

    -- 5. نجهز الرسالة ونرجعها
    v_result_message := 'Age updated successfully. Difference is: ' || v_diff || ' years.';
    RETURN v_result_message;
END;
$body$
LANGUAGE plpgsql;
```

---

</div>
