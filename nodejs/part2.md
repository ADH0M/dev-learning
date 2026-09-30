# Node.js

In 2009, a developer named **Ryan Dahl** introduced Node.js with a simple but powerful idea:

> What if JavaScript could run outside the browser?

Instead of limiting JavaScript to webpages, **Node.js** allowed JavaScript to **run directly on the machine**. This meant JavaScript could interact with system resources such as files, networks, and processes.

Suddenly, JavaScript was no longer just a browser language. It could now be used to build servers, backend applications, and scalable systems.

---

## What Exactly Is Node.js?

At this point, it’s natural to ask:

> What exactly is Node.js?

Node.js is **not a programming language**, and it is also **not a framework**.

Instead, Node.js is a **JavaScript runtime environment** that allows JavaScript code to run outside the browser.

A runtime environment is simply a system that provides everything needed to execute a program. In the case of Node.js, it provides JavaScript with the ability to interact with the operating system.

This means JavaScript programs can now perform tasks such as:

- Reading and writing files
- Creating web servers
- Opening network connections
- Running command-line tools

All of these capabilities which were not available to JavaScript inside the browser.

But this raises another interesting question.

If JavaScript was originally designed for browsers, **how does Node.js actually make it work on the server?**

To answer that, we need to look at **Node.js architecture**.

---

## Understanding Node.js Architecture

To understand how Node.js enables JavaScript to run on the server, we need to look at what is happening under the hood.

At a high level, Node.js is built on three important components:

- The **V8 JavaScript Engine**
- **C++ bindings**
- **libuv**

Each of these components plays a specific role in allowing JavaScript to interact with the system.

- **V8** is responsible for executing JavaScript code.
- C++ bindings act as a bridge between JavaScript and the native C/C++ code used by Node.js and libuv to interact with the operating system.
- **libuv** handles asynchronous operations such as file system access, networking, and the event loop.

Together, these components allow Node.js to run JavaScript programs while also interacting with the operating system.

A simplified view of Node.js architecture looks like this:

![](https://cdn.hashnode.com/uploads/covers/695355c61aa33c7fda042da1/70b46655-8a60-48f8-a552-212dc25fe187.png align="center")

Now that we have a high-level view of Node.js architecture, let’s start by understanding the first and most fundamental component: the **V8 JavaScript Engine**.
