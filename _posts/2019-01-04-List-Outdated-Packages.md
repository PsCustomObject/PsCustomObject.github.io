---
title: "Python - List Outdated Packages"
excerpt: "How to list all outdated Python packages with Pip and how to upgrade them"
categories:

  - Python
  - Tips
tags:
  - Python
  - Pip
  - Pip Packages
  - Python3
---

Python3 comes with a list of pre-installed packages which are updated from time to time. It is really easy to list any *outdated* package with the following command:

```python
pip list --outdated

# Output
Package    Version   Latest     Type
---------- --------- ---------- -----
astroid    2.0.4     2.1.0      wheel
certifi    2018.4.16 2018.11.29 wheel
pylint     2.1.1     2.2.2      wheel
setuptools 40.0.0    40.6.3     wheel
six        1.11.0    1.12.0     wheel
```

As you can see 5 packages on my machine are outdated.

To update a specific package to the latest version you can use

```python
pip install --upgrade  <PackageName>

# Alternatively you can use the short form
pip install -U <PackageName>
```

In you don't want to install each update individually you can use *pipdate*

```python
# Install pipdate
sudo pip3 install pipdate

# Update all packages
sudo pipdate3
```

**Note for Windows users:** *Pipdate* is already installed by default so you don't need to install it

**Note for Mac users:** If you installed *Python* via **homebrew** there is no need to use *sudo* to install *pipdate* or upgrade packages
