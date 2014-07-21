---
layout: post
title:  Variable Scope in Ruby
date:   2014-06-22
categories: culture
---

Variables are pointers which tell the computer the memory location that holds the associated data. In Ruby, variables are divided into five different types.

<h3>Ruby Global Variable</h3>

Global variables in Ruby begin with $. Global variables have the largest scope, can be written, used and referenced anywhere in the program. The use of global variable in Ruby is rare, because it breaks the rule of encapsulation; global variables can be too easily changed by any class or methods, making it inconsistent with the overall design concept of Ruby Object Oriented programming.
    
<h3>Ruby Constants</h3>

Constants begin with an uppercase letter, they are defined within a class or module. Constants in ruby cannot be modified by other methods once it’s declared, and they cannot be declared from within methods. A single constant name can only be assigned to one group of data, and these data and constant name stays consistent throughout the entire program.

<h3>Ruby Class Variable</h3>

Class variables in Ruby begin with @@. They must be initialized before any attempt to use it. Class variables can be accessed anywhere in the particular class, they are shared by all descendants of the class. Class variables need to be treated with care; because if a subclass attempts to change the attributes, it also effect the attribute of the base class. Class variables are usually used for things such as system configuration, when the changes need to apply to all the objects in the class hierarchy tree.

<h3>Ruby Instance Variable</h3>

Instance variable in Ruby begin with @. Instance variable lives within a class instance, and they can only be referenced by class methods. Initially instance variable can only be read/write from within the class; they can be read/write outside of the class if getter method and setter method are defined.

<h3>Ruby Local Variable</h3>

Local variable in Ruby begin with lower case letter. They have the smallest scope. The information stored in local variable is only accessible from within the particular method or blocks that contains it. Outside the method, it can be assigned completely different data set. The reference to the local variable ends once the current scope ends.

Questions? Email me at stephensxub@gmail.com.

<a href="{{ site.url }}">Back Home</a>