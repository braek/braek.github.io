---
layout: post
title: How to show SQL queries in Django unit tests
---

Recently I wanted to see which SQL queries were executed in some Django unit tests.

In Java and Hibernate this is easily accomplished by setting the **show_sql** setting to **true,** but Django does not offer a setting like that.

Below is a code example to show the SQL queries in Django unit tests.

```
from django.test import TestCase
from django.test.utils import override_settings
from django.db import connection
from django.conf import settings


def show_sql():
    for query in connection.queries:
        print(query['sql'])


class YourTestCase(TestCase):
    @override_settings(DEBUG=True)
    def test_your_unit_test(self):
        # Do some database stuff
        show_sql()
```

Make sure that the **DEBUG** setting is set to **True** with the **override_settings** decorator. In the example above, the decorator is only used on a single unit test, but you can also use the decorator on a complete test case.

Happy unit testing!
