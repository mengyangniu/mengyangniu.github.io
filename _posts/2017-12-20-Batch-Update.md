---
title: Batch Update
date: 2017-12-20 00:16:00
categories:
- Script
tags:
- Linux
- Script
---

<center>To batch update items using homebrew or pip</center>

<!-- more -->

## Homebrew
```bash
for formula in $(brew outdated | cat);
    do brew upgrade $formula;
done
```

## pip
```python
# pip version <= 10.0
import pip
from subprocess import call

for dist in pip.get_installed_distributions():
    call("pip install --upgrade " + dist.project_name, shell=True)
```

```python
# pip version > 10.0 (18.0)
import pip
from pip._internal.utils.misc import get_installed_distributions
from subprocess import call

for dist in get_installed_distributions():
    call("pip install --upgrade " + dist.project_name, shell=True)
```

## TODO

Bash/zsh
