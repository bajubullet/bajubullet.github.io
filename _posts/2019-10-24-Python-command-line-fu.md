Some times it is easy to do certain jobs with small python scripts but if you don't want to save a file here is a quick example to run python line a command

```bash
$ python -c "import datetime; print(datetime.datetime.utcnow())"
```

but how do you pass values in such situations, for that you can use:

```bash
$ echo "world" | python -c 'print("Hello", raw_input(), "!")'
```
