# Callbacks, Asynchrony and Javascript

### Callbacks
Callbacks are simply functions that are passed as arguments to some function which calls the "call back" function at some time.
```javascript
function foo(somearg, callback){
  console.log(somearg);
  // ... maybe other stuff
  callback(); 
  // .. maybe other stuff
}

// callback function 
function cb(){
  console.log('I am the callback');
}

// calling our foo function that takes two arguments,
// one of them is our callback function,
// (reference to the callback function, to be precise)
foo('i am just an argument', cb);

// or we can implement foo() like this
// foo('i am just an argument', () => {
//   console.log('i am the callback.');
// });
```
Here, cb() is our callback function which is "called back" in other function called foo().One of the arguments foo() takes is callback that is reference to cb() which is called after some lines of code in our example. 
Now lets see why we need callback functions.

### Why do we need callbacks?
Lets say we want some action to happen when we finish some task. For instance, we want to upload ourphoto and post it. The sequence of this operation is: upload photo first and then post it. How can we achieve it?
```javascript
function uploadImage(img, cb) {
console.log("Uploading image...");
// do some stuff to upload image
// ...
console.log("Image uploaded.");
cb(img);
// ...
}

// callback function
function postImage(img) {
console.log("Posting image: ", img);
}

uploadImage("/path/to/image.jpg", postImage);
```
We need to call postImage() after uploading the image but we dont know when exactly the image upload completes. Thats why we let uploadImage() know to call our callback after it does some image upload stuff.
But, can’t we just call the postImage() function(callback) without passing it, just like calling another function inside a function?? 
```javascript
function uploadImage(img) {
console.log("Uploading image...");
// do some stuff to upload image
// ...
console.log("Image uploaded.");
postImage(img);
// ...
}

// callback function
function postImage(img) {
console.log("Posting image: ", img);
}

uploadImage("/path/to/image.jpg");
```
Yes, you could have done it, if you wrote uploadImage() yourself. If it was written by someone else or it is a part of a library, you could have been allowed to pass the callback that takes one argument(img). For example: map() method in Javascript takes a callback with three arguments(More on this: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map).

### Synchronous and Asynchronous callbacks
Every example we have seen till now, we have used synchronous callbacks.It means we know when our callback is going to be called. In previous example, we know cb(img) gets executed after console.log("Image uploaded.").And one important thing to note here is, synchronous callbacks return value(in our example we didn't explicitly return anything, however). It means everything waits untill the function returns.This has very significant implications in single threaded language like Javascript.
Javascript is single threaded, which means it has one call stack. Call stack is where functions get pushed and popped off for execution. We don't want to fill our call stack with loads of time consuming functions(CPU intensive tasks like image processing, I/O request, etc) at the same time. But Javascript is a language for the web. What's the point of it if it  doesn't handle network requests efficiently since it only has one call stack? Does one call stack mean the user has to wait for 10 seconds to upload photo, meanwhile staring at the screen because nothing works untill the image is uploaded? Why implement just one call stack then, are Javascript creators that stupid?
All these questions can be answered by one function: setTimeout().

setTimeout() takes one callback and minimum time(in milliseconds) after which that callback is to be executed. More on why i said 'minimum' time later.
And one thing, setTimeout() is not a Javascript function. Its not present in the source code of JS engines like V8. -What?
Yeah, it is a web api(exposed by browser). We will talk about this later.
```javascript
function foo() {
console.log("Before calling setTimeout().");
setTimeout(() => {
console.log("Log me after 3 seconds.");
}, 3000);
console.log("After calling setTimeout().");
}

foo();
```
We got output in the sequence:
Before calling setTimeout(). 
After calling setTimeout().
Log me after 3 seconds.

More questions?
Before answering all these questions, I want to introduce next very important thing in Javascript called 'event loop'. In short, event loop pushes a callback from callback queue if our call stack is empty. Thats it! Checkout this awesome talk on event loop: (https://www.youtube.com/watch?v=8aGhZQkoFbQ). Callback queue is where our callbacks get queued, not the synchronous callbacks, but essentially the callback we passed to setTimeout().

foo() is pushed into the call stack. In function foo, console.log('Before calling setTimeout().') executes first because it is pushed into the call stack and returns immediately logging the output(not much work!).Simple. When setTimeout() is called, it is also pushed into the call stack. But since setTimeout() is our special function, it gets some special treatement. It gets popped off immediately and the callback it takes is passed to the web api- not pushed to our call stack!!
Web apis are provided by browsers(For eg, DOM, XMLHttpRequest, etc). So after 3 seconds, the web api sends the callback to the callback queue. Then the event loop picks the callback from callback queue and execute it in call stack if the call stack is empty. If it is not empty, it waits. Therefore, our callback takes 'minimum' of 3 seconds to execute. It might take more than 3 seconds due to the fact that call stack might not be empty when event loop picks the callback from callback queue.
So in our example, console.log('After calling setTimeout().') executes after setTimeout() is popped off. Meanwhile our callback goes through web api, then callback queue and finally picked up by event loop to get pushed and executed in the call stack.So console.log('Log me after 3 seconds.') executes last although the sequence of the program tells us otherwise. This type of callback is called Asynchronous callback.
Javascript runtime is shown in the figure below:
![Javascript runtime](https://raw.githubusercontent.com/rohit120582sharma/Documentation/master/images/runtime.png)

Asynchronous callbacks run on another thread(access to threads provided by browser) after the function(setTimeout()) returns.But synchronous callbacks run before the function(eg: uploadImage()) returns.
One of the reasons Javascript is single threaded is, complexity - single thread means less complexity. Another reason is, Javascript was intended to do short and quick tasks, initially.

### Lessons learned
Don't pile the call stack with useless time consuming stuff. Javascript is useful for I/O but not for CPU intensive tasks because more CPU time means more time spent by functions in call stack which means event loop cannot push callbacks to the call stack. 
Another thing is, we need to know what type of callbacks we are using. Its developer's responsiblity to know how the callback needs to be implemented as per the api documentation. For example: Node.js implements error first callbacks.

