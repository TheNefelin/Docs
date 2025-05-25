# Python

> [Python](https://www.python.org/)

* Virtual Environment
```cmd
pip install virtualenv
virtualenv -p python3 venv
.\venv\Scripts\activate
```
* list of all dependency
```cmd
pip list
```
* create the requirements file
```cmd
pip freeze > requirements.txt 
```
* install all dependency from requirements.txt
```cmd
pip install -r requirements.txt
```