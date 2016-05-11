# Python虚拟环境virtualenv安装

在开发Python应用程序的时候，系统安装的Python3只有一个版本：3.4。所有第三方的包都会被pip安装到Python3的site-packages目录下。

如果我们要同时开发多个应用程序，那这些应用程序都会共用一个Python，就是安装在系统的Python 3。如果应用A需要jinja 2.7，而应用B需要jinja 2.6怎么办？

这种情况下，每个应用可能需要各自拥有一套“独立”的Python运行环境。`virtualenv`就是用来为一个应用创建一套“隔离”的Python运行环境。

首先，我们用pip安装virtualenv：

```
pip install virtualenv
```

然后，假定我们要开发一个新的项目，需要一套独立的Python运行环境，可以这么做：

- 第一步，创建目录：

```bash
$ mkdir myproject
$ cd myproject/
```

- 第二步，创建一个独立的Python运行环境，命名为venv：

```bash
$ virtualenv --no-site-packages venv

Using base prefix '/usr/local/.../Python.framework/Versions/3.4'
New python executable in venv/bin/python3.4
Also creating executable in venv/bin/python
Installing setuptools, pip, wheel...done.

```

新建的Python环境被放到当前目录下的`venv`目录。有了`venv`这个Python环境，可以用`source`进入该环境：

```bash
$ source venv/bin/activate
(venv)Mac:myproject michael$
```

#### 小结

virtualenv为应用提供了隔离的Python运行环境，解决了不同应用间多版本的冲突问题。

