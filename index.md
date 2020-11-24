#JavaScript Inheritance Revisited

In one of my previous [blogs](https://namitamalik.github.io/), I had talked about **[inheritance in JavaScript](http://namitamalik.github.io/Inheritance-in-JavaScript/)**. The main idea of that blog was to share that how **inheritance** can be achieved using **prototype** **chaining**.

But the demo given in the [that blog](http://namitamalik.github.io/Inheritance-in-JavaScript/) had some serious flaws or if not a flaw then you can say a bad advice or maybe not a good way to implement **inheritance**.

Demo in the [previous blog](http://namitamalik.github.io/Inheritance-in-JavaScript/) on **inheritance** looked like:

```JavaScript
function LivingThing() {
    this.move = function() {
        console.log("I am living thing! I can move!!");
    };
}
function Bird() {
    this.fly = function() {
        console.log("I am bird! I can fly!!");
    };
}
function Peacock() {
    this.dance = function() {
      console.log("I am Peacock! I can dance");
    };
}
Bird.prototype = new LivingThing();
Bird.prototype.constructor = Bird;
Peacock.prototype = new Bird();
Peacock.prototype.constructor = Peacock;
var peacock = new Peacock("A");
peacock.dance(); // I am Peacock! I can dance
peacock.fly(); // I am bird! I can fly!!
peacock.move(); // I am living thing! I can move!!
```

So let's first discuss the problems with the above piece of code then we will discuss the solution!

**First Problem**: We were defining a **function** inside the **constructor** **function**, so every time a new object would be created, it would get its own copy of that member **function**.

**Old Peacock Class:**
```JavaScript
function Peacock() {
    this.dance = function() {
      console.log("I am Peacock! I can dance");
    };
}
```

Now suppose of there are 10 Peacock objects in the chain, then 10 ```dance()``` **functions** would also be there(Each object would be having a ```dance()``` **function** which is defined in **constructor** **function**) because we know that **functions** are data in **JavaScript**, therefore if one **function** takes 10 bytes of data then 10 objects would obviously take 100 bytes which is quite an inefficient way of doing things. See this diagrammatic representation showing what will happen if numerous ```Peacock``` objects are created:

![Function as data in per Instance.png](https://raw.githubusercontent.com/NamitaMalik/JavaScript-Inheritance-Revisited/master/Function%20as%20data%20in%20per%20Instance.png)

> NOTE: If we are defining 5 **functions** into **constructor** **function**, and each **function** takes 10 bytes then each object will take 50 bytes and 10 objects will take 10*50=500 bytes. And memory will continuously increase by 50 bytes with each object, which is seriously a bad way.

**Second Problem**: When we do ```Peacock.prototype = new Bird();```, a new Bird object would be created, and stored to Peacock **prototype**. So all the Peacock objects are now having same object in **prototype** as parent object(Bird object). As all the Peacock objects have single parent Bird object, so whatever we will change in that Bird object, will be reflect for all the child peacock objects. If one child object will update any parent property, it will be updated for all other child peacock objects, which is not correct **inheritance**.

```JavaScript
function Bird() {
    this.birdProperty = {
        flySpeed:'20m/s',
        maxHeight:'5km'
    };
}
function Peacock() {
}
Peacock.prototype = new Bird();
Peacock.prototype.constructor = Peacock;
var p1 = new Peacock();
var p2 = new Peacock();
console.log("p1's parent Properties", p1.birdProperty); // { flySpeed: '20m/s', maxHeight: '5km' }
console.log("p2's parent Properties", p2.birdProperty); // { flySpeed: '20m/s', maxHeight: '5km' }
p1.birdProperty.flySpeed = '30m/s';
console.log("p1's parent Properties", p1.birdProperty); // { flySpeed: '30m/s', maxHeight: '5km' }
console.log("p2's parent Properties", p2.birdProperty); // { flySpeed: '30m/s', maxHeight: '5km' }
```

> NOTE: Here we can notice that we were updating flySpeed of peacock p1, and peacock p2's flySpeed has been updated too.

**Third Problem**: If we would try to update parent property via child object, and if property is primitive then it will create a new property in child object instead of updating parent property, and same thing will apply to object if we assign new object. When we assign new object in any variable then it will create a new variable in child object and assign reference of that newly created object into that newly created variable(So there will be two variables with same name, one in parent Bird object and second in child Peacock object so it may give you wrong/unexpected results.). But if you are updating any property of object value, then it will get updated in parent object property(As discussed in Second Problem.).

**Fourth Problem**: Suppose ```LivingThing``` class has a property with name **food**, which is set into **Constructor** **function** as below:

```JavaScript
function LivingThing(food) {
    this.food = food;
}
```

And ```Bird``` class also has a property named **flySpeed**, which is also set into **Constructor** **function**, and user can also pass **food** property along with **flySpeed** to set, as Bird is child class of ```LivingThing``` So it should have **food** property as well:

```JavaScript
function Bird(food, flySpeed) {
    // How to set food property as it is declare in parent class ??
    this.flySpeed = flySpeed;
}
```

And ```Peacock``` class also has a property named **color**, which is also set into **Constructor** **function** and user can also pass **food** and  **flySpeed** properties along with **color**, as Peacock is child class of ```Bird``` so it should have **food** and **flySpeed** properties too.

```JavaScript
function Peacock(food, flySpeed, color) {
    // How to set food and flySpeed properties as it is declared in parent class ??
    this.color = color;
}
```

And whenever user is creating an object of ```Peacock``` class, he will be assuming that he will pass all three properties **color**, **flySpeed** and **food**(e.g. ```var peacock = new Peacock("White", "10m/s", "snakes")```) and all of these properties will be set. But that is not the case is happening here.

![Figure 1 - Inheritance Revisited.jpg](https://raw.githubusercontent.com/NamitaMalik/JavaScript-Inheritance-Revisited/master/Figure%201%20-%20Inheritance%20Revisited.jpg)

Above diagram shows how **inheritance** is happening through **prototype chaining** in and in addition to it, it also shows that how **function** declared in super class is available in the further sub classes also.

So what is the better approach? How to avoid **function** getting created with each object? How to implement right **inheritance**? And how to solve all above problems?

**The Solution**:

**First Solution**: Let's keep the **function** of **constructor** **function** at place which is common, so that they can be accessed by all the Peacock **object**. To do this let's make this **function** **static** and to make a **function** **static** in **JavaScript**, all we have to do is put that **function** in the **prototype**.

So, instead of creating **function** ```dance()``` inside the **constructor** **function** of Peacock, create it inside the **prototype** of Peacock so it will be common for all the objects.

**New Peacock Class:**
```JavaScript
function Peacock() {
}
Peacock.prototype.dance = function () {
    console.log("I am Peacock! I can dance!!");
};
```

Now if we create multiple objects of Peacock class then all the Peacock objects will be having same object in **prototype** property, so all the  Peacock object will get ```dance()``` method via **prototype chaining**. Now ```dance()``` method will take memory once only. :-)

> NOTE: With the help of this solution, our **First Problem** will be solved. :-)

**Second Solution**: Create an **object** of **prototype** and pass it in the **prototype** of next object. So **Inheritance** will perform for **static**/ **prototype** members. This can be seen in the snippet below:

```JavaScript
// LivingThing Class
function LivingThing() {
}
LivingThing.prototype.move = function () {
    console.log("I am living thing! I can move!!");
};
// Bird Class
function Bird() {
}
Bird.prototype = Object.create(LivingThing.prototype);
Bird.prototype.constructor = Bird;
Bird.prototype.fly = function () {
    console.log("I am bird! I can fly!!");
};
// Peacock Class
function Peacock() {
}
Peacock.prototype = Object.create(Bird.prototype);
Peacock.prototype.constructor = Peacock;
Peacock.prototype.dance = function () {
    console.log("I am Peacock! I can dance!!");
};
var peacock = new Peacock("White", "10m/s", "snakes");
peacock.dance(); // I am Peacock! I can dance!!
peacock.fly(); // I am bird! I can fly!!
peacock.move(); // I am living thing! I can move!!
```

In the above snippet, we have passed the copy of **prototype** **object** of LivingThing class in the **prototype** of Bird class, with the help of **Object.create**, as ```Bird.prototype = Object.create(LivingThing.prototype)```.

Similarly, in the **prototype** of Peacock class, we have passed the copy of **prototype** **object** of Bird class with the help of **Object.create()** as ```Peacock.prototype = Object.create(Bird.prototype);```.

**Object.create** was basically takes following two arguments(second one being the optional):

1. Prototype
2. Set of properties

On the basis of **prototype** and set of properties passed **Object.create** creates a new object.

Let's us see a diagrammatic representation of the snippet given above:

![Figure 2 - Inheritance Revisited.jpg](https://raw.githubusercontent.com/NamitaMalik/JavaScript-Inheritance-Revisited/master/Figure%202%20-%20Inheritance%20Revisited.jpg)

**Third Solution**: We passed some properties to our **Peacock** object which could not be done in the previous implementation. What if we passed "White" , "10m/s" and "snakes" to our Peacock object?

For this problem, we have to call parent class **constructor** into child class **constructor** like other languages. In other languages, when we call parent class **constructor** from child class **constructor**, then parent class **constructor** is called with same object/reference of child class object. So we have to take care of both the things - Calling parent class **constructor** into child class **constructor** and calling the parent class **constructor** with the reference of child class object only. Calling parent class **constructor** into child class **constructor** is very easy and for calling parent class **constructor** with same child class object's reference, we can use **JavaScript** delegation feature( **call**/ **apply**).

```JavaScript
// LivingThing Class
function LivingThing(food) {
    this.food = food;
}
LivingThing.prototype.move = function () {
    console.log("I am living thing! I can move!! And I eat: ", this.food);
};
// Bird Class
function Bird(food, flySpeed) {
    LivingThing.apply(this, [food]);
    this.flySpeed = flySpeed;
}
Bird.prototype = Object.create(LivingThing.prototype);
Bird.prototype.constructor = Bird;
Bird.prototype.fly = function () {
    console.log("I am bird! I can fly!! And My speed is: ", this.flySpeed);
};
// Peacock Class
function Peacock(food, flySpeed, color) {
    Bird.call(this, food, flySpeed);
    this.color = color;
}
Peacock.prototype = Object.create(Bird.prototype);
Peacock.prototype.constructor = Peacock;
Peacock.prototype.dance = function () {
    console.log("I am Peacock! I can dance!! And my Color is: ", this.color);
};
var peacock = new Peacock("snakes", "10m/s", "While");
peacock.dance(); // I am Peacock! I can dance!! And my Color is:  While
peacock.fly(); // I am bird! I can fly!! And My speed is:  10m/s
peacock.move(); // I am living thing! I can move!! And I eat:  snakes
```

Closely see the ```Peacock``` function and notice that we have made the ```Bird``` function point to the ```Peacock``` object. We have used ```call``` here as we know that there is no ```super``` keyword in ```JavaScript``` to point to the parent **constructor**. Similarily, we have made ```LivingThing``` function point to the peacock object. So the gist is that we are executing ```LivingThing``` and ```Bird``` functions in ```context``` to ```Peacock``` object only.

> NOTE: With the help of second and third solution, our remaining Problem will solve. :-)

There are a lot of ways to achieve a single thing, but it depends upon the need of the project and the situation that one can decide which way to adopt!

Follow Me
---
[Github](https://github.com/NamitaMalik)

[Twitter](https://twitter.com/namita13_04)

[LinkedIn](https://in.linkedin.com/in/namita-malik-a7885b23)

[More Blogs By Me](https://namitamalik.github.io/)