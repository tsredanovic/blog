---
layout: post
title:  Virtual environments in Python
categories: Python
excerpt: A virtual environment is a Python environment such that the Python interpreter, libraries and scripts installed into it are isolated from those installed in other virtual environments, and (by default) any libraries installed in a “system” Python.
---

![]({{site.baseurl}}/images/2021-06-08-virtual-environments-in-python.png)

## Virtual environments in Python

A virtual environment is a Python environment such that the Python interpreter, libraries and scripts installed into it are isolated from those installed in other virtual environments, and (by default) any libraries installed in a “system” Python, i.e., one which is installed as part of your operating system.

Now that we got the formal definition out of the way, here is an example:

Lets say you have two projects: `Project A` and `Project B`, both of which have a dependency on the same library, a popular python web framework [Django](https://www.djangoproject.com/)). The problem becomes apparent when we start requiring different versions of Django. Maybe `ProjectA` was built some time ago using Django v1.11, while `ProjectB` is new, fresh and requires the newer Django v3.2.

The solution here would simply be to create a separate virtual environment for each project and install the appropriate version of Django inside.

**Versions used:** Python 3.7.9

### Creating a virtual environment

Python 3 has the [venv](https://docs.python.org/3/library/venv.html) module from the standard library installed so all you have to do is execute the following command to create a virtual environment:

```bash
python3 -m venv /path/to/new/venv
```

Running this command will create the target directory (creating any parent directories that don’t exist already) containing:
- `pyvenv.cfg` file - pointing to the Python installation from which the command was run
- `bin` (`Scripts` on Windows) subdirectory - containing a copy/symlink of the Python binary/binaries
- `lib/pythonX.Y/site-packages` (`Lib\site-packages` on Windows) subdirectory - contains installed dependencies (initially empty)

A common name to use when creating the target directory is `venv` or `.venv`. 

It is also worth noting that this directory should be ignored with `.gitignore` if you are using git.

### Activating a virtual environment

Once a virtual environment has been created, it can be activated using a script in the virtual environment's `bin` directory. The invocation of the script is platform-specific:

| Platform | Shell           | Command to activate                          |
| -------- | --------------- | -------------------------------------------- |
| POSIX    | bash/zsh        | `$ source /path/to/venv/bin/activate`        |
|          | fish            | `$ source /path/to/venv/bin/activate.fish`   |
|          | csh/tcsh        | `$ source /path/to/venv/bin/activate.csh`    |
|          | PowerShell Core | `$ /path/to/venv/bin/Activate.ps1`           |
| Windows  | cmd.exe         | `C:\> \path\to\venv\Scripts\activate.bat`    |
|          | PowerShell      | `PS C:\> \path\to\venv\Scripts\Activate.ps1` |

After activation your command prompt will be prefixed with the name of your environment looking like this:

```bash
...$ source venv/bin/activate
(venv) ...$
```

### Using a virtual environment

After activation you can use [pip](https://pip.pypa.io/en/stable/) as you normally would to manage your packages inside the virtual environment. 

For example running a `pip freeze` command, which outputs installed packages, would not output anything initially. You can install some packages like Django and all of its dependencies with `pip install Django`. Running `pip freeze` again would now return:

```
asgiref==3.3.4
Django==3.2.4
pytz==2021.1
sqlparse==0.4.1
typing-extensions==3.10.0.0
```

It is recommended to always provide a file which lists necessary packages for your project. You can do just that by outputing `pip freeze` to a [requirements.txt](https://pip.pypa.io/en/stable/user_guide/#requirements-files) file like so:

```bash
pip freeze > requirements.txt
```

### Deactivating a virtual environment

You can deactivate your virtual environment and go back to the system context by executing `deactivate` in you command prompt like so:

```bash
(venv) ...$ deactivate
...$
```

### Useful links

Here are a few interesting projects which expand on the virtual environment's functionality:

- [virtualenvwrapper](https://pypi.org/project/virtualenvwrapper/) - a useful set of scripts for creating and deleting virtual environments

- [pew](https://pypi.org/project/pew/) - provides a set of commands to manage multiple virtual environments

- [tox](https://pypi.org/project/tox/) - a generic virtualenv management and test automation command line tool, driven by a tox.ini configuration file

- [nox](https://pypi.org/project/nox/) - a tool that automates testing in multiple Python environments, similar to tox, driven by a noxfile.py configuration file
