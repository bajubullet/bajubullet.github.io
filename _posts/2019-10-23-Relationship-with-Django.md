Newer versions of Django have introduced `on_delete` parameter in `ForeignKey` field. This has introduced a few gotchas.

Earlier it didn't matter which way you declare a relationship **_child to parent_** or **_parent to child_**. But now it does matter.

If you use `on_delete=models.CASCADE` which is the most sensible option, the second **_parent to child_** relationship can create problems.

Here let me demonstrate:
```python
class User(models.Model):
  username = models.CharField(max_length=63)
  address = models.ForeignKey(Address, on_delete=models.CASCADE)


class Address(models.Model):
  full_address = models.TextField()
```

In this case parent(User) has a `ForeignKey` relationship to child(Address), this will create a side-effect. If you delete the `Address` object, it will also delete the related `User` object because of `on_delete=models.CASCADE`.

So situations like this should always be declared with a **_child to parent_** relationship as follows:
```python
class User(models.Model):
  username = models.CharField(max_length=63)


class Address(models.Model):
  user = models.ForeignKey(User, on_delete=models.CASCADE)
  full_address = models.TextField()
```
