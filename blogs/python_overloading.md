# Exploring Function Overloading in Python with Decorators

### functional overloading ??
so if you have written any kind of object oriented programming languages like c++ or java , you know what functional overloading is , if you don't here is a quick example written in c++ 
```cpp
int add(int a, int b) {
    return a + b;
}

double add(double a, double b) {
    return a + b;
}
```
Here you can see that we have two different functions with the same name but the compiler treats them differently based on their arguments and return types , This is basically functional overloading.
Python out of the box doesnt support functional overloading , if you write something like this
```python 
def greet(name):
    print("hello" , name)

def greet(name1 , name2):
    print("hello" , name1 , " and " , name2)

greet("tervicke") #trying to call the original function
```
This will return an error saying that greet expects 2 arguments but only 1 was provided because the newest definition overwrites the previous definition
So we are trying to solve this exact issue by using decorators 

### decorators ??
Decorators in python are syntax which is used to modify the behaviour of functions which changing the original source code , My interaction with them is mainly when I work with flask
```python
# sample flask code , has nothing to do with our project
@app.route("/") # <- decorator
def hello_world():
    return "<p>Hello, World!</p>"

```
Essentially decorators are just python functions that take functions as arguments , let us take a look at a very simple example by creating and using a custom decorator

```python
def mydec(func):
    print("decorating a function now")
    def wrapper(*args , **kwargs):
        print("before the function is executed")
        func(*args , **kwargs)
        print("after the function is executed")
    return wrapper

@mydec  
def greet():
   print("hello") 
   
greet()
```
running this code will yield this ouptut  
```angular2html
decorating a function now
before the function is executed
helloworld
after the function is executed
```
When we declare the greet function (without calling it), the print statement inside the decorator runs and prints "decorating a function now". You can verify this by removing the call to greet() and observing the output

This happens because decorating a function like @mydec is equivalent to writing greet = mydec(greet). The decorator is applied at the time of function definition, not when the function is called. As a result, greet gets replaced by the wrapper function, since that is what mydec returns

So now we know what exactly happens when we use a decorator .   
Lets see how we can use this to perform functional overloading

### The overloaded decorator
let us start by creating a decorator called overloaded
```python
def overloaded(func) :
    print("something to do when the function is being overloaded")
    def wrapper(*args , **kwargs):
        print("call the function here? which one ??")
    return wrapper
```
Okay let us think about the overloading part , I think you guys might have figured out that we need to store all the functions that are decorated so that we can dispatch or call the function depending upon the number of arguments etc . 
So let us start by creating a class to save the functional signature or the so called properties of of a function <br>
<br>
**NOTE** : This is a very basic implementation so even beginners can follow along , you can pretty easily implement the type annotation etc (hit me up if you do )
<br>

```python
import inspect # inbuilt module
class funcData():
    def __init__(self , func):
        self.func = func #the actual function
        self.name = func.__name__ #the name of the function
        sig = inspect.signature(func) # sig gets the signature of the the function  
        self.args = len(sig.parameters) #gets the number of arguments
```
That's it , That's all the things we are gonna store as function data , most of the things are self-explanatory , apart from that we use a module to extract information about the function
let us write the code to extract the funcdata and store it 
```python
def overloaded(func) :
    data = funcData(func)
    funcs.append(data)
    def wrapper(**kwargs , *args):
        print("call the function here? which one ??")
    return wrapper
```
Here funcs is a global list , used to store the data , we now have the data of the func whenever the overloaded decorator is used .
How do we display or return the correct wrapper ?
Now we are gonna do a brute-force search over all the funcs to get the correct function and then return it 
```python
def overloaded(func):
    data = funcData(func)
    funcs.append(data)
    def wrapper(*args , **kwargs):
        for fun_data in funcs:
            if fun_data.name == func.__name__ and fun_data.args == len(args) + len(kwargs):
                return fun_data.func(*args , **kwargs)
        return None
    return wrapper
  ```
Here we are doing nothing but looping through all the funcs data and then checking if the function name and the arguments add up correctly . if yes we return the same function and then finally we return the wrapper.    
there are different ways to tackle if we do not find the function you can raise a exception , I am simply returning a None type so python interpreter itself will report the error that None cannot if u try to call a function which is supposed to be overloaded with the wrong set of arguments.
Lets the the entire code once with the example we used in c++.
```python
import inspect

class funcData():
    def __init__(self , func):
        self.func = func
        self.name = func.__name__
        sig = inspect.signature(func)
        self.args = len(sig.parameters)
    
funcs = [] 

def overloaded(func):
    data = funcData(func)
    funcs.append(data)
    def wrapper(*args , **kwargs):
        for fun_data in funcs:
            if fun_data.name == func.__name__ and fun_data.args == len(args) + len(kwargs):
                return fun_data.func(*args , **kwargs)
        return None
    return wrapper

@overloaded
def greet(str1 , str2):
    print("hello" , str1 ,"and" ,str2)
    
@overloaded 
def greet(str1):
    print("hello" , str1)
    
greet("alice")
greet("alice" , "bob")
```
Yes thats it , when u run the above code u will see this output 
```angular2html
hello alice
hello alice and bob
```
We are essentially calling the same function greet with 2 different definitions and the overloaded decorator .<br>

### Final thoughts
This was a very simple and immature implementations but this same idea can be used to write a full fledge functional overloading system esp by adding type annotations and improving the time complexity of searching the function instead of a bruteforce .