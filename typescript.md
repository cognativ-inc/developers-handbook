# Typescript guide

## Describing data

TypeScript is a powerful tool for describing the shape of objects and functions in your code, providing type safety and improving the maintainability of your codebase. However, using the `any` type is equivalent to opting out of type checking for a variable, which can undermine the benefits of using TypeScript in the first place. As such, we strongly recommend that you avoid using `any` in your code whenever possible.

To illustrate, consider the following example:

```typescript
function addNumbers(a: any, b: any) {
  return a + b;
}
```

In this example, the `addNumbers` function takes two parameters, both of which have the `any` type. This means that the function will accept any type of input for these parameters, and will not perform any type checking on them. This can lead to unexpected behavior or errors at runtime, and can make it difficult to reason about the behavior of the function.

To avoid these issues, it's generally better to use more specific types for your function parameters whenever possible. For example:

```typescript
function addNumbers(a: number, b: number) {
  return a + b;
}
```

In this updated example, we've replaced the `any` type with the `number` type for the `a` and `b` parameters, which makes it clear that the function expects numeric inputs. This provides stronger type safety and can help catch errors earlier in the development process.

Overall, avoiding the use of any in your code can help improve the reliability and maintainability of your TypeScript projects.

Let’s check this other example:

```typescript
type Result = "success" | "failure"
function verifyResult(result: Result) {
    if (result === "success") {
        console.log("Passed");
    } else {
        console.log("Failed")
    }
}
```

In this example, we define a `Result` type that is a union of two string literal types: `"success"` and `"failure"`. We then define a function `verifyResult` that takes a single parameter of type `Result`, which means it can only be one of the two string literals defined in the `Result` type.

By using a type like `Result`, we can ensure that the function `verifyResult` only accepts valid input, which in this case is either `"success"` or `"failure"`. This improves the type safety and reliability of our code, making it easier to catch errors at compile time rather than at runtime.

## Use enums

TypeScript's `enum` feature is a powerful tool for defining a set of named constants and defining standards that can be reused throughout your codebase. By using `enum`, you can create a well-defined set of options that are easy to read, write, and maintain. We recommend that you export your enums at the global level, and then let other classes import and use them.

For example, suppose you want to create a set of possible actions to capture the events in your codebase. TypeScript provides both numeric and string-based enums to help accomplish this. Here's an example of how to use a string-based enum:

```typescript
export enum ActionType {
  BUILD = 'BUILD',
  MODIFY = 'MODIFY',
  ERASE = 'ERASE',
}
```

In this updated example, we've defined an `ActionType` enum with three string values: `"BUILD"`, `"MODIFY"`, and `"ERASE"`. We've also exported the enum at the global level using the export keyword, which allows other classes and modules to import and use the ActionType enum.

To use this `ActionType` enum in another file, you can import it like this:

```typescript
import { ActionType } from './enums';

function handleAction(action: ActionType) {
  // ...
}
```

In this example, we've imported the `ActionType` enum from a separate file called `enums.ts`. We can now use the `ActionType` enum as a type for our `action` parameter, which means it can only be one of the three string values defined in the `ActionType` enum.

By using enums in your TypeScript code, you can improve its readability, maintainability, and type safety. It's a best practice to export enums at the global level and import them as needed, as this makes it easier to reuse them throughout your codebase.

Another full example would be:

```typescript
enum EventType {
    CREATE,
    DELETE,
    UPDATE
}

class InfraEvent {
    constructor(event: EventType) {
        if (event === EventType.CREATE) {
            // Call for other function
            console.log(`Event Captured :${event}`);
        }
    }
}

let eventSource: EventType = EventType.CREATE;
const eventExample = new InfraEvent(eventSource)
```

### Dos

* Use enums to define a set of named constants and define standards that can be reused in your code base.
* Use descriptive and meaningful names for your enum values to make your code more readable and self-explanatory.
* Use string-based enums instead of numeric enums when the values have semantic meaning, to avoid confusion and ensure type safety.
* Use `const` enums when you don't need the values to be looked up at runtime, as they can improve performance and generate smaller code.
* Use enums in combination with interfaces or type aliases to define more complex types that capture the behavior and properties of your code.

