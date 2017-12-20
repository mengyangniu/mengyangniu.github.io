---
title: Batch Update
date: 2017-12-20 00:16:00
categories:
- Script
tags:
- Linux
- Script
---

<center><i>To batch update items using homebrew of pip</i></center>

<!-- more -->

## Homebrew
```bash
for formula in $(brew outdated | cat);
    do brew upgrade $formula;
done
```

## pip
```python
# python script
import pip
from subprocess import call
for dist in pip.get_installed_distributions():
    call("pip install --upgrade " + dist.project_name, shell=True)
```

## TODO
Bash/zsh
