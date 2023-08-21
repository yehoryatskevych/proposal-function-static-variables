# ECMAScript function static variables and blocks
A proposal to add a `static` keyword for variable declaration in functions.

## Motivation
Locally defined `static` variables maybe be useful to optimize memory allocation and function execution time which is achieved with single variable allocation and initialization. Variables with `static` keywords will be allocated and initialized only once after the first function call, next function calls will use the already allocated variable with the last value. It is similar to the `static` keyword in `C++` and can be described as a global variable that can be accessed only from the specific function it was declared.

Here is an example how it can be used:
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

func(); // OUT: "Static var a allocated!", "counter: 0"
func(); // OUT: "counter: 1"
func(); // OUT: "counter: 2"
```

This is extremely useful for functions with high execution rate and function with constants that require a complex operation for the initialization that only should be called once. So functions can store their own state for the next executions and be independent of the global context.

## Static blocks
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

func(); // OUT: "Static var a allocated!", "counter: 0"
func(); // OUT: "counter: 1"
func(); // OUT: "counter: 2"
```
