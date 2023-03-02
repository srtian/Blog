在Python中，面向对象最重要的概念就是类和实例（Instance）。类是抽象的模板。而实例是根据类创建出来的一个个具体的“对象”，每个对象都拥有相同的方法，但各自的数据可能不同。定义类是通过class关键字：

```python
class Students(object):
    def __init__(self, name, age):
        self.name = name
        self.age = age

srtian = Students('srtian', 18)
srtian.name //srtian
srtian.age //18
```


### 数据封装
使用面向对象编程的一个重要特点就是数据封装，比如下面这样：

```python
class Students(object):
    def __init__(self, name, age):
        self.name = name
        self.age = age

    def printName(self):
        print('%s: %s' % (self.name, self.age))

    def safHi(self):
        print('hi, i am %s, i am %s' % (self.name, self.age))


srtian = Students('srtian', 18)
srtian.printName()
srtian.safHi()
```
此外，在class内部，可以有属性和方法，而外部代码可以通过直接调用实例变量的方法来才做数据，这样据可以隐藏掉内部的复杂逻辑。但外部代码还是可以自由的去修改实例的属性：

，如果想要实例内部的属性不被外部访问，可以把属性的名字前加上两个下划线__。在Python里面，实例的变量名如果以__开头，就变成来一个私有变量，只有内部可以访问：
```python
class Students(object):
    def __init__(self, name, age):
        self.__name = name
        self.__age = age

    def printName(self):
        print('%s: %s' % (self.__name, self.__age))

    def safHi(self):
        print('hi, i am %s, i am %s' % (self.__name, self.__age))


srtian = Students('srtian', 18)
srtian.printName()
srtian.safHi()

srtian.__name
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: 'Student' object has no attribute '__name'
```
这样就确保外部代码不能随便的修改对象内部的状态，使得代码变得更加健壮。

需要注意的是，在Python中，变量名类似__xxx__这样的，是特殊变量，特殊变量是可以被直接访问的，不是私有变量。以及以一个下划线开头的实例变量名，比如_name，这种实例变量是可以在外部访问的，但按照约定俗成的规定，这种其实表示：虽然可以直接被访问，但请将其视为私有变量，不要随意访问。