### Don'ts

* Don't use enums for values that are likely to change, as this can make your code brittle and hard to maintain.
* Don't use numeric enums when the values don't have semantic meaning, as this can lead to confusion and errors when values are used incorrectly.
* Don't rely too heavily on enums for type safety. While enums can help enforce type safety, they are not a silver bullet and should be used in conjunction with other TypeScript features like type aliases and interfaces.
* Don't define enums that are too large or complex. Enums that are too large or have too many properties can be difficult to work with and maintain, and can slow down the performance of your code.

## Use Interfaces

Using interfaces is a best practice in TypeScript for creating contracts that define the structure of objects and functions in your code. An interface defines a set of properties and methods that a class or object must adhere to in order to comply with the contract.

For example, you can use an interface to standardize the properties of a class and ensure that callers provide the expected parameters when using the class. Here's an example of how to use an interface:

```typescript
interface User {
  id: number;
  name: string;
  email: string;
  isAdmin?: boolean;
}

class UserDatabase {
  constructor(private users: User[]) {}

  addUser(user: User) {
    this.users.push(user);
  }

  getUserById(id: number) {
    return this.users.find(user => user.id === id);
  }

  getAllUsers() {
    return this.users;
  }
}
```

Here are some dos and don'ts when working with interfaces in TypeScript:

### Dos

* Use interfaces to define the structure of objects or functions in your code.
* Give interfaces descriptive and meaningful names that accurately describe what they represent.
* Use optional properties and index signatures sparingly, as they can make your code harder to read and maintain.
* Use `extends` and intersection types to compose interfaces together and create more complex types.
* Use interfaces to define contracts that must be adhered to by classes or objects, improving the reliability and maintainability of your code.

### Don'ts

* Don't use interfaces for primitive types like `string`, `number`, or `boolean`, as these types are already well-defined in TypeScript.
* Don't use interfaces solely for documentation purposes. While interfaces can serve as documentation, they should primarily be used to define the structure of objects or functions in your code.
* Don't make interfaces too complex or verbose, as this can make them harder to read and maintain.
* Don't rely too heavily on interfaces for type safety. While interfaces can help enforce type safety, they are not a silver bullet and should be used in conjunction with other TypeScript features like type aliases and enums.

Overall, interfaces are a powerful tool in TypeScript for defining contracts and improving the reliability and maintainability of your code. By following these dos and don'ts, you can use interfaces effectively and avoid common pitfalls when working with them.

### Other bad practices on interfaces

Here are some other bad practices to avoid when working with interfaces in TypeScript:

* Avoid using `?` on every property of an interface. While optional properties can be useful in certain cases, overusing them can make your code harder to read and maintain, as it makes it more difficult to understand what properties are required and what aren't. Instead, only use optional properties when it makes sense to do so.

* Avoid using the `!` non-null assertion operator in interfaces. While non-null assertions can be useful in certain cases, overusing them can make your code less safe, as it can result in runtime errors if a null or undefined value is encountered. Instead, use optional properties and null checks to handle situations where a value may be undefined or null.

* Avoid defining interfaces with too many properties. Interfaces that are too large can be difficult to work with and maintain, as it can be hard to keep track of all the properties and their interactions. Instead, try to break down larger interfaces into smaller, more focused ones that are easier to work with.

* Avoid defining interfaces that are too restrictive. Interfaces should define the minimum requirements for a given type or object, without being too prescriptive about how that type or object is used. If an interface is too restrictive, it can limit the flexibility and extensibility of your code, making it harder to modify and maintain.

* Avoid using overly complex interfaces that are difficult to understand. Interfaces should be easy to read and understand, so that other developers can quickly grasp what they represent and how they are used. If an interface is too complex or hard to understand, it can lead to confusion and errors in your code.

### Use readonly when needed

