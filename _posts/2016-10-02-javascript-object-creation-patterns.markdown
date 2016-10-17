---
layout: post
title:  "JavaScript Object Creation Patterns"
date:   2016-10-02 13:0:0 +0700
categories: JavaScript
---

I'm reading *Professional JavaScript for Web* recently. It's really good, and I'm impressed by the contents of object creation patterns.

***

<br>

#### The Factory Pattern:

Developers create functions to encapsulate the creation of objects with specific interfaces.

~~~ javascript
function createPerson(name, age, job){
    var o = new Object();
    o.name = name;
    o.age = age;
    o.job = job;
    o.sayName = function(){
        alert(this.name); };
        return o;
    }
}
var person1 = createPerson(“Nicholas”, 29, “Software Engineer”);
var person2 = createPerson(“Greg”, 27, “Doctor”);
~~~

Cons:

* The factory pattern didn’t address the issue of object identification.(No prototype in the object)

***

<br>

#### The Constructor Pattern

~~~ javascript
function Person(name, age, job){
    this.name = name;
    this.age = age;
    this.job = job;
    this.sayName = function(){ alert(this.name);
    };
}
var person1 = new Person(“Nicholas”, 29, “Software Engineer”);
var person2 = new Person(“Greg”, 27, “Doctor”);
~~~

Using new to call a constructor in this manner essentially causes the following four steps to be taken:

1. Create a new object.
2. Assign the this value of the constructor to the new object (so this points to the new object).
3. Execute the code inside the constructor (adds properties to the new object).
4. Return the new object.

Each **Person** instance has a **constructor** property that points back to **Person**. Each of the objects can be indicated by using the **instanceof** operator:

~~~ javascript
alert(person1 instanceof Object); //true
alert(person1 instanceof Person); //true
alert(person2 instanceof Object); //true
alert(person2 instanceof Person); //true
~~~

Pros:

* Instances can be identified as a particular type.

Cons:

* Functions of the same name on different instances are not equivalent:

~~~ javascript
alert(person1.sayName == person2.sayName); //false
~~~

***

<br>

#### The Prototype Pattern

~~~ javascript
function Person(){
}
Person.prototype.name = “Nicholas”;
Person.prototype.age = 29;
Person.prototype.job = “Software Engineer”;
Person.prototype.sayName = function(){
    alert(this.name);
};

var person1 = new Person();
person1.sayName(); //”Nicholas”

var person2 = new Person();
person2.sayName(); //”Nicholas”

alert(person1.sayName == person2.sayName); //true
~~~

Instead of assigning object information in the constructor, they can be assigned directly to the prototype.

When a function created, a function object will be created and added to current scope, and another object will be created to represent the function object's prototype:

~~~ console
//In Chrome Console
> function Person(){
  }
< undefined
> //a new object named Person has created
  //a new object represents Person's prototype has created
  this.Person;
< function Person(){
  }
> Person.prototype;
< ▶Object {}
> Person.prototype.constructor; //Person.prototype.constructor points to Person
< function Person(){
  }
> var person1 = new Person();
< undefined
> var person2 = new Person();
< undefined
> //There is no standard way to access [[Prototype]] from script,
  //but Chrome support a property on every object called __proto__
  person1.__proto__ == Person.prototype;
< true
> person2.__proto__ == Person.prototype;
< true
> //all the instances of Person share the same prototype
~~~

Whenever a property is accessed for reading on an object, a search is started to find a property with that name. The search begins on the object instance itself. If a property with the given name is found on the instance, then that value is returned; if the property is not found, then the search continues up the pointer to the prototype, and the prototype is searched for a property with the same name. If the property is found on the prototype, then that value is returned.

Although it’s possible to read values on the prototype from object instances, it is not possible to overwrite them. If you add a property to an instance that has the same name as a property on the prototype, you create the property on the instance, which then masks the property on the prototype.

~~~ javascript
function Person(){
}

Person.prototype.name = “Nicholas”;

var person1 = new Person();
var person2 = new Person();

