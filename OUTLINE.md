# Tutorial outline

This document outlines the structure of the tutorial.

## Overview

It this section we must introduce Fastify by explaining:

- What it is
- Why it has been created
  - Key features
- The benchmarks
- {history} When it has been created
- {history} Who created it

### Project Principles

We should mention the design principles of Fastify and the philosophy behind it:

- SOLID principles
- Everything that can be a plugin, should be a plugin (and not added to the core)
- Do you need it? You build it.

### This Tutorial

Explain the purpose of this tutorial, how it is structured, and what the reader can expect to learn.

**Project Objectives**:

- Learn the Fastify Web Framework
- Async Resource Integration (eg: Database Integration with PostgreSQL)
- Data Validation and Sanitization
- Read incoming requests
- Error Handling
- OpenTelemetry integration

List the creators of the tutorial and their roles.

## Foundamentals

### Installation

Set up the environment, build an empty folder and install Fastify.

### Hello World

Create a simple Fastify server that responds with "Hello World" to HTTP GET requests.
Test the route with `node:test` and `inject` to ensure it works correctly.

### Hello World for production

Same as the hello world, but with the shape of a typical Fastify application for production use.
It must set the timeouts: https://backend.cafe/handling-http-timeouts-in-fastify

### Routing

Explain the different types of routes in Fastify:

- Static routes
  - Generic route definition
- Path parameters
- 404 Not Found
- Asynchronous vs Synchronous routes (include a simple table to compare them)

### The plugin system

Overview of the plugin system in Fastify.
Let's isolate some routes in different plugins with different `prefix` values.

We must explain the boot sequence and how the plugins are loaded in Fastify.
Within the `prefix` example, we must mention the option parameter.

### Decorators

Using the encapsulation, we can explain how to use decorators in Fastify.

#### Request/Reply Decorators

Same as the previous section, but for request and reply decorators.

#### Plain object Decorators

Here we must explain how to use plain object decorators in Fastify to avoid
the reference issue (blocked by default in Fastify v5)

### Reading incoming requests

Explain how to read incoming requests in Fastify, including:

- Query parameters
- Route parameters
- Headers
- Body parameters
- Files
- Cookies

### Validation

How to validate incoming requests in Fastify using schemas.

### Serialization

To speed up the response, we can use serialization in Fastify.
Explain how to use serialization in Fastify and the security implications of using it.

### Hooks

Explain the concept of hooks in Fastify and how they can be used
to modify the request/response lifecycle.

### Error Handling

Explain how to handle errors in Fastify.

### Lifecycle overview

Now that we have all the pieces, we can unveil the different lifecycle phases
of Fastify.

### Startup Lifecycle

Explain the startup lifecycle of Fastify (application boot sequence and hooks).

### Request Lifecycle

It includes the route matching and hooks.

### Response Lifecycle

### Shutdown Lifecycle

## Application Development

This section covers real-world application development topics using Fastify.

### Proxy Support

Explain how to configure Fastify to work behind a reverse proxy, such as Nginx or Apache.

### Loading application configuration

Explain how to load application configuration in Fastify, including:

- Using environment variables
- Using an external secrets manager

### Logging

Share a brief overview of the logging system in Fastify and some
good default configurations to use.

### OpenTelemetry

Guide to integrating OpenTelemetry with Fastify by using the `@fastify/otel` plugin.

### Security

Mention some security best practices when using Fastify, such as:

- Configuring CORS
- Rate limiting
- Input validation (already covered in the validation section)

### External plugins

Mention some official Fastify plugins that can be used to extend the functionality of Fastify.
Warn the reader to check a community plugin before using it,
as it may not be maintained or may not follow the Fastify principles.
