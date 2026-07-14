# 11 — Object-Oriented Programming (OOP)

## Classes and Objects
A **class** is a blueprint; an **object** (instance) is a specific thing built from that blueprint.

```python
class Dog:
    def __init__(self, name, breed):    # constructor — runs when object is created
        self.name = name                  # instance attribute
        self.breed = breed

    def bark(self):                        # instance method
        print(f"{self.name} says Woof!")

my_dog = Dog("Jimmy", "Labrador")      # creating an object (instance)
my_dog.bark()                             # Jimmy says Woof!
print(my_dog.name)                          # Jimmy
```

## Understanding `self`
`self` refers to the specific instance calling the method. It must be the first parameter of every instance method, though Python passes it automatically when you call `object.method()`.
```python
my_dog.bark()          # Python internally does: Dog.bark(my_dog)
```

## `__init__` — the constructor
Runs automatically when a new object is created. Used to initialize instance attributes.
```python
class Student:
    def __init__(self, name, marks):
        self.name = name
        self.marks = marks

s1 = Student("Soham", 90)
```

## Instance attributes vs Class attributes
```python
class Dog:
    species = "Canine"     # class attribute -> shared by ALL instances

    def __init__(self, name):
        self.name = name     # instance attribute -> unique per object

d1 = Dog("Jimmy")
d2 = Dog("Rex")
print(d1.species, d2.species)     # Canine Canine -> same for both
print(d1.name, d2.name)             # Jimmy Rex -> different per instance
```

## The Four Pillars of OOP

### 1. Encapsulation — bundling data and methods, restricting direct access
```python
class Account:
    def __init__(self, balance):
        self.__balance = balance     # double underscore = "private" (name-mangled)

    def deposit(self, amount):
        self.__balance += amount

    def get_balance(self):
        return self.__balance

acc = Account(1000)
acc.deposit(500)
print(acc.get_balance())     # 1500
# acc.__balance would raise an AttributeError (can't access directly)
```
Note: Python doesn't have true "private" like Java/C++ — `__balance` is just name-mangled to `_Account__balance`, technically still accessible, but the convention signals "don't touch this directly."

### 2. Inheritance — a class reusing/extending another class
```python
class Animal:
    def __init__(self, name):
        self.name = name

    def speak(self):
        print(f"{self.name} makes a sound")

class Dog(Animal):              # Dog inherits from Animal
    def speak(self):              # method overriding
        print(f"{self.name} barks")

d = Dog("Jimmy")
d.speak()     # Jimmy barks   -> uses Dog's version, not Animal's
```

### super() — call the parent class's method
```python
class Dog(Animal):
    def __init__(self, name, breed):
        super().__init__(name)     # calls Animal's __init__
        self.breed = breed
```

### 3. Polymorphism — same method name, different behavior per class
```python
class Cat(Animal):
    def speak(self):
        print(f"{self.name} meows")

animals = [Dog("Jimmy"), Cat("Whiskers")]
for a in animals:
    a.speak()     # each calls its OWN version of speak() -> polymorphism
```

### 4. Abstraction — hiding complex implementation, exposing only what's needed
```python
from abc import ABC, abstractmethod

class Shape(ABC):
    @abstractmethod
    def area(self):
        pass     # subclasses MUST implement this

class Circle(Shape):
    def __init__(self, radius):
        self.radius = radius

    def area(self):
        return 3.14159 * self.radius ** 2

c = Circle(5)
print(c.area())     # 78.53975
# Shape() directly would ERROR -> can't instantiate an abstract class
```

## Dunder (magic/special) methods
Methods with double underscores that let objects work with Python's built-in syntax.
```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __str__(self):                    # controls what print(obj) shows
        return f"Point({self.x}, {self.y})"

    def __eq__(self, other):                # controls what == does between objects
        return self.x == other.x and self.y == other.y

p1 = Point(1, 2)
p2 = Point(1, 2)
print(p1)          # Point(1, 2)   -> uses __str__
print(p1 == p2)      # True          -> uses __eq__
```

## Class methods and static methods
```python
class MathUtils:
    @staticmethod
    def add(a, b):          # doesn't need access to self or the class
        return a + b

    @classmethod
    def create_default(cls):  # takes the class itself (cls) instead of an instance
        return cls(0, 0)

MathUtils.add(3, 4)     # 7 -> called directly on the class, no instance needed
```
