# java-node

Native interface between Java and Node (v4.x) library

Author: Christopher J. Brody

License: MIT

I can hereby testify that this project is completely my own work and not subject to agreements with any other parties.
In case of code written under direct guidance from sample code the link is given for reference.
In case I accept contributions from any others I will require CLA with similar statements.
I may offer this work under other licensing terms in the future.

NOTE: This project is under development and should be considered experimental.
API is subject to change and some optimizations are expected to be required.

Highlights:
- Java-driven startup
- Static function callback Javascript --> Java
- Callback Java --> Javascript

Status:
- Builds on OSX (TODO put in shell script)
- Java-driven startup working
- JS --> Java static function call using reflection working with support for _integer_ parameters and return value
- Java --> JS callback now working with support for _integer_ parameters and return value

TODO:
- build and test on Linux
- String, array, "double-precision" (32-bit floating point), and simple Object parameters and return value JS <--> Java
- JS --> Java with function return value
- JNI efficiency ref: http://www.ibm.com/developerworks/library/j-jni/
- Support build on Linux
- Use proper Java package IDs
- Java API to get function call parameters using a real object instead of a long (64-bit) pointer handle

For future consideration:
- True object factory on the Java side
- Non-static function calls JS --> Java
- Deal with 64-bit numbers (not truly supported by JS)
- integration with node-java which provides nice support for non-static Javascript --> Java function calls
- Windows
- Also target JXCore
- Support mobile platforms (with help from JXCore)
- Support throwing exceptions in both directions JS <--> Java

External requirements:
- Java JDK (tested with Oracle Java JDK 8)
- Build tools
- `node-gyp`

Externally fetched:
- Node.js (4.1.2) source

## Usage

### Start from Java

To start Node.js library from Java:

```Java
public class MyJavaNodeProgram {
  public static void main(String[] args) {
    JNode.start(args);
  }
}
```

and run from command line:

```Java
java MyJavaNodeProgram node -e "console.log('asfd')"
```

### Javascript call to static Java function

Test Javascript code:

```Javascript
var JNodeCB = require('./build/Release/JNodeCB.node');

var staticTestMethodObject = JNodeCB.getStaticMethodObject('JNodeTestCB', 'testMethod');

var testCallResult = staticTestMethodObject.call(3, 4);
console.log('got static test call result: ' + testCallResult);
```

NOTE: New-style Javascript code needs extra arguments in the function call, will go away.

Test Java class:

```Java
public class JNodeTestCB {
  public static void testMethod(long fciHandle) {
    System.out.println("Java testMethod() called");

    int argCount = JNodeNative.fciArgCount(fciHandle);
    System.out.println("arg count: " + argCount);
    if (argCount < 2) {
      System.err.println("ERROR: not enough arguments");
      return;
    }
    if (!JNodeNative.fciArgIsNumber(fciHandle, 0) ||
        !JNodeNative.fciArgIsNumber(fciHandle, 1)) {
      System.err.println("ERROR: incorrect arguments, number arguments expected");
      return;
    }

    double a = JNodeNative.fciArgNumberValue(fciHandle, 0);
    System.out.println("number argument a: " + a);
    double b = JNodeNative.fciArgNumberValue(fciHandle, 1);
    System.out.println("number argument b: " + b);

    // return the sum:
    JNodeNative.fciSetReturnNumberValue(fciHandle, a + b);
  }
}
```

To run from command line:

```shell
java JNodeTest node jnodecbTest.js
```

### Callback from Java to Javascript

Javascript:

```Javascript
var JNodeCB = require('./build/Release/JNodeCB.node');

var staticTestMethodObjectWithCallback = JNodeCB.getStaticMethodObject('JNodeTestCB', 'testMethodWithCallback');

staticTestMethodObjectWithCallback.call(function(a, b) {
  console.log('Got callback from Java');
  console.log('a: ' + a);
  console.log('b: ' + b);
  return a+b;
});
```

Java:

```Java
public class JNodeTestCB {
  public static void testMethodWithCallback(long fciHandle) {
    System.out.println("Java testMethodWithCallback() called");

    int argCount = JNodeNative.fciArgCount(fciHandle);
    System.out.println("arg count: " + argCount);
    if (argCount < 1) {
      System.err.println("ERROR: not enough arguments");
      return;
    }

    if (!JNodeNative.fciArgIsFunction(fciHandle, 0)) {
      System.err.println("ERROR: incorrect argument, function argument expected");
      return;
    }

    long fph = JNodeNative.fciArgFunctionAsPersistentHandle(fciHandle, 0);
    long fco = JNodeNative.fcoFromHandle(fph);
    JNodeNative.fcoAddIntegerParameter(fco, 123);
    JNodeNative.fcoAddIntegerParameter(fco, 456);
    int r = JNodeNative.fcoIntCallAndDestroy(fco);
    JNodeNative.functionPersistentHandleDestroy(fph);
    System.out.println("Got result back from JS callback: " + r);
  }
}
```

## Build

```shell
make
```

NOTE: This will fetch and extract Node `4.1.2` source code if necesssary.

## Testing

### Simple Java-driven startup test

Run simple Java-driven startup test:

```shell
java JNodeTest -e "console.log('3 + 4 = ' + (3+4))"
```

### Two-way Javascript/Java callback test

Javascript in `jnodecbTest.js`:

```Javascript
var JNodeCB = require('./build/Release/JNodeCB.node');

var staticTestMethodObject = JNodeCB.getStaticMethodObject('JNodeTestCB', 'testMethod');

var testCallResult = staticTestMethodObject.call(3, 4);
console.log('got static test call result: ' + testCallResult);

var staticTestMethodObjectWithCallback = JNodeCB.getStaticMethodObject('JNodeTestCB', 'testMethodWithCallback');

staticTestMethodObjectWithCallback.call(function(a, b) {
  console.log('Got callback from Java');
  console.log('a: ' + a);
  console.log('b: ' + b);
  return a+b;
});

console.log('END OF TEST');
```

### Build and run Javascript/Java callback tests

Build Java callback test:

```shell
make all
```

To run Java callback test, with automatic result checking:

```shell
make test
```
or
```shell
java JNodeTest jnodecbTest.js
```

## To regenerate JNI header

First rebuild JNodeNative.class:

```shell
javac JNodeNative.java
```

Then regenerate JNodeNative.h:

```shell
javah JNodeNative
```

`JNodeNative.h` should now be regenerated.
