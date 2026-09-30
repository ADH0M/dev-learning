# 1️⃣ V8 Engine – Executing JavaScript

At the core of Node.js lies the **V8 JavaScript Engine**.

V8 is an open-source JavaScript engine developed by Google and written primarily in **C++**. It was originally created to run JavaScript inside the Chrome browser, but Node.js uses the same engine to execute JavaScript outside the browser.

In simple terms, V8 is responsible for **parsing and executing JavaScript code**.

However, computers cannot understand JavaScript directly. They only understand **machine code** — the low-level instructions executed by the CPU.

So when a JavaScript program runs in Node.js, the V8 engine converts that JavaScript code into machine code that the computer can execute.

Now you might wonder: **How does the V8 engine execute JavaScript?** Let's see.

---

## How V8 Executes JavaScript

When you run a JavaScript program, the code goes through several stages inside the V8 engine.

**1\. Parsing**

First, V8 reads the JavaScript source code and parses it into a structure called an **Abstract Syntax Tree (AST)**.

The AST represents the structure of the program and helps the engine understand how different pieces of code are related.

For example, the expression:

```javascript
let sum = 5 + 3;
```

can be represented conceptually as:

```plaintext
      =
     / \
   sum   +
        / \
       5   3
```

This structured representation allows V8 to analyze the program.

**2\. Bytecode Generation**

Once the AST is created, V8 converts it into **bytecode**.

Bytecode is a lower-level representation of the program that can be executed more efficiently than raw JavaScript.

This step is handled by V8's interpreter called **Ignition**.

```plaintext
JavaScript
    ↓
   AST
    ↓
 Bytecode
```

At this point, the program can already start running.
