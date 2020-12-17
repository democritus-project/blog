---
title: "Enumerating Exception Handling/Raising"
date: 2020-12-16T18:42:33-05:00
tags: ["exceptions", "try-except", "error handling", "errors", "raise", "tests", "generating tests"]
---

## Introduction

This blog post will detail how we enumerated the ways that exceptions can be handled and raised to test some new functions in Democritus. It is designed to give some insight into:

1. How we design tests in Democritus
2. The complexity required to write robust code
3. How to test exception handling/raising in Python

The end product of this blog post was used to generate the tests for the `ast_data::python_exceptions_handled` and `ast_data::python_exceptions_raised` functions (both of which take some code and return the names of the exceptions handled/raised). To make sure our tests are robust and thorough, we wanted to enumerate the possible ways in which exceptions can be handled/raised. In this blog post, we start discussing `raise` statements (both outside and inside of try-except blocks) and then move on to how exceptions can be handled in try-except blocks.

## Raising Exceptions

A `raise` statement can occur outside a try-except block or inside of one. We'll consider both locations.

### Raise Outside of Try-Except Block

There are, at least, two ways to categorize a `raise` statement that occurs outside (or inside) a try-except block:

1. Whether the `raise` statement raises a custom exception or a built-in one
2. Whether the `raise` statement raises the exception by referencing the exception explicitly or through a variable name

We represent the possible combinations of these two criteria like this:

```python
raise_categories = cartesian_product(('custom', 'built_in'), ('explicit', 'named'))
assert raise_categories == [('custom', 'explicit'), ('custom', 'named'), ('built_in', 'explicit'), ('built_in', 'named')]  # True
```

### Raise Inside of Try-Except Block

But one can also use `raise` inside of a try-except block, so we must expand our enumeration to include the following categories:

1. Does the `raise` statement raise the exception handled by the except handler or a different exception?
2. If the `raise` statement raises the exception handled by the except handler, does it raise it using a name defined in the except block?

Category two (above) differentiates between the following situations:

```python
try:
    1 / 0
except ZeroDivisionError as e:
    raise ZeroDivisionError  # the exception being handled is also raised, but not using the name "e"
```

```python
try:
    1 / 0
except ZeroDivisionError as e:
    raise e  # the exception being handled is also raised by the name defined in the except block
```

The capture the second criteria, we can expand our `exception_categories` above to differentiate between `raise` statements that use a name defined before the except handler and those which use the name in the except handler:

```python
raise_categories_in_except = cartesian_product(('custom', 'built_in'), ('explicit', 'named_before_except', 'named_in_except'))
assert raise_categories_in_except == [('custom', 'explicit'),
 ('custom', 'named_before_except'),
 ('custom', 'named_in_except'),
 ('built_in', 'explicit'),
 ('built_in', 'named_before_except'),
 ('built_in', 'named_in_except')]  # True
```

To capture the first criteria, we take `cartesian_product(('afore_mentioned', 'different'), exception_categories)`:

```python
raise_categories_in_try_except = cartesian_product(('afore_mentioned', 'different'), raise_categories_in_except)
# if a category has "named_in_except", it must also have "afore_mentioned"#
raise_categories_in_try_except = list(filter(lambda x: x[0] == 'afore_mentioned' if x[1][1] == 'named_in_except' else True, raise_categories_in_try_except))
assert raise_categories_in_try_except == [('afore_mentioned', ('custom', 'explicit')),
 ('afore_mentioned', ('custom', 'named_before_except')),
 ('afore_mentioned', ('custom', 'named_in_except')),
 ('afore_mentioned', ('built_in', 'explicit')),
 ('afore_mentioned', ('built_in', 'named_before_except')),
 ('afore_mentioned', ('built_in', 'named_in_except')),
 ('different', ('custom', 'explicit')),
 ('different', ('custom', 'named_before_except')),
 ('different', ('built_in', 'explicit')),
 ('different', ('built_in', 'named_before_except'))]  # True
```

We must also capture a case where there is only a `raise` statement and no name is given:

```python
raise_categories_in_try_except.append('')
assert raise_categories_in_try_except == [('afore_mentioned', ('custom', 'explicit')),
 ('afore_mentioned', ('custom', 'named_before_except')),
 ('afore_mentioned', ('custom', 'named_in_except')),
 ('afore_mentioned', ('built_in', 'explicit')),
 ('afore_mentioned', ('built_in', 'named_before_except')),
 ('afore_mentioned', ('built_in', 'named_in_except')),
 ('different', ('custom', 'explicit')),
 ('different', ('custom', 'named_before_except')),
 ('different', ('built_in', 'explicit')),
 ('different', ('built_in', 'named_before_except')), '']  # True
```

Thus, we have a pretty good enumeration of how a `raise` statement can be used. Now we'll consider how exceptions can be handled.

## Handling Exceptions

The primary way to categorize except handlers is to consider whether or not they handle one or many exceptions. For example, the except handlers in the examples above all handle one (ZeroDivisionError) exception.

In the example below, the except handler handles three exceptions (RuntimeError, RuntimeWarning, and ZeroDivisionError):

```python
try:
    1 / 0
except (RuntimeError, RuntimeWarning, ZeroDivisionError):
    pass
```

So, we need to take into account the fact that an except handler can handle one or many exceptions:

```python
cartesian_product(('one', 'many'), raise_categories_in_try_except)
```

This produces:

```python
[('one', ('afore_mentioned', ('custom', 'explicit'))),
 ('one', ('afore_mentioned', ('custom', 'named_before_except'))),
 ('one', ('afore_mentioned', ('custom', 'named_in_except'))),
 ('one', ('afore_mentioned', ('built_in', 'explicit'))),
 ('one', ('afore_mentioned', ('built_in', 'named_before_except'))),
 ('one', ('afore_mentioned', ('built_in', 'named_in_except'))),
 ('one', ('different', ('custom', 'explicit'))),
 ('one', ('different', ('custom', 'named_before_except'))),
 ('one', ('different', ('built_in', 'explicit'))),
 ('one', ('different', ('built_in', 'named_before_except'))),
 ('one', ''),
 ('many', ('afore_mentioned', ('custom', 'explicit'))),
 ('many', ('afore_mentioned', ('custom', 'named_before_except'))),
 ('many', ('afore_mentioned', ('custom', 'named_in_except'))),
 ('many', ('afore_mentioned', ('built_in', 'explicit'))),
 ('many', ('afore_mentioned', ('built_in', 'named_before_except'))),
 ('many', ('afore_mentioned', ('built_in', 'named_in_except'))),
 ('many', ('different', ('custom', 'explicit'))),
 ('many', ('different', ('custom', 'named_before_except'))),
 ('many', ('different', ('built_in', 'explicit'))),
 ('many', ('different', ('built_in', 'named_before_except'))),
 ('many', '')]
```

This gives us a complete enumeration (for our testing purposes) of how exceptions can be handled and raised.

## Summary

So, the data above (in the last code block) provides us with everything we need to create tests. For example, the entry `('many', ('afore_mentioned', ('built_in', 'explicit')))` will be translated into:

```python
try:
    pass
except (RuntimeError, RuntimeWarning):
    raise RuntimeError
```

We hope this was insightful and encourages you to explore this esthetic earth!
