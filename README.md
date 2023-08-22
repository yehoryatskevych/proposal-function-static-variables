# ECMAScript function static variables and blocks
A proposal to add a `static` keyword for variable declaration in functions.

## The problem
Allocation in JavaScript can hardly be called "fast". JavaScript is a great language, and its features significantly reduce development time, making it the best language for prototyping and fast development. However, it is still an interpreted language, which takes a lot of time for native calls and allocations. Therefore, it would be great to have more options for manual code optimization.

Let's consider an example with a function that just creates a 3D vector using some extra parameter:

```javascript
function task(i) {
    new Vector3(i + 1.13, i + 5.231, i + 7.1247);
}
```

Every time this function is called, JavaScript allocates a new object for the `Vector3` instance. We can easily measure the execution time of this function by calling it `10^8` times:

```javascript
const ITERATIONS_NUM = Math.pow(10, 8);

console.time('task');
for (let i = 0; i < ITERATIONS_NUM; ++i) {
    task(i);
}
console.timeEnd('task');
```

The result:
```Bash
task: 920.739ms
```

I don't think that's a good result. So let's test again, but move the vector allocation to the global scope:

```javascript
let vec = new Vector3(0, 0, 0);

function task(i) {
    vec.set(i + 1.13, i + 5.231, i + 7.1247);
}

const ITERATIONS_NUM = Math.pow(10, 8);

console.time('task');
for (let i = 0; i < ITERATIONS_NUM; ++i) {
    task(i);
}
console.timeEnd('task');
```

The result:
```bash
task: 78.667ms
```

Looks much better, even though we are setting the value every call. But is it reasonable to put all variables into the global context? I don't think so, especially if your goal is to write high-performance code with many functions that need to be optimized. So you can use closure with an Immediately Invoked Function Expression (IIFE), like this:
```javascript
const task = (() => {
    let vec = new Vector(0, 0, 0);
    return function task(i) {
        vec.set(i + 1.13, i + 5.231, i + 7.1247);
    }
})();
```

However, we are immediately confronted with the fact that we can't use function declarations, and hoisting no longer works for us. So we have to declare these functions before we use them, and we already get restrictions that slow down development speed and interfere with the desired structuring of the code, especially when there are a lot of such functions. Using such optimizations in methods becomes almost impossible.

## The solution
One solution to this problem could be local static variables, as implemented in some other C-like low-level languages. These variables would allow us to optimize memory allocation and function execution time by allocating and initializing the variables only once after the first function call. Subsequent function calls would use the already allocated variable with the last value. It's similar to global variables that can only be accessed from the specific function in which they are declared.

Here is an example of how it could be used:

```javascript
function task(i) {
    static let vec = new Vector(0, 0, 0);
    vec.set(i + 1.13, i + 5.231, i + 7.1247);
}
```

and a more complex example of how it should work:

```javascript
function func() {
    static let isFirstCall = true;
    static let counter = 0;

    if (isFirstCall) {
        isFirstCall = false;
        console.log('Static variables initialized!');
    }

    console.log("Counter:", counter);

    counter++;
}

func(); // OUT: "Static variables initialized!", "Counter: 0"
func(); // OUT: "Counter: 1"
func(); // OUT: "Counter: 2"
```

This code proposal introduces a highly useful feature for functions with high execution rates and constants that require complex operations for initialization. It allows functions to store their own state for subsequent executions and remain independent of the global context. With this feature, functions can efficiently handle their own state persistence and eliminate the need for global variables or external dependencies.

To make it more consistent with the current static implementation and more flexible it also can be added with the support of [class-static-block](https://github.com/tc39/proposal-class-static-block) proposal.

```javascript
function func() {
    static let counter = 0;

    static {
        console.log('Static variables initialized!');
    };

    console.log("Counter:", counter);

    counter++;
}

func(); // OUT: "Static variables initialized!", "Counter: 0"
func(); // OUT: "Counter: 1"
func(); // OUT: "Counter: 2"
```

## Additional benefits
In addition to helping with optimization, it can also be useful to store some function states independently from global scope for example for util/helper functions that need to have some state and pre-allocation, refactoring is also made easier due to the elimination of the need to move variables and "helpers" for the "helper".

```javascript
function funtionWithComplexPreallocation(value) {
    static const magic = 6816;
    static const magicOffset = 285615417;
    static const magicRatio = complexOperation(magic);
    return magicOffset + (value * magicRatio);   
}
```

or 

```javascript
function funtionWithComplexPreallocation(value) {
    static const magic = 6816;
    static const magicOffset = 285615417;
    static let magicRatio;

    static {
        for (let i = 0; i < 1024; ++i) {
            magicRatio = Math.sqrt(magicRatio) * magic;
        }
    }

    return magicOffset + (value * magicRatio);   
}
```
