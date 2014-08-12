---
layout: post
title: Whitespace charecters in git diff
---


When it comes to version control systems git is the master of them all. But for git many people install separate diff tools but I like the default git diff with colors enabled but the problem is normal git diff does not show whitespace charecters. To fix this you can use the `-R` option like this:


```
git diff -R
```

or


```
git diff -R <filename>
```

Hope this helps.