Some properties can only be modified when an object is first created. You can specify this by putting readonly before the name of the property, as the following example shows.

```typescript
interface Position {
    readonly latitude: number;
    readonly longitute: number;
}
```

### Extend interfaces

Extending interfaces reduces duplication, because you don't have to copy the properties between interfaces. Also, the reader of your code can easily understand the relationships in your application.

```typescript
 interface BaseInterface{
    name: string;
  }
interface EncryptedVolume extends BaseInterface{
    keyName: string;
  }
interface UnencryptedVolume extends BaseInterface {
    tags: string[];
  }
```


## Use factories

In an Abstract Factory pattern, an interface is responsible for creating a factory of related objects without explicitly specifying their classes. For example, you can create a Lambda factory for creating Lambda functions. Instead of creating a new Lambda function within your construct, you’re delegating the creation process to the factory. For more information on this design pattern, see [Abstract Factory in TypeScript](https://refactoring.guru/design-patterns/abstract-factory/typescript/example)

## Use destructuring on properties

Destructuring is a powerful JavaScript feature introduced in ECMAScript 6 (ES6) that allows you to extract values from arrays or objects and assign them to individual variables in a more concise and readable way.

With destructuring, you can extract multiple pieces of data from complex objects or arrays, including nested objects and arrays. This can make your code more modular and easier to understand, as it allows you to work with individual values directly instead of accessing them through the object or array.

```typescript
const point = [10, 20];
const [x, y] = point;

console.log(x); // Output: 10
console.log(y); // Output: 20
```

On objects:

```typescript
const object = {
    objname: "obj",
    scope: "this",
};

const oName = object.objname;
const oScop = object.scope;

const { objname, scope } = object;
```

## Don’t use the var keyword

The `let` statement is used to declare a local variable in TypeScript. It’s similar to the `var` keyword, but it has some restrictions in scoping compared to the `var` keyword. A variable declared in a block with `let` is only available for use within that block. The `var` keyword has global scope, which means that it’s available and can be accessed only within that function. You can re-declare and update `var` variables. It’s a best practice to avoid using the `var` keyword.

## Use === instead of ==

There are two operators to detect the equality: `==` and `===`. `===` does a type conversion before checking the equality. It is considered best practice to always use the former set when comparing or checking equality between two operators.

This will help you not to coerce the values of the operator. Let us have a look at an example.

```typescript
"Mengueche" == new String("Mengueche") // true
```

In the above example, you can see the `==` operator gives out the result as accurate, which should not be the case as they do not have the same type. `==` will only give you the comparison between the values & not based on the data types.

Instead, you should use `===` in avoid the issue.

## Avoid the use of any types

You can eliminate the use of `any` type in your code. `Any` data type in TypeScript is used when we don’t know the exact kind of variable that should be used in the case. Although `any` type was never a bad thing, just that the user should know the type they are working on.

This improves code simplicity and complexity of the code. Instead, one should use `unknown` when you do not know the type. It is similar to `any` but has one difference: it can only interact after seeing the type (through something like a type guard or inference).

Here is an example:

```typescript
const foo: any = "foo";
const bar: unknown = "bar";

foo.length; // Works, type checking is effectively turned off for this
bar.length; // Errors, bar is unknown

if (typeof bar === "string") {
  bar.length; // Works, we now know that bar is a string
}
```

## Use access modifiers for classes

Similar to Java, TypeScript comes with access modifiers for classes. These access modifiers have different properties. We have `public`, `protected`, or `private` access modifiers.

* **private:** only accessible inside the class
* **protected:** only accessible inside the class and through subclasses
* **public:** accessible anywhere

This helps preserve the class members' security; it includes properties and attributes and prevents unauthorized access and use. This will help classes not to pop up abruptly anywhere within the script.

Example:


```typescript
class Student{
  protected name: string;
  private marks: number;

  constructor(name: string, marks: number) {
    this.name = name;
    this.marks = marks
  }

  public getMarks(){
    return salary
  }
```

Here, you cannot access `Marks` unless you use the `getMarks` method.

```typescript
class Child extends Student {
  viewDetails() {
    console.log(this.marks); // error: property 'marks’' is private
    console.log(this.getMarks()); // success
  }
}
```

But you can access the name using a sub-class.

```typescript
class class Child  extends Student{
  viewDetails(){
    console.log(this.name);
  }
}
```

## Enable `strict` check on

The use `strict` directive was actually introduced in ECMAScript 5 (ES5) as a way to enable strict mode in JavaScript. When you use `use strict` at the beginning of a script or a function, you're telling the browser or engine to enforce a stricter set of rules when interpreting the code. This can help catch common programming mistakes and improve the overall quality of your code.

In TypeScript, you can enable `strict` mode using the strict configuration option in the `tsconfig.json` file. This option enables several strict checks that can help prevent common errors and improve the overall quality of your code. For example, enabling `strictNullChecks` can help catch `null` or `undefined` errors at compile-time, while enabling `strictFunctionTypes` can help ensure that function types are compatible with each other.

## Use tuples for fixed length arrays

```typescript
let marks: number[] = [1, 2, 3];
```

You can use the above `marks` array to store different number of items in different places of the same script. TS is not gonna restrict you as long as you provide all the values with the correct defined data type.

```typescript
let marks: number[] = [1, 2, 3];
marks = []; // success
marks = [1]; // success
marks = [1, 2, 3, 4, 5]; // success
```

However this can lead to nasty logical errors in cases where the array length is a constant. To avoid these you should use `array` as a tuple, whenever the size should be a fixed size. Tuple is a properly defined array with each of the expecting value’s data type.

```typescript
let marks:[number, number] = [1, 2]; // tuple of 2 number values
marks = [10, 20]; // success
marks = [1]; // syntax error
marks = [1, 2, 3, 4, 5] // syntax error
```

## Use type aliases in repetitive data types

Assume that you have multiple variables/ objects in your script which all follow the same structure of data types.

```typescript
let man: {name: string, age: number} = {name = "john", age=30};
let woman: {name: string, age: number} = {name = "Anne", age=32};
```

To avoid this redundant chunks of `type` declarations and to re-use types, you can use type aliases.

```typescript
type Details = {name: string, age: number}; // defining type alias
let man: Details = {name = "john", age=30}; // using type alias
let woman: Details = {name = "Anne", age=32};
```

The additional benefit of using a `type` alias is, your intention of defining the data is now clearly visible.

## Using the `Object` Type

The `Object` type is a built-in feature of TypeScript that allows you to refer to the base object type. It can be used to improve the type safety of your code by ensuring that all objects have certain properties or methods.

For example, you can use the `Object` type to create a more type-safe function that takes an object as an argument:

```typescript
function printObject(obj: Object) {
 console.log(obj);
}
```

You can also use the `Object` type to create more type-safe variables:

```typescript
let obj: Object = { name: "John", age: 30 };
let str: string = obj.name; // valid
let num: number = obj.age; // valid
```

By using the `Object` type, you can ensure that all objects have certain properties or methods, which can improve the type safety of your code.

## Use “never”

In TypeScript, `never` is a special type that represents values that will never occur. It’s used to indicate that a function will not return normally, but will instead throw an error. This is a great way to indicate to other developers (and the compiler) that a function can’t be used in certain ways, this can help to catch potential bugs.

For example, consider the following function that throws an error if the input is less than 0:

```typescript
function divide(numerator: number, denominator: number): number {
 if (denominator === 0) {
 throw new Error("Cannot divide by zero");
 }
 return numerator / denominator;
}
```

Here, the function divide is declared to return a number, but if the denominator is zero, it will throw an error. To indicate that this function will not return normally in this case, you can use `never` as the return type:

```typescript
function divide(numerator: number, denominator: number): number | never {
 if (denominator === 0) {
 throw new Error("Cannot divide by zero");
 }
 return numerator / denominator;
}
```