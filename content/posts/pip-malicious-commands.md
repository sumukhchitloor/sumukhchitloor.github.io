+++
draft = false
date = 2024-03-31T10:00:00+05:30
title = "Using pip to Run Malicious Commands"
description = "Discover how attackers exploit pip to run malicious commands during Python package installation. Learn techniques for prevention, detection, and secure package management."
slug = "pip-malicious-commands"
authors = ["Sumukh Chitloor"]
keywords = ["python-security", "package-exploit", "pip-security", "code-injection"]
tags = ["security", "python", "pip", "malware", "code-execution"]
externalLink = ""
series = ["Malware Development"]
+++

So, recently I was solving a hacktheboox room OnlyForYou and I got the user flag but I was struggling to get the root flag. User had this previlage to run as sudo.

```bash
(root) NOPASSWD: /usr/bin/pip3 download http\://127.0.0.1\:3000/*.tar.gz
``` 

![Sudo pip privilege showing download permission](/images/pip-malicious-blog/pip-exploit.png "Sudo permission to run pip download")

But I had never seen a malicious use of pip before so I started doing research. And I founf some interesting stuffs.


### Python Package
I've never built a python package myself, but there are tons of resources for that. There are different ways to go about it, and the vulnerability exists when using the setup.py variation and when the package is in a .tar.gz. 

In the python package, there is cmdclass in the setup, which causes pip to execute the provided command function upon both download and install of the package.

```bash
cmdclass={
        'install' : RunInstallCommand,
        'egg_info': RunEggInfoCommand
    },
```

```bash
def RunCommand():
    os.system("chmod u+s /bin/bash")

class RunInstallCommand(install):
    def run(self):
        RunCommand()
        install.run(self)
```

These are the main parts in the setup.py. When we build the package cmd class executes the `RunInstallCommand()`  which will run the `RunCommand()` and executes the malicious command.

### Building the package
We can go and build the package using 

```bash
python -m build
```

After building the package `./dist` directory will be created containing the wheel and `tar.gz` files. For this demo exploit we are only interesting in the `tar.gz`. file.

Suppose you recently attempted to download or install a Python package using pip. In that case, Python may have provided you with a.whl file. The reason for this is that when developers construct a Python package using, for example, the `pip -m build` command, pip automatically tries to produce a secondary.whl file in addition to the tar.gz file, which is subsequently published to the Python Package manager platform along. When a user downloads or installs this package, PIP will automatically download and install the.whl file to the user's machine. The way wheels function eliminates the need for setup.py execution.

Even though pip prefers wheels over tar.gz files, hackers can nevertheless publish python packages without a.whl file on purpose. When a user downloads a Python package from PyPi, pip will utilize the.whl file first, but will fall back to the tar.gz file if the.whl file is not there.

### Exploitation
![exploitation image](/images/pip-malicious-blog/exploitation.avif "exploitation image")

As I was only allowed to download anything using pip on localserver at port 3000, I added malicious .tar.gz in the repository on the local Gogs' git server. 

When I downloaded that malicious package using pip download, my malicious command was executed

```bash
sudo /usr/bin/pip3 download http\://127.0.0.1\:3000/john/Test/src/master/this_is_fine_wuzzi-0.0.1.tar.gz
```

after we download this,

![prev esc image](/images/pip-malicious-blog/priv-esc.png "priv esc image")

we escalated our shell as root.


### What can be done about this?
- Prefer packages with wheels, less likely to run malicious setup.py
- Still inspect code if from an untrusted source, wheels are not inherently safe


### Resources
- [create-malicious-python-package](https://exploit-notes.hdks.org/exploit/linux/privilege-escalation/pip-download-code-execution/#1.-create-malicious-python-package)