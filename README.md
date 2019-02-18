# markdown2html

A simple markdown to html converter.

Only a subset of markdown is implemented.

## install

```
python setup.py install
```

## supported syntax

```
# title
```

```
## subtitle
```

```
[link] (https://www.example.com/)
```

```
![image] (https://upload.wikimedia.org/wikipedia/commons/7/70/Example.png)
```

(code-snippet seperated with a newline)

## usage

```
usage: markdown2html [-h] [filename]

converts markdown to html

positional arguments:
  filename    Input markdown filename. '-' for stdin.

optional arguments:
  -h, --help  show this help message and exit
```
