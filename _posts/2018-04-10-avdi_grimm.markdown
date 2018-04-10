---
layout: post
title:  "Паттерны из книги Confident Code by Avgi Grimm"
date:   2018-04-10 00:05:41 +0900
category: ruby
---

Основная концепция книги сводится к тому, чтобы все процессы сводились к 4 шагам:
- Collecting input
- Performing work
- Delivering output
- Handling failures

![Процесс](http://dl3.joxi.net/drive/2018/04/10/0018/2910/1186654/54/62df256b6b.jpg)

Далее нам необходимо всегда делить виртульно наш код на сообщения и получателя.
Выглядит это так:

![message_role](http://dl3.joxi.net/drive/2018/04/10/0018/2910/1186654/54/ff7d5be6a0.jpg)

Итак по порядку.

## Collecting Input

- # Introduction to collecting input
  Разделять методы по их основным предназначением.
  Одна задача на класс/метод дает нам возможность уменьшить связность между кодовой базой и уменьшить потенциальные баги.
  Создавать классы и константы, которые можно дальше использовать в других классах/методах. Если что-то нужно поменять, то поменяется в одном месте, а не во всей кодовой базе.

Use built-in conversion protocols
Conditionally call conversion methods
Define your own conversion protocols
Define conversions to user-defined types
Table of Contents
Use built-in conversion functions
Use the Array() conversion function to array-ify inputs
Define conversion functions
Replace "string typing" with classes
Wrap collaborators in Adapters
Use transparent adapters to gradually introduce abstraction
Reject unworkable values with preconditions
Use #fetch to assert the presence of Hash keys
Use #fetch for defaults
Document assumptions with assertions
Handle special cases with a Guard Clause
Represent special cases as objects
Represent do-nothing cases as null objects
Substitute a benign value for nil
Use symbols as placeholder objects
Bundle arguments into parameter objects
Yield a parameter builder object
Receive policies instead of data
* Delivering Results
Write total functions
Call back instead of returning
Represent failure with a benign value
Represent failure with a special case object
Return a status object
Yield a status object
Signal early termination with throw
* Handling Failure
Prefer top-level rescue clause
Use checked methods for risky operations
Use bouncer methods