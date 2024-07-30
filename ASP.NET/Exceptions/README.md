# Exceptions

 Here you can find how to handle exceptions in ASP.NET Core.

## Table of Contents

- [Overview](#overview)
- [Exception Filter](./ExceptionFilter/README.md)
- [Exception Handler (or Middleware)](./ExceptionHandler/README.md)

## Overview

Exception filter and exception handler are two ways to handle exceptions in ASP.NET Core.
But they have different purposes.

### Exception Filter

- Used to handle exceptions in the MVC pipeline.
- It's a filter that runs before and after the action method.
- It's a good place to handle exceptions and return a response.

### Exception Handler (or Middleware)

- Used to handle exceptions globally.
- It's a middleware that runs for every request.
- It's a good place to log exceptions and return a response.