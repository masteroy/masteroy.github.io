---
layout: post
title:  "JavaScript Inheritance"
date:   2016-10-18 0:50:0 +0700
categories: JavaScript
---

Inheritance in JavaScript is quite different from that in OO programming language. *Implementation inheritance* is the only type of inheritance supported by JavaScript.

***

<br>

#### Prototype Chaining

~~~ javascript
function SuperType(){
    this.property = true;
}

SuperType.prototype.getSuperValue = function(){
    return this.property;
};

var superTypeInstance = new SuperType();

function SubType(){
    this.subproperty = false;
}

//inherit from SuperType
SubType.prototype = new SuperType();
SubType.prototype.getSubValue = function (){
    return this.subproperty;
};

var subTypeInstance = new SubType();
alert(instance.getSuperValue()); //true
~~~

Let's see what happen step by step.

~~~ javascript
function SuperType(){
    this.property = true;
}

SuperType.prototype.getSuperValue = function(){
    return this.property;
};
~~~

**SuperType Prototype** is created and a new function **getSuperValue** is added to **SuperType Prototype**.

![1](/assets/javascript-inheritance-1.png)

~~~ javascript
var superTypeInstance = new SuperType();
~~~

An instance of **SuperType**, and its **[[prototype]]** points to **SuperType Prototype**.

![2](/assets/javascript-inheritance-2.png)

~~~ javascript
function SubType(){
    this.subproperty = false;
}
~~~

**SubType Prototype** is created.

![3](/assets/javascript-inheritance-3.png)

~~~ javascript
SubType.prototype = new SuperType();
SubType.prototype.getSubValue = function (){
    return this.subproperty;
};
~~~

**SubType Prototype** points to a new **SuperType Object**, and adds a new function **getSubValue** to **SubType Prototype**.

Notice that the new **SubType Prototype** has the same properties as **SuperTypeInstance**, except function **getSubValue**. It's reasonable because both are created by **new SuperType()**.

To compare the previous **SubType Prototype** and the new **SubType Prototype**, there is no **constructor** in the new **SubType Prototype**. This confused me at first, since other OO language's class has its own constructor. Obviously, I misunderstood the concept, and **constructor** is useless in prototype, you can find more details here: [Why is it necessary to set the prototype constructor?][why-is-it-necessary-to-set-the-prototype-constructor]

![4](/assets/javascript-inheritance-4.png)

Let's make it simpler:

![5](/assets/javascript-inheritance-5.png)

~~~ javascript
var subTypeInstance = new SubType();
~~~

Here, an instance of **SubType** is created. There is an expression **this.subproperty = false;** in **SubType**'s constructor function, so **subTypeInstance** has a property **subproperty** and value **false**.

![6](/assets/javascript-inheritance-6.png)

Actually, every object in JavaScript inherits from **Object**, and **SuperType** has a property **[[Prototype]]** points to **Object**

![7](/assets/javascript-inheritance-7.png)

Let's make it more concise:

![8](/assets/javascript-inheritance-8.png)

Aha! It is what Prototype Chaining exactly is. Prototype Chaining is an important concept in JavaScript Inheritance, so I try to make sure that I understand as much details as I could.

Cons:

* When implementing inheritance using prototypes, the prototype actually becomes an instance of another type, meaning that what once were instance properties are now prototype properties. (**property** in **SubType** once were **SuperType** instance property, but now are **SubType** prototype property now, and **property** is shared by all **SubType** instances)
![9](/assets/javascript-inheritance-9.png)
* You cannot pass arguments into the **supertype** constructor when the **subtype** instance is being created.

***

<br>

### Constructor Stealing (Object Masquerading or Classical Inheritance)

~~~ javascript
function SuperType(){
    this.colors = ["red", "blue", "green"];
}
function SubType(){
    //inherit from SuperType
    SuperType.call(this);
}

var instance1 = new SubType();
instance1.colors.push("black");
alert(instance1.colors); //"red,blue,green,black"

var instance2 = new SubType();
alert(instance2.colors); //"red,blue,green"
~~~

By using the call() method (or alternately, apply()), the **SuperType** constructor is called in the context of the newly created instance of **SubType**.
![10](/assets/javascript-inheritance-10.png)
Notice that, the instance of **SubType** is not an instance of **SuperType**.

~~~ javascript
SubType.prototype.isPrototypeOf(instance1); //true
SubType.prototype.isPrototypeOf(instance2); //true
SuperType.prototype.isPrototypeOf(instance1); //false
SuperType.prototype.isPrototypeOf(instance2); //false
~~~

One advantage that *constructor stealing* offers over *prototype chaining* is the ability to pass arguments into the **supertype constructor** from within the **subtype constructor**.

~~~ javascript
function SuperType(name){
    this.name = name;
}

function SubType(){
    //inherit from SuperType passing in an argument
    SuperType.call(this, "Nicholas");
    //instance property
    this.age = 29;
}

var instance = new SubType();
alert(instance.name); //"Nicholas";
alert(instance.age); //29
~~~

Cons:

* Methods must be defined inside the constructor, so there’s no function reuse. Furthermore, methods defined on the supertype’s prototype are not accessible on the subtype, so all types can use only the constructor pattern.

***

<br>

### Combination Inheritance (Pseudoclassical Inheritance)

*Combination inheritance* combines *prototype chaining* and *constructor stealing* to get the best of each approach.

