---
id: vquqspgi7f6ncuii62yd1q9
title: Sphinx
desc: ''
updated: 1676797540693
created: 1676797479790
---

## requirements

```
pip install sphinx

pip install sphinx-rtd-theme
# https://sphinx-themes.org/#themes
```

## sphinx-quickstart

```
mkdir docs
cd docs
sphinx-quickstart
```

## sphinx-apidoc

```
sphinx-apidoc -o ./docs .
```

**Note**: The sub folders without `__init__.py` are ignored by sphinx-apidoc.


## edit config files


```
# config.py
import os
import sys

sys.path.insert(0, os.path.abspath('..'))

extensions = [
    'sphinx.ext.autodoc',
    'sphinx.ext.todo',
    'sphinx.ext.viewcode',
]

language = 'zh_CN'

html_theme = 'sphinx_rtd_theme'
```

```
# index.rst

modules
```

## make html