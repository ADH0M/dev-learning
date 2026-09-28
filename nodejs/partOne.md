# How Node.js Works Under the Hood: V8, libuv and C++ Bindings Explained

Did you know? When JavaScript was first introduced, it was designed to run **only inside web browsers**.

Every browser contains a JavaScript engine responsible for executing JavaScript code.  
For example, Google Chrome uses the **V8 JavaScript Engine** to parse and run JavaScript programs.

Inside the browser environment, JavaScript has access to features related to the webpage, such as:

- Manipulating the DOM
- Responding to user events
- Making network requests using browser APIs
                                  
However, browser environments intentionally restrict JavaScript from interacting directly with the operating system.

Because of these limitations, developers traditionally relied on backend languages like **Java, PHP, C# or Python** to build servers.

But today, JavaScript is no longer limited to the browser. Developers use it to build :

- backend applications.
- real-time systems.
- command-line tools
- large-scale APIs.
                       
So how did JavaScript suddenly gain the ability to create servers, read files, and interact with the operating system?

The answer lies in **Node.js**.

Let’s see how Node.js made this possible.


* * *

## Enter Node.js

In 2009, a developer named **Ryan Dahl** introduced Node.js with a simple but powerful idea:

> What if JavaScript could run outside the browser?

Instead of limiting JavaScript to webpages, **Node.js** allowed JavaScript to **run directly on the machine**. This meant JavaScript could interact with:

- system resources such as files
- networks
- processes.
                         
Suddenly, JavaScript was no longer just a browser language. It could now be used to :

- build servers
- backend applications
- scalable systems.

## What Exactly Is Node.js?

At this point, it’s natural to ask:

> What exactly is Node.js?

Node.js is **not a programming language**, and it is also **not a framework**.

Instead, Node.js is a **JavaScript runtime environment** that allows JavaScript code to run outside the browser.

A **runtime environment** is simply a system that provides everything needed to execute a program. In the case of **Node.js**, it provides JavaScript with the ability to interact with the operating system.

This means JavaScript programs can now perform tasks such as:

* Reading and writing files
* Creating web servers
* Opening network connections
* Running command-line tools

All of these capabilities which were not available to JavaScript inside the browser.

But this raises another interesting question.

If JavaScript was originally designed for browsers, **how does Node.js actually make it work on the server?**

To answer that, we need to look at **Node.js architecture**.
