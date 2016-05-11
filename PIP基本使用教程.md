# PIP基本使用教程

pip包升级

```bash
$ pip install --upgrade p
```

安装相应包

```bash
$ pip install pkg_name
```

查看某个库的信息

```bash
$ pip show Jinja2
---
Name: Jinja2
Version: 2.7.3
Location: /path/to/virtualenv/lib/python2.7/site-packages
Requires: markupsafe
```

查看已经安装的库：

```
$ pip list
```

获取过期的库：

```
$ pip list --outdated
```