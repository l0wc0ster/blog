[back](/)

Example 1. You have a script such as backup.py in a directory with file write permissions. If the script contains import of libraries, for example «import zipfile», then you can create a reverse or bind shell named «zipfile.py» in the same directory, the Python interpreter will run it before looking at the libraries in the standard directories:

```
python3 -c'import sys; print ("\ n" .join (sys.path)) '

/usr/lib/python35.zip

/usr/lib/python3.5

/usr/lib/python3.5/plat-x86_64-linux-gnu

/usr/lib/python3.5/lib-dynload

/usr/local/lib/python3.5/dist-packages

/usr/lib/python3/dist-packages
```
![Image](/img/python_lib_hijacking/1.png)