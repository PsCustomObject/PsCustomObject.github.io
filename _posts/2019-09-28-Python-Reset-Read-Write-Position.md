---
title: "Reset Read or Write position in a file with Python with seek()"
excerpt: "How do I go back to the beginning of file after reading it with the read() method in Python?"
toc: true
categories:
  - Python
tags:
  - Python
  - Seek
  - Read
  - Python IO
  - Python file management
---

This is something I used to stumble on a lot in the early days of my Python coding adventure so I thought to write a quick article about it.

## Python Open file

Opening a file for read or write in Python can be achieved with the following code:

```python
# Open file
file = open("/Python/Files/MyFile.txt")
```

## Python Read file

To read content of the file we simply use the **read()** method

```python
# Read file to the end of file
file.read()

'This is the first line\nAnd a second\nAnd even a third\nShall we put a fourth?\nWhy not a fifth\nOr a sixt\n'
```

Note that Python will print the *newline* character and will not split lines for you (but that's material for another day/post)

## Where is my text?

Say you want to print again file content issue the same command again will leave you with something like this

```python
# Read file again
file.read()
''
```

Not a lot to see on screen, so where did our file's content go?

## Python seek()

When python reads a file's content it will move *file current position* to the end of the file so trying to re-read it again will yield the above result. Nothing as there is nothing else to read.

You can easily go back to the beginning of the file with the *seek()* method which is used like this:

```python
# Go back to position 0
# Or beginning of file
file.seek(0, 0)
```

- **Syntax of seek() method** fileObject.seek(offset[, whence])

- **offset** is the position of the read/write pointer within the file.

- **whence** is optional and defaults to 0 which means absolute file positioning, other possible values are 1 which means seek relative to the current position and 2 which means seek relative to the file's end
{: .notice--warning}

Now if you try to read the file again all content will be correctly displayed

```python
file.read()

'This is the first line\nAnd a second\nAnd even a third\nShall we put a fourth?\nWhy not a fifth\nOr a sixt\n'
```
