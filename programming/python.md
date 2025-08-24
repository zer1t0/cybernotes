# Python

## Creating virtual environments

Install the following packages:
```
sudo apt install python3-pip
sudo pip3 install virtualenvwrapper --break-system-packages
```

Copy the following configuration into our .bashrc:
```sh
export VIRTUALENVWRAPPER_PYTHON=/usr/bin/python3
export VIRTUALENVWRAPPER_VIRTUALENV=/usr/local/bin/virtualenv
source /usr/local/bin/virtualenvwrapper.sh
```

And then close and reopen the shell.

Then we can create a virtual environment with:
```
mkvirtualenv test-env
```

We can activate the environment with:
```
workon test-env
```

And deactivate with:
```
deactivate
```

To delete the environment:
```
rmvirtualenv test-env
```

Additionally we can specify commands to be executed when the environment is
activated or deactivated by editing and making executable the following files:
```
$ ls ~/.virtualenvs/<env-name>/bin/p*activate
~/.virtualenvs/<env-name>/bin/postactivate
~/.virtualenvs/<env-name>/bin/postdeactivate
~/.virtualenvs/<env-name>/bin/preactivate
~/.virtualenvs/<env-name>/bin/predeactivate
```

Source: 
- [A guide to Python virtual environments with virtualenvwrapper](https://opensource.com/article/21/2/python-virtualenvwrapper) by Ben Nuttall
