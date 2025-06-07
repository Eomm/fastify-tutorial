---
title: Markdown page example
---

# Learn Fastify with Test

This book will teach you Fastify from the ground up, following an hands on test-driven approach. This means we are going to write a lot of tests and use them to drive the exploration of Fastify's features. This writing is heavily inspired by [Learn Go with Tests](https://quii.gitbook.io/learn-go-with-tests)

We are going to use javascript and not typescript through this book. This doesn't affect the concept that we are going to learn, they are applicable to both languages. You can gladly follow along this book writing your own code in typescript. Keep in mind that in some context a little typescript gymnastics will be required to make the provided examples work.

To take full appreciation of this book you need to be familiar with javascript and Node.js. Some topics like javascript modules, async/await, promises and dynamic imports are not covered in this book but they are a requirement. You should also be familiar with what a automated unit test is and why testing code is important.

We are going to cover:

- Basics of Fastify
- Routing
- Error Handling and validation
- Logging
- Hooks
- Plugins
- Rendering

## Requirements

First of all we need to install Node.js. We encurage to install the latest version, or at least version 22.
You can use [brew](https://formulae.brew.sh/formula/node) or [nvm](https://github.com/nvm-sh/nvm) or [volta](https://volta.sh/).

Having done that we have to create an empty directory and then initialize a new npm project with `npm init -y`.
It's necessary since we are going to install fastify using npm.

<a href="/docs/hello-world" >Let's go</a>
