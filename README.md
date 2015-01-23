#JavaScript Inheritance Revisited

In one of my previous [blogs](http://codechutney.in/blog/author/namita.malik/), I had talked about **[inheritance in JavaScript](http://codechutney.in/blog/javascript/inheritance-in-javascript/)**. The main idea of that blog was to share that how **inheritance** can be achieved using **prototype** **chaining**.

But the demo given in the [that blog](http://codechutney.in/blog/javascript/inheritance-in-javascript/) had some serious flaws or if not a flaw then you can say a bad advice or maybe not a good way to implement **inheritance**.

Demo in the [previous blog](http://codechutney.in/blog/javascript/inheritance-in-javascript/) looked like:

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

So let's discuss the problem with the above piece of code:

**First Problem**: We were defining a **function** inside the **constructor** **function**, so every time a new object would be created, it would get its own copy of that member **function**.

**Old Peacock Class:**
```JavaScript
function Peacock() {
    this.dance = function() {
      console.log("I am Peacock! I can dance");
    };
}
```

Now suppose of there are 10 Peacock objects in the chain, then 10 ```dance()``` **functions** would also be there(Each object would be having a **function** which is defined in **constructor** **function**) because we know that **functions** are data in **JavaScript**, therefore if one **function** takes 10 bytes of data then 10 objects would obviously take 100 bytes which is quite an inefficient way of doing things. See this diagrammatic representation showing what will happen if numerous ```Peacock``` objects are created:

![Function as data in per Instance.png](https://raw.githubusercontent.com/NamitaMalik/JavaScript-Inheritance-Revisited/master/Function%20as%20data%20in%20per%20Instance.png)

> NOTE: If we are defining 5 **functions** into **constructor** **function**, and each **function** takes 10 bytes then each object will take 50 bytes and 10 objects will take 10*50=500 bytes. And memory will continuously increase by 50 bytes with each object. Which is seriously bad way.

So what is the better approach? How to avoid **function** getting created with each object?

**The Solution**

Let's keep the **function** of **constructor** **function** at place which is common, so that they can be accessed by any Peacock **object**. To do this let's make this **function** **static** and to make a **function** **static** in **JavaScript**, all we have to do is put that **function** in the **prototype**.

So, instead of creating **function** ```dance()``` inside the **constructor** **function** of Peacock, create it inside the **prototype** of Peacock so it will be common in all the objects as below:

**New Peacock Class:**
```JavaScript
function Peacock() {
}
Peacock.prototype.dance = function () {
    console.log("I am Peacock! I can dance!!");
};
```

Now if we create multiple objects of Peacock class, all having same object in **prototype** property, so all the  Peacock object will get ```dance()``` method via **prototype chaining**. Now ```dance()``` method will take memory once only. :-)

**Second Problem**: When we do ```Peacock.prototype = new Bird();```, a new Bird object would be created, and store to Peacock **prototype**. So when we create new object of Peacock class, all the Peacock objects having same object in **prototype** as parent object(Bird object). As all the Peacock objects have single parent Bird object, so whatever we will change in that Bird object, will be reflect for all the child peacock object. If one child object want to update its parent property, but when it will update parent property, it will updated for all other peacock objects, which is not correct **inheritance**.

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

> NOTE: If will notice, we were updating flySpeed of peacock p1, and p2's flySpeed has also updated.

![Figure 1 - Inheritance Revisited.jpg](https://raw.githubusercontent.com/NamitaMalik/JavaScript-Inheritance-Revisited/master/Figure%201%20-%20Inheritance%20Revisited.jpg)

Above diagram shows how **inheritance** is happening through **prototype chaining** in and in addition to it, it also shows that how **function** declared in super class is available in the further sub classes also.



**Third Problem**: If we would try to update parent property via child object, It will set new property in child object instead of updating parent property.




Now, create an **object** of **prototype** and pass it in the **prototype** of next object. This can be seen in the snippet below:

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

In the above snippet, we have added **function** move() to the **prototype** of LivingThing **object**. In the **prototype** of Bird, we have passed the copy **object** of **prototype** of LivingThing **object** with the help of **Object.create**, as ```Bird.prototype = Object.create(LivingThing.prototype)```.

Similarly, in the **prototype** of Peacock **object**, we have passed the copy **object** of **prototype** of Bird with the help of **Object.create()** as ```Peacock.prototype = Object.create(Bird.prototype);```.

**Object.create** was basically takes following two arguments(second one being the optional):

1. Prototype
2. Set of properties

On the basis of **prototype** and set of properties passed **Object.create** creates a new object.

Let's us see a diagrammatic representation of the snippet given above:

![Figure 2 - Inheritance Revisited.jpg](https://raw.githubusercontent.com/NamitaMalik/JavaScript-Inheritance-Revisited/master/Figure%202%20-%20Inheritance%20Revisited.jpg)

Now, other thing that came into picture here is that suppose we ```LivingThing``` class has a property with name **food**, which is set into **Constructor** **function** as below:

```JavaScript
function LivingThing(food) {
    this.food = food;
}
```

And ```Bird``` class also has a property with named **flySpeed**, which is also set into **Constructor** **function** as below:

```JavaScript
function Bird(flySpeed) {
    this.flySpeed = flySpeed;
}
```

And ```Peacock``` class also has a property with named **color**, which is also set into **Constructor** **function** as below:

```JavaScript
function Peacock(color) {
    this.color = color;
}
```

And whenever we are creating an object of ```Peacock``` class, is should have all three properties **color**, **flySpeed** and **food**. But it would not have, because we are implementing **inheritance** with **prototype** property. And we also can not set all the properties **color**, **flySpeed** and **food** in one statement like, we pass all three properties into ```Peacock``` class as: ```var peacock = new Peacock("White", "10m/s", "snakes");```.

we passed some properties to our **Peacock** object which could not be done in the previous implementation. We have passed "White" , "10m/s" and "snakes" to our Peacock object. So how are we able to do this?

Closely see the ```Peacock``` function and notice that we have made the ```Bird``` function point to the ```Peacock``` object. We have used ```call``` here as we know that there is no ```super``` keyword in ```JavaScript``` to point to the parent constructor. Similarily, we have made ```LivingThing``` function point to the peacock object. So the gist is that we are executing ```LivingThing``` and ```Bird``` functions in ```context``` to ```Peacock``` object.

```JavaScript
// LivingThing Class
function LivingThing(food) {
    this.food = food;
}
LivingThing.prototype.move = function () {
    console.log("I am living thing! I can move!! And I eat: ", this.food);
};
// Bird Class
function Bird(flySpeed, food) {
    LivingThing.apply(this, [food]);
    this.flySpeed = flySpeed;
}
Bird.prototype = Object.create(LivingThing.prototype);
Bird.prototype.constructor = Bird;
Bird.prototype.fly = function () {
    console.log("I am bird! I can fly!! And My speed is: ", this.flySpeed);
};
// Peacock Class
function Peacock(color, flySpeed, food) {
    Bird.call(this, flySpeed, food);
    this.color = color;
}
Peacock.prototype = Object.create(Bird.prototype);
Peacock.prototype.constructor = Peacock;
Peacock.prototype.dance = function () {
    console.log("I am Peacock! I can dance!! And my Color is: ", this.color);
};
var peacock = new Peacock("White", "10m/s", "snakes");
peacock.dance(); // I am Peacock! I can dance!! And my Color is:  While
peacock.fly(); // I am bird! I can fly!! And My speed is:  10m/s
peacock.move(); // I am living thing! I can move!! And I eat:  snakes
```

There are a lot of ways to achieve a single thing, but it depends upon the need of the project and the situation that one can decide which way to adopt!