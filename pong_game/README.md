Installing Kivy on the Windows:

Setup Terminal and pip
```
python -m pip install --upgrade pip setuptools virtualenv
```

Create virtual environment
1. Create the virtual environment:
``` python -m venv kivy_venv ```
2. Activate the environment
``` kivy_venv\Scripts\activate ```

While activating environment you will get one error "running scripts is disabled on this system". to resolve this issue you can do following:

As an Administator, you can set the execution policy by typing this into your PowerShell window:
```
Set-ExecutionPolicy RemoteSigned
```

When you are done, you can set the policy back to its default value with:
```
Set-ExecutionPolicy Restricted
```

You may see an error, if yes then run below to change the execution policy for the current user:
```
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
```

Install Kivy

Pre-Compiled Wheels:
```
python -m pip install "kivy[base]" kivy_examples
```

To additionally install kivy with audio/video support, install either kivy[base, media] or kivy[full]


Create an application

Creating a kivy application is as simple as:
- Sub-classing the ```App``` class
- implement its ````build()``` method so it returns a ```Widget``` instance ( the root of your widget tree)
- instantiating this class, and calling its ````run()``` method.