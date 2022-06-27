# JinkoDB 
#### The fastest data management system based on JSON File

## Installation
```python
pip install jingodb
```
## requirements
```python
pip install orjson
```


------------

# Quick start
```python
from JinckoDB import connect

con = connect('test.json' , compressed=True)
```
Connect Class:
- Parameter db : dabase name (better be json)
- Parameter Compressed : if True json file will be on one line
- Parameter Cache : if on data will be store in RAM and be more faster . don\'t forgot , you can use `commit()` in Cache mode for backup data

# Types of data storage
In the JingoDB we have 3 types of data storage:
 - Config : key-value but configuration-optimized for easy usage.
 - Key-Value : Like normal json or python dictionary .
 - Tables (Like collection): rows/columns and records same SQLs
 
 and you can save [Orjson Types](https://github.com/ijl/orjson#types "Orjson Types") in JingoDB

 
# Speed
JingoDB is much faster than your json file-like database of mangers

This is an in-table data record comparison between JingoDB and TinyDB:
```python
from tinydb import TinyDB
from JinckoDB import connect
from time import time
db = TinyDB('tiny.json')
table = db.table('user')
start = time()
for i in range(2000):
    table.insert({'name': 'hosein' , 'lname' : 'erish' , 'age' : 15})
print('TinyDB : ',time() - start)

db = connect('jongo.json')
db.mkTable('user' , 'name' , 'lname' , 'age')
start = time()
for i in range(2000):
    db.inTable('user' , 'hosein' , 'erish' , 15)
db.commit()
print('JingoDB : ',time() - start)
```
### Result : `TinyDB :  7.953874111175537 JingoDB :  0.005991220474243164`
`and without commit (cache mode) : JingoDB :  0.0013866424560546875`
In fact, JingoDB is ~1000 times faster than tinyDB.

# Base Store Data Structure
In the JingoDB data will store like this:
```python
║
║                   ┌──__INFO__TableColumns
╬═══ __table__ ────────Records
║                   ┌──Name
╬═══ __config__────────Atters
║
╬═══ __key__   ──key──value
║
```
# Configs
- Create a new config:

```python
from JinckoDB import connect

con = connect('test.json' , compressed=False, cache=False)

class myConfig:
    def __init__(self) -> None:
        self.name = 'hosein'
        self.lname = 'erish'
        self.age = 15
        self.__class__.__name__ = 'hosein_erish_data'

con.mkConfig(myConfig())
con.commit()
```
config will be stored in `['__config__']['hosein_erish_data']` like this:
```json
"hosein_erish_data": {
      "name": "hosein",
      "lname": "erish",
      "age": 15,
      "__id__": 0
    }
```
Note: configs can not have same name
- Give config:

```python
from JinckoDB import connect

con = connect('test.json' , compressed=False, cache=False)

z = con.giveConfig('hosein_erish_data')
print(z.name)
# hosein

```
- Update config:
There is 2 way to update configs:
1. change give object:

```python
from JinckoDB import connect

con = connect('test.json' , compressed=False, cache=False)

z = con.giveConfig('hosein_erish_data')
updater = z.__dict__
updater['name'] = 'Mohamad'
updater['lname'] = 'Abbasi'
z.__dict__ = updater
con.upConfig('hosein_erish_data' , z )
con.commit()
```
2. change with keywords:

```python
from JinckoDB import connect

con = connect('test.json' , compressed=False, cache=False)

con.upConfig('hosein_erish_data' , name = 'Mohamad' , lname = 'Abbasi')
con.commit()
```
# Tabels
In the gincko db tables are save with this pattern:
```json
"__table__"{
"table_name" : [
     [{'__INFO__': [col1_name , col2_name]}]
	 ],
	 [record1],
	 [record2],
	 [record3]
}
```

- Create table :
In gincko db create and edit table are merged:

Create table with model:
```python
from JinckoDB.JinckoDB import connect , preModel

con = connect('test.json' , compressed=False , cache=False)

class table:
    def __init__(self) -> None:
        self.name = None
        self.lname = None
        self.phone = None
        self.id = None
        self.__exists__ = False
        self.__class__.__name__ = 'employee'
con.mkModelTable(table())
con.commit()
```
Create table without model:
```python
from JinckoDB.JinckoDB import connect , preModel

con = connect('test.json' , compressed=False , cache=False)

con.mkTable('employee' , 'name' , 'lname' , 'phone' , 'id',Exists=False)
con.commit()
```
Table will save in `['__table__']['__employee__']` If table now exists before if parameter exists be True table will change with new details was imported.
```json
"employee": [
      {
        "__INFO__": [
          "name",
          "lname",
          "phone",
          "id"
        ]
      }
    ]
```
- Insert Into Table

with model
```python
from JinckoDB.JinckoDB import connect , preModel

con = connect('test.json' , compressed=False , cache=False)
@preModel
class model:
    def __init__(self) -> None:
        self.name = 'hosein'
        self.lname = 'erish'
        self.phone = '+989010000000'
        self.id = 231

con.inTable('employee' , __model__=model)
con.commit()
```
**notic atters name is not important function receives them with table order top of down**

without model
```python
from JinckoDB.JinckoDB import connect , preModel

con = connect('test.json' , compressed=False , cache=False)
con.inTable('employee' ,'hosein' , 'erish' ,'+989010000000' , 231)
con.commit()
```

- Update table:

```python
from JinckoDB.JinckoDB import connect , preModel , finder

con = connect('test.json' , compressed=False , cache=False)
con.upTable('employee' , finder._is('name' , 'hosein') , name = 'mohamad')
con.commit()
```

data will be : 
```json
[
        "mohamad",
        "erish",
        "+989010000000",
        231
      ]
```

### finder class:
|name|condition|write|
| :------------ | :------------: | :------------ |
|`\_is`|`==`|`finder.\_is(column , value)`|
|`not_is`|`!=`|`finder.not_is(column , value)`|
|`condition`|`custom condition in cond param`|`finder.condition(column,cond, value)`|
|`all`|`all data`|`finder.all()`|

- Delete Record:

```python
from JinckoDB.JinckoDB import connect , preModel , finder

con = connect('test.json' , compressed=False , cache=False)
con.delRecord('employee' , finder._is('name' , 'mohamad'))
con.commit()
```
Also you can use delRecord with id (number of inserted):
```python
from JinckoDB.JinckoDB import connect , preModel , finder
con = connect('test.json' , compressed=False , cache=False)
con.delRecord('employee' ,_id =2)
con.commit()
```

-Delete Table:
```python
con = connect('test.json' , compressed=False , cache=False)
con.delTable('employee')
con.commit()

```
json will be like this:
```json
"__table__": {},
```
- Give table:
first we insert 50 random value in json:

```python
from JinckoDB.JinckoDB import connect , preModel , finder

con = connect('test.json' , compressed=False , cache=False)
con.mkTable('user' , 'name' , 'lname' , 'age')

for i in range(50):
    @preModel
    class insert:
        def __init__(self) -> None:
            self.name = 'hosein'
            self.lname = f'{i}_erish'
            self.age = i
    con.inTable('user' ,__model__ = insert )
con.commit()
```
and json file now:

```json
{
  "__table__": {
    "user": [
      {
        "__INFO__": [
          "name",
          "lname",
          "age"
        ]
      },
      [
        "hosein",
        "0_erish",
        0
      ],
	  [
        "hosein",
        "1_erish",
        1
      ],
	  ....
```

Find records are have age bigger than 47:
```python
from JinckoDB.JinckoDB import connect , preModel , finder

con = connect('test.json' , compressed=False , cache=False)
a = con.giveTable('user' , finder.condition('age' , '>=' , 48))
print(a)

```
result = `[['hosein', '48_erish', 48], ['hosein', '49_erish', 49]]`

# Keys
Working with keys is so easy in `__keys__` you create a variable in json (more usage in cache mode)
```python
from JinckoDB.JinckoDB import connect , preModel , finder

con = connect('test.json' , compressed=False , cache=False)
#set and update keys
con.setkey('key name' , 'test')
#get key
a = con.giveKey('key name')
#delete key
con.delKey('key name')

print(a)
```
Result : `test`



