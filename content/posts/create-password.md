---
title: "Creating a Password"
date: 2020-12-02T18:42:33-05:00
---

In the `democritus_core::fun.py` code, there is a `password_create` function.

I wanted to break this function down just to show you how we are doing it.

Here is the code:

```python
PASSWORD_CHARACTER_SET = '0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ!#$%&()*+,-.:;<=>?@[]^_`{|}~'


def password_create(*, length: int = 15, character_set: str = PASSWORD_CHARACTER_SET) -> str:
    """Create a password of the given length using the given character_set."""
    from random_wrapper import random_choices

    return ''.join(random_choices(character_set, length))

```

The function takes two, keyword-only arguments (you can read more about keyword-only arguments [here](https://www.python.org/dev/peps/pep-3102/)). It will choose `length` number of characters from the given `character_set`. The default character set is `PASSWORD_CHARACTER_SET` which, as seen in the code above, includes most printable, non-whitespace characters (excluding some problematic ones like forward and back slashes). Because `random_choices` returns a list, this list will be converted to a string using `''.join()` and returned.

That's how we can easily create a password generator using Democritus! Enjoy... and keep exploring!