alert(person1.hasOwnProperty(“name”)); //false
alert(“name” in person1);  //true

person1.name = “Greg”;
alert(person1.name);   //”Greg” - from instance
alert(person1.hasOwnProperty(“name”)); //true
alert(“name” in person1);  //true

alert(person2.name);   //”Nicholas” - from prototype
alert(person2.hasOwnProperty(“name”)); //false
alert(“name” in person2);  //true

delete person1.name;   //delete name property from instance
alert(person1.name);   //”Nicholas” - from the prototype
alert(person1.hasOwnProperty(“name”)); //false
alert(“name” in person1);  //true
~~~

Pros:

* All properties on the prototype are shared among instances, which is ideal for functions.

Cons:

* It negates the ability to pass initialization arguments into the constructor, meaning that all instances get the same property values by default.
* Properties that contain primitive values are also shared among instances.

***

<br>

#### Combination Constructor/Prototype Pattern

~~~ javascript
function Person(name, age, job){
    this.name = name;
    this.age = age;
    this.job = job;
    this.friends = [“Shelby”, “Court”];
}

Person.prototype = {
    constructor: Person,
    sayName : function () {
        alert(this.name);
    }
};

var person1 = new Person(“Nicholas”, 29, “Software Engineer”);
var person2 = new Person(“Greg”, 27, “Doctor”);

person1.friends.push(“Van”);

alert(person1.friends); //”Shelby,Court,Van”
alert(person2.friends); //”Shelby,Court”
alert(person1.friends === person2.friends); //false
alert(person1.sayName === person2.sayName); //true
~~~

The hybrid constructor/prototype pattern is the most widely used and accepted practice for defining custom reference types in ECMAScript. Generally speaking, this is the default pattern to use for defining reference types.

***

<br>

#### Dynamic Prototype Pattern

This pattern is used to solve the separation between the constructor and the prototype.

~~~ javascript
function Person(name, age, job){

    //properties
    this.name = name;
    this.age = age;
    this.job = job;

    //methods
    if (typeof this.sayName != “function”){
        Person.prototype.sayName = function(){
            alert(this.name);
        };
    }
}

var friend = new Person(“Nicholas”, 29, “Software Engineer”); friend.sayName();
~~~

The if block is necessary, otherwise the Person.prototype.sayName will create a new function object when a new instance is initialized.

***

<br>

#### Parasitic Constructor Pattern

~~~ javascript
function Person(name, age, job){
    var o = new Object();
    o.name = name;
    o.age = age;
    o.job = job;
    o.sayName = function(){
        alert(this.name);
    };
    return o;
}

var friend = new Person(“Nicholas”, 29, “Software Engineer”);

friend.sayName(); //”Nicholas”
~~~

This pattern is exactly the same as the factory pattern, except that the function is called as a constructor, using the new operator:

~~~ javascript
// The Factory Pattern
var person = createPerson(“Nicholas”, 29, “Software Engineer”);

// Parasitic Constructor Pattern
var person = new Person(“Nicholas”, 29, “Software Engineer”);
~~~

Cons:

* You cannot rely on the **instanceof** operator to indicate the object type, since the constructor returns a new object instead the function itself.

***

<br>

#### Durable Constructor Pattern

Durable objects are best used in secure environments (those that forbid the use of this and new) or to protect data from the rest of the application (as in mashups).

A durable constructor is similar to the parasitic constructor pattern, with two differences:

1. Instance methods on the created object don’t refer to **this**.
2. The constructor is never called using the **new** operator.

~~~ javascript
function Person(name, age, job){
    //create the object to return
    var o = new Object();
    //optional: define private variables/functions here

    //attach methods
    o.sayName = function(){
        alert(name);
    };

    //return the object
    return o;
}

var friend = Person(“Nicholas”, 29, “Software Engineer”);
friend.sayName(); //”Nicholas”
// The friend variable is a durable object, and there is no way
// to access any of its data members without calling a method.
~~~

Pros:

* This pattern can protect data.

Cons:

* **instanceof** will not work.

***

<br>
