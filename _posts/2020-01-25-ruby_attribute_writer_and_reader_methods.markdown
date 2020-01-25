---
layout: post
title:      "Ruby Attribute Writer and Reader Methods"
date:       2020-01-25 22:05:47 +0000
permalink:  ruby_attribute_writer_and_reader_methods
---


Instance variables, denoted with a @ symbol, allow you to access information about the instance outside the scope of one particular method, within the scope of the instance. The purpose of an attribute writer method is to be able to assign local variables to an instance variable, and the purpose of an attribute reader method is to access the value of that instance variable. 

In order for the reader method to read the variable, the variable needs to be defined. It is the job of the writer method to define the variable. A writer method looks like: 

```
def name=(name_given)
	@name = name_given 
end 
```

The writer method (name=), accepts an argument of a local variable (name_given), and sets it to be the value of the instance variable (@name). Writer methods are also referred to as “setter methods” because they set the value of an instance variable. 

To allow access to the value of this instance variable, we need an attribute reader, or getter method to ask the instance of the class what the variables value is. 

An attribute reader method looks like:

```
def name
	@name = name
end
```

The purpose of this reader method is to give us the ability to call the method Class.name on an instance of the class to return the instance variable’s value. The reader method is also called the getter method because it “gets” or retrieves the value of the instance variable. 

Now we can use the method .name to return a name of an instance of a variable, in this case let’s say on the Person class, Person.name will return the assigned string. 

An instance variable can be set to any data type. Above we defined it to a local variable because we were defining a method that can be used to pass in a data type through an argument to be set to that instance variable. In our example, if we wanted to set the value of name to a Person’s name, we would pass in a string, “Karen”, to set the value of @name = “Karen”. A string is an object that holds a group of characters — usually a word or sentence, between quotation marks. If you were to try to set your instance variable to, karen (without quotation marks, and lowercase because uncased would indicate an uninitialized constant), @name = karen, Ruby would think you were setting it to a variable of karen, which at this point is an empty variable. 

A few things can go wrong without a writer and reader method:

1)  If you do not have a writer method, and try to call an instance variable, @name, it will not give you an error message because it has a value of nil, and will return nothing. 

2)  If you do not have a reader method, and you try to call an undefined method on a class, Person.name, you will receive this error message:

```
NoMethodError:
       undefined method `name’ for Person:Class
```

3)  If you call a variable without defining it, name, or if you call a defined variable outside of the scope of the method it was defined in (hence the purpose of instance variables), it will give an undefined local variable name error:

```
NameError:
       undefined local variable or method `name’ for Person:Class
```

Readers and writers are usually defined is by an attribute accessor class method: attr_accessor. This does the job of both attribute writers and attribute readers. 

```
class Person
	attr_accessor   :name 

end
```

If you wanted the ability to read a variable (but not write),  or write a variable (but not read it), you would use the attr_reader, or attr_writer methods, respectively.