~~~ javascript
function SuperType(name){
    this.name = name;
    this.colors = ["red", "blue", "green"];
}

SuperType.prototype.sayName = function(){
    alert(this.name);
};

function SubType(name, age){
    //inherit properties
    SuperType.call(this, name);
    this.age = age;
}

//inherit methods
SubType.prototype = new SuperType();
SubType.prototype.sayAge = function(){
    alert(this.age);
};

var instance1 = new SubType("Nicholas", 29);
instance1.colors.push("black");
alert(instance1.colors); //"red,blue,green,black"
instance1.sayName(); //"Nicholas";
instance1.sayAge(); //29

var instance2 = new SubType("Greg", 27);
alert(instance2.colors); //"red,blue,green"
instance2.sayName(); //"Greg";
instance2.sayAge(); //27
~~~

![11](/assets/javascript-inheritance-11.png)

*Combination inheritance* is the most frequently used inheritance pattern in JavaScript. It also preserves the behaviour of **instanceof** and **isPrototypeOf()** for identifying the composition of objects.

***

<br>

### Prototypal Inheritance

~~~ javascript
function object(o){
    function F(){}
    F.prototype = o;
    return new F();
}

var person = {
    name: "Nicholas",
    friends: ["Shelby", "Court", "Van"]
};
var anotherPerson = object(person);
anotherPerson.name = "Greg";
anotherPerson.friends.push("Rob");

var yetAnotherPerson = object(person);
yetAnotherPerson.name = "Linda";
yetAnotherPerson.friends.push("Barbie");

alert(person.friends); //"Shelby,Court,Van,Rob,Barbie"
~~~

The code above made me quite confused at first. I think **anotherPerson** and **yetAnotherPerson** share the same properties with **person**, because **name** and **friends** can be found in the prototype chain scope. But I was wrong, and I found an awesome article which can solve my problems: [JavaScript Prototypal Inheritance][why-is-it-necessary-to-set-the-prototype-constructor]

So, the prototype chain looks like this:
![12](/assets/javascript-inheritance-12.png)

ECMAScript 5 formalized the concept of *prototypal inheritance* by adding the **Object.create()** method. So the code can be simplified:

~~~ javascript
var person = {
    name: "Nicholas",
    friends: ["Shelby", "Court", "Van"]
};

var anotherPerson = Object.create(person);
anotherPerson.name = "Greg";
anotherPerson.friends.push("Rob");

var yetAnotherPerson = Object.create(person);
yetAnotherPerson.name = "Linda";
yetAnotherPerson.friends.push("Barbie");

alert(person.friends); //"Shelby,Court,Van,Rob,Barbie"
~~~

*Prototypal inheritance* is useful when there is no need for the overhead of creating separate constructors, but you still need an object to behave similarly to another.

***

<br>

### Parasitic Inheritance

The idea behind *parasitic inheritance* is similar to that of the *parasitic constructor* and *factory patterns*: create a function that does the inheritance, augments the object in some way, and then returns the object as if it did all the work.

~~~ javascript
function object(o){
    function F(){}
    F.prototype = o;
    return new F();
}

function createAnother(original){
    var clone = object(original); //create a new object by calling a function
    clone.sayHi = function(){     //augment the object in some way
        alert("hi");
    };
    //return the object
    return clone;
}

var person = {
    name: "Nicholas",
    friends: ["Shelby", "Court", "Van"]
};

var anotherPerson = createAnother(person);
anotherPerson.sayHi(); //"hi"
~~~

Pros:

* *Parasitic inheritance* is another pattern to use when you are concerned primarily with objects and not with custom types and constructors.

Cons:

* Adding functions to objects using *parasitic inheritance* leads to inefficiencies related to function reuse.

***

<br>

### Parasitic Combination Inheritance

*Combination Inheritance* is the most often-used pattern for inheritance but it has its own inefficiencies. The most inefficient part of the pattern is that the supertype constructor is always called twice: once to create the subtype’s prototype, and once inside the subtype constructor.

Consider the previous *Combination Inheritance* example:
![11](/assets/javascript-inheritance-11.png)

As you can see, there are two sets of **name** and **colors** properties: one on the instance and one on the **SubType prototype**. This is the result of calling the **SuperType constructor** twice.

*Parasitic Combination Inheritance* is a way to solve this problem.

~~~ javascript
function inheritPrototype(subType, superType){
    var prototype = object(superType.prototype);    //create object
    prototype.constructor = subType;                //augment object
    subType.prototype = prototype;                  //assign object
}

function SuperType(name){
    this.name = name;
    this.colors = ["red", "blue", "green"];
}

SuperType.prototype.sayName = function(){
    alert(this.name);
};

function SubType(name, age){
    SuperType.call(this, name);
    this.age = age;
}

inheritPrototype(SubType, SuperType);

SubType.prototype.sayAge = function(){
    alert(this.age);
};

var subTypeInstance = new Subtype("James", 26);
~~~

The basic idea is this: instead of calling the supertype constructor to assign the subtype’s prototype, all you need is a copy of the supertype’s prototype.

![13](/assets/javascript-inheritance-13.png)

It's perfect now and the prototype chain is kept intact, so both **instanceof** and **isPrototypeOf()** behave as they would normally.


[why-is-it-necessary-to-set-the-prototype-constructor]: http://stackoverflow.com/questions/8453887/why-is-it-necessary-to-set-the-prototype-constructor
[javascript-prototypal-inheritance]:http://stackoverflow.com/a/14049482
