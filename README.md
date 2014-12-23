**JavaScript Inheritance Revisited**

In one of my previous blog, I had talked about inheritance in JavaScript. The main idea of that blog was to share that how inheritance can be achieved using prototype chaining.
But the demo given in the previous blog had some serious flaws or if not a flaw then you can say a bad advice or maybe not a good way to implement inheritance.

Demo in the previous blog looked like:

```JavaScript
function LivingThing() {
}
LivingThing.prototype.move = function () {
    console.log("I am living thing! I can move!!");
};
function Bird() {
    LivingThing.call(this);
}
Bird.prototype = Object.create(LivingThing.prototype);
Bird.prototype.constructor = Bird;
Bird.prototype.fly = function () {
    console.log("I am bird! I can fly!!");
};
function Peacock() {
    Bird.call(this);
}
Peacock.prototype = Object.create(Bird.prototype);
Peacock.prototype.constructor = Peacock;
Peacock.prototype.dance = function () {
    console.log("I am Peacock! I can dance");
};
var peacock = new Peacock();
peacock.dance(); // I am Peacock! I can dance
peacock.fly(); // I am bird! I can fly!!
peacock.move(); // I am living thing! I can move!!
```

So let's discuss the problem with the above piece of code:

1. We were creating a function inside the constructor. So every time a new object would be created, it would get its own copy of that function. So suppose of there are 10 objects in the chain, then 10 functions would also be there. Let's understand more with the help of this diagram:
