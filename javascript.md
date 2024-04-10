<!-- @format -->

# Javascript

Learn about the best practices for JavaScript development at Cognativ to avoid common issues during code review. By following these guidelines, you can save time and prevent headaches. Many of these principles are based on the principles of clean code.

This guide will be written in full Javascript examples, for Typescript read the proper chapter.

Unless client requirement, always work in ES6.

## Variables

### Learn the difference between var, let and const and when to use them

In JavaScript, `var`, `let`, and `const` are all used to declare variables, but they have some key differences in terms of scope and reassignment.

`var` is the oldest way to declare a variable in JavaScript. Variables declared with `var` are function scoped, meaning they are only accessible within the function they were declared in. If a variable is declared with var outside of any function, it becomes a global variable and can be accessed from anywhere in the code. They are also hoisted, which means they are accessible before they are declared.

```Javascript
var count = 5; 
console.log(count); // Output: 5
count = 10;
console.log(count); // Output: 10
```

`let` is a newer way to declare variables, introduced in ECMAScript 6. Like `var`, variables declared with `let` are also block scoped, meaning they are only accessible within the block they were declared in. However, unlike `var`, variables declared with `let` are not hoisted, so they cannot be accessed before they are declared.

```Javascript
let y = 5; 
console.log(y); // Output: 5
y = 10;
console.log(y); // Output: 10
```

`const` is also a newer way to declare variables, also introduced in ECMAScript 6. Variables declared with `const` are also block scoped, like `let`, **but they are read-only**, meaning *they cannot be reassigned a new value* once they are declared.

```Javascript
const count = 5; 
console.log(count); // Output: 5
count = 10; // This will throw a TypeError: Assignment to constant variable.
```

Learning the difference is key. Sometimes developers used `var`, `let` or `const` in places when they weren’t meant for. A Typescript example we saw doing code review:

```Javascript
export function validateExpiryDate(valueDate: Date): ValidationErrors{
  let today = new Date();
  let MaxExpiryDate = new Date();
  MaxExpiryDate.setFullYear(MaxExpiryDate.getFullYear() + 10);

  if(valueDate <= today || valueDate >= MaxExpiryDate) {
    return { invalidExpiryDate: true};
  }
  return {};
} 
```

In this case, `today` and `MaxExpiryDate` are declared as a `let` variables, which means their contents can be reassigned in the future. In this function, as you see in the example, the use of let isn’t necessary. By declaring `const` is better and more secure to prone errors by accidentally trying to reassign the variables.

```Javascript
export function validateExpiryDate(valueDate: Date): ValidationErrors{
  const today = new Date();
  const MaxExpiryDate = new Date();
  MaxExpiryDate.setFullYear(MaxExpiryDate.getFullYear() + 10);

  if(valueDate <= today || valueDate >= MaxExpiryDate) {
    return { invalidExpiryDate: true };
  }
  return {};
}
```

### Naming variables properly


#### JavaScript Variable Naming Conventions

There are a few conventions for naming variables in JavaScript:

1. **Camel case**: This is the most common convention for naming variables in JavaScript. In camel case, the first letter of the variable name is in lowercase, and subsequent words in the variable name have their first letter capitalized. For example, `firstName` or `customerAddress`.

2. **Pascal case**: Similar to camel case, but the first letter of the first word is also capitalized. This is commonly used for constructors and classes in JavaScript. For example, `FirstName` or `CustomerAddress`.

3. **Snake case**: In snake case, words in the variable name are separated by an underscore. For example, `first_name` or `customer_address`.

4. **Kebab case**: In kebab case, words in the variable name are separated by a hyphen. For example, `first-name` or `customer-address`.

5. **Use meaningful names**: The variable name should be descriptive and indicate the purpose of the variable. Avoid short or generic names like `x`, `temp`, `data`, etc.

6. **Use consistent naming**: Be consistent in your naming conventions throughout your codebase. This makes the code easier to read and understand.

7. **Use the same convention for variables, functions and class names**: Using the same naming convention for variables, functions, and class names makes the codebase more consistent and easier to read.

It's worth mentioning that in javascript there are some reserved words that cannot be used as variables or function names, you can check them in the [ECMAScript documentation](https://www.ecma-international.org/ecma-262/11.0/index.html#sec-reserved-words)

It's also important to keep in mind that these conventions are just that, conventions, and not strict rules. The most important thing is to be consistent with your naming throughout your codebase and to make it easy to read and understand.

At Cognativ we enforce camelCase for *functions*, variables and Pascal case for *Classes* and SNAKE_CASE for *magic numbers* (see below). Avoid, unless by client request, the usage of other conventions.

#### Using meaningful variable names

Avoid one, two or three letter variables, ex.:

```Javascript
let x = { name: 'Diego' }
```

Better do:

```Javascript
let postAuthor = { name: 'Diego' }
```

One way to improve this is by providing more context and examples of how proper naming conventions can help make the code more readable and understandable. For example:

```Javascript
const today = new Date()
```

While "today" may seem obvious at first glance, it can be confusing later on when browsing the code or doing searches. Is it referring to the current date? Or the day of the month? Or the day of the week in words? Many assumptions can be made. To make the code more clear and specific, use a more descriptive name for the variable. For example:

```Javascript
const currentDate = new Date()
```

This clearly indicates that the variable holds the current date. Similarly, for a specific feature, you can use a more descriptive name, like:

```Javascript
const currentPostDate = new Date()
```

We use a lot of date variables in our applications, but we rarely differentiate them. Describing the variables with proper names helps to quickly find and differentiate them later on in your code. This improves the readability and maintainability of the code, making it easier to understand and debug.

Avoid simple words:

```Javascript
const temp = true
```

```Javascript
const today = new Date()
```

```Javascript
const id = props.route.params._id;
```

Try to use:

```Javascript
const currentActiveUser = true
```

```Javascript
const currentDate = new Date()
```

```Javascript
const userId = props.route.params._id;
```

On loops, people commonly uses the old Java convention for iterating a loop using variable `i`: 

```Javascript
for (let i = 0; i < 10; i++) {
  console.log(i);
}
```


While this is not a “big problem”, you’re not forced to use variables names with one letter, you can do a good descriptive variable like: 

```Javascript
for (let counter = 0; counter < 10; counter++) {
    console.log(counter);
}
```

It's also worth noting that the loop variable doesn't have to be an integer, it can be any type of variable, and you can also change the initial value, the condition and the increment/decrement.

```Javascript
let counter = 10;
while (counter > 0) {
    console.log(counter);
    counter -= 1;
}
```

In this example, instead of using a for `loop` we use a while `loop`, and the variable `counter` starts at 10 and decreases by 1 in each iteration until it reaches 0.

The key is to make sure the variable is used consistently and it's meaningful to the purpose of the `loop`.

#### Using CAPS as variables

Using CAPS on variables is a good practice but for certain situations, for example, for making extreme searchable items and defining global variables and defining magic numbers.

We all know magic numbers are frowned upon as a programming practice. They may give no indication of their meaning, and when used multiple times, can result in future inconsistencies. They can expose you to the risk of typos, hinder maintenance and have an impact on readability. 

```Javascript
const MILLISECONDS_PER_DAY = 60 * 60 * 24 * 1000; //86400000;
```

#### Use searchable names EDIT

It's important that the code we do write is readable and searchable and camel case helps a lot. But by _not_ naming variables that end up being meaningful for understanding our program, we complicate our readers.

Make your names searchable. Tools like
[buddy.js](https://github.com/danielstjules/buddy.js) and
[ESLint](https://github.com/eslint/eslint/blob/660e0918933e6e7fede26bc675a0763a6b357c94/docs/rules/no-magic-numbers.md)
can help identify unnamed constants.

**Bad:**

```javascript
setTimeout(blastOff, 86400000);
```

**Good:**

```javascript
// Declare them as capitalized named constants.
const MILLISECONDS_PER_DAY = 60 * 60 * 24 * 1000; //86400000;

setTimeout(blastOff, MILLISECONDS_PER_DAY);
```

Usually we use this convention for magic numbers, as you have noticed, our code is plenty of these examples where lots of numbers are there and they.

We also may use it for global .env variables or by naming reducer actions.

### Avoid mental mapping

Sometimes, we may use single letter props and it makes it difficult:

```javascript
const locations = ["Austin", "New York", "San Francisco"];
locations.forEach(l => {
  doStuff();
  doSomeOtherStuff();
  // ...
  // ...
  // ...
  // Wait, what is `l` for again?
  dispatch(l);
});
```

It should be way better doing this:

```Javascript
locations.forEach(location => {
  doStuff();
  doSomeOtherStuff();
  // ...
  // ...
  // ...
  dispatch(location);
});
```


### Don't add unneeded context

Overdoing variable names if the class/object already hints you the meaning.

**Bad:**

```javascript
const Airplane = {
  airplaneMake: "Piper",
  airplaneModel: "Cherokee",
  airplaneColor: "White"
};

function paintAirplane(airplane, color) {
  airplane.airplaneColor = color;
}
```

**Good:**

```javascript
const Airplane = {
  make: "Piper",
  model: "Cherokee",
  color: "White"
};

function paintAirplane(airplane, color) {
  airplane.color = color;
}
```



## Comments

Writing comments in programming is good because it helps other developers understand the purpose and functionality of your code. Comments can also serve as documentation for future maintenance or updates to the code.

An undocumented function:

```Javascript
function validateEmail(email) {
  if (typeof email !== 'string') return false;
  if (!email.trim() || email === null || email === undefined) return false;
  if (email.length > 320) return false;
  if (email.split('@').length !== 2) return false;
  const emailparts = email.split('@');
  if (!emailparts[0].trim() || !emailparts[1].trim()) return false;
  if (emailparts[0].includes(' ') || emailparts[1].includes(' ')) return false;
  return true;
}
```

A well documented function:

```Javascript
/**
 * @description A function that receives an email as a string and returns a boolean if the email is valid or not.
 *
 * Criteria acceptance:
 *
 * Must:
 *
 * 1. Contain only one @ symbol
 * 2. Contain a domain name
 * 3. Contain a username
 * 4. Be a typefo string
 *
 * Must not:
 *
 * 1. Contain spaces at the beginning or end
 * 2. Contain spaces in the username
 * 3. Contain spaces in the domain name
 * 4. be an empty string
 * 5. be null
 * 6. be undefined
 * 7. be a number, an array, an object, a function, etc.
 * 8. longer than 320 characters
 *
 * @param {string} email the email to be validated
 * @returns {boolean} true if the email is valid, false if the email is invalid
 *
 */

function validateEmail(email) {
  // check if the input is a string
  if (typeof email !== 'string') return false;
  
  // check if the input is empty, null or undefined
  if (!email.trim() || email === null || email === undefined) return false;
  
  // check if the input is longer than 320 characters
  if (email.length > 320) return false;
  
  // check if the email contains only one @ symbol
  if (email.split('@').length !== 2) return false;
  
  // check if the email contains a domain name and a username
  const emailparts = email.split('@');
  if (!emailparts[0].trim() || !emailparts[1].trim()) return false;
  
  // check if there are spaces in the username or domain name
  if (emailparts[0].includes(' ') || emailparts[1].includes(' ')) return false;

  return true;
}
```

This code is exceptionally well explained, it adds:

* A brief explanation of what the function is all about and the output.
* A criteria acceptance for returning `true` or `false`
* `@params` required for the function
* The `@response`
* It adds context to every validation since they may be trickier

Writing comments on code also helps you to follow a guideline of steps, also, when you are using that function Visual Studio Code and other IDEs will hint you properly, it’s a win win situation.

### Some advice when writing comments

‌**Never leave commented code.** Avoid leaving commented code. 

```Javascript
function addUserInfo(name, surename) {
  ...
  //checkSurenames();
  // if (!surename) return false
  return username
}
```

Your code should be clean of these type of comments. The code to be merged cannot contain unfinished commented code. It makes it hard to maintain and people accidentally can uncomment lines and break the main code.

**Use clear and concise language.** Avoid using jargon or technical terms that may not be familiar to all readers.

A bad comment:

```Javascript
// iterate thru arr and add 1 to each element
for (let i = 0; i < arr.length; i++) {
  arr[i] += 1;
}
```

This comment doesn't provide any useful information. It simply restates what the code is doing.

A good comment for this code would provide more context or explain why the operation is being performed:

```Javascript
// increase the value of each element in the array by 1
// This is used to increase the tax rate by 1%
for (let i = 0; i < arr.length; i++) {
  arr[i] += 1;
}
```

**Write comments at the appropriate level of detail.** Avoid providing too much information in a comment, as it can make the code harder to read.

A wall of text comment:

```Javascript
/**
 * The following function takes in an array of numbers and a target number. 
 * It then initializes two pointers, left and right, both set to the first 
 * element of the array. It then enters a while loop that continues until 
 * left is greater than or equal to right. Within the while loop, it 
 * calculates the sum of the elements at the left and right pointers and 
 * compares it to the target number. If the sum is less than the target, it 
 * increments the left pointer. If the sum is greater than the target, it 
 * decrements the right pointer. If the sum is equal to the target, it returns 
 * true. If the loop completes and the function has not yet returned, it 
 * returns false. */

function twoPointerSum(arr, target) {
  let left = 0;
  let right = arr.length - 1;
  while (left < right) {
    let sum = arr[left] + arr[right];
    if (sum < target) {
      left++;
    } else if (sum > target) {
      right--;
    } else {
      return true;
    }
  }
  return false;
}
```

It would be much better this way:

```Javascript
/**
* The function uses two pointer technique to check if there are any two numbers in an array that add up to a target
* @param {Array} arr - an array of numbers
* @param {Number} target - target number
* @returns {Boolean} - return true if there are any two numbers in an array that add up to a target, otherwise return false
*/
function twoPointerSum(arr, target) {
  let left = 0;
  let right = arr.length - 1;
  while (left < right) {
    let sum = arr[left] + arr[right];
    if (sum < target) {
      left++;
    } else if (sum > target) {
      right--;
    } else {
      return true;
    }
  }
  return false;
}
```

Providing criteria acceptance isn’t a bad idea, but only when it really matters, ex. When making functions that has many validations (ex, the email validator function above).

**Use comments to describe the purpose of a section of code rather than its implementation.** The code should be self-explanatory, but comments can provide context and explain the reasoning behind certain decisions.

```Javascript
// create new object, set properties x and y to input values, and return object
const createObject = (x, y) => {
  const obj = {};
  obj.x = x;
  obj.y = y;
  return obj;
}

```

This comment does not add any value to the code, it simply restates what the code is doing.

A good comment for this code would describe the purpose of the function and its inputs and outputs:

```Javascript
/**
* createObject - a function that creates an object with properties x and y set to input values
* @param {Number} x - x value
* @param {Number} y - y value
* @returns {Object} - return an object with properties x and y set to input values
*/
const createObject = (x, y) => {
  const obj = {};
  obj.x = x;
  obj.y = y;
  return obj;
}
```

This comment provides a clear overview of what the function does and what are its inputs and outputs. It makes it easy for the reader to understand the purpose and functionality of the code.



**Keep comments up to date.** As code changes, make sure to update any related comments to reflect the new functionality.

Let’s mess developers:

```Javascript
// get the first item from the array
const firstItem = arr[0];

// remove the first item from the array
arr.splice(0, 1);
```

The first comment says that the code is getting the first item from the array, however, the second line of code actually removes the first item from the array, so the comment is not accurate anymore.

A good comment for this code would be updated to reflect the current functionality of the code:

```Javascript
// get and remove the first item from the array
const firstItem = arr.shift();
```

This comment accurately describes what the code is doing, which is getting and removing the first item from the array using the `shift()` method.

It's important to keep comments up-to-date with the code, so that readers can rely on the comments to understand the functionality of the code. It's a good practice to review and update comments regularly as the codebase evolves.

**Don’t have journal comments** If you need journaling, do this with the right tool: a version control. Just commit your changes declaring what are the changes, they’re better in every sense. There’s no need for dead code, commented code, and especially journal comments. Use git log to get history!

Bad:

```Javascript
/**
 * 2016-12-20: Removed monads, didn't understand them (RM)
 * 2016-10-01: Improved using special monads (JP)
 * 2016-02-03: Removed type-checking (LI)
 * 2015-03-14: Added combine with type-checking (JR)
 */
function combine(a, b) {
  return a + b;
}
```

**Use a consistent format for comments.** This makes it easy for others to find and understand them.

Here's an example of a bad comment that uses inconsistent formatting:

```Javascript
//calculate the average of an array of numbers
//input: array of nums
//output: average num
function calculateAverage(nums) {
  let total = 0;
  for (let num of nums) {
    total += num;
  }
  return total/nums.length;
}
```

In this example, the comments use different capitalization, punctuation, and formatting, making it harder to read and understand.

A good comment for this code would use consistent formatting:

```Javascript
/**
 * calculateAverage - a function that calculates the average of an array of numbers
 * @param {Array} nums - an array of numbers
 * @returns {Number} - the average of the array of numbers
 */
function calculateAverage(nums) {
  let total = 0;
  for (let num of nums) {
    total += num;
  }
  return total/nums.length;
}
```

This comment uses the same formatting, capitalization and punctuation throughout, making it easy to read and understand. Using a consistent format for comments helps make the codebase more organized and easier to navigate, especially for developers who are new to the project.

**Use inline code when it matters.** Inline comments can be useful in some cases, but it depends on the specific context and the level of complexity of the code.

Inline comments are comments that are placed directly next to the code they are describing, often on the same line. They can be useful for providing brief explanations or clarifications for specific lines of code that may not be obvious from the code itself.

Use when:

* The following block is not obvious as itself.
* When the function has validations, conditionals

Writing inline codes also helps to memorize the criteria acceptance of the task. If you have a function that has to do a, b, c it helps to keep track of everything:

```Javascript
function doSomething() {
  // Must do a
  ...
  // Must do a
  ... 
  // Must do a
  ...

  return true;
}
```

**Avoid positional markers**. They usually just add noise. Let the functions and variable names along with the proper indentation and formatting give the visual structure to your code.

Bad code:

```Javascript
// ********************************************************
// Function to fetch an user
// ********************************************************

export const fetchUserProfile = userId => {
  return async dispatch => {
    // ********************************************************
    // Function to fetch the loged in user
    // ********************************************************
    const fetchData = async () => {
      // we call the API
      const response = await fetch(`${API_URL}/v1/users/${userId}`);
      // we check if there's an error
      if (!response.ok) {
        throw new Error('could not fetch any data');
      }
      // if OK then we get the response
      const data = await response.json();
      // we return data
      return data;
    };

    // Once we have the data, we will dispatch it
    try {
      const userData = await fetchData();
      dispatch(userActions.setCurrenUserProfile({ user: userData.user }));
    } catch (error) {
      console.log(error);
    }
  };
};
```

Much better:

```Javascript
/**
 * this function will fetch the user profile and return an object with the user data
 * 
 * @param {*} userId 
 * @returns {object} { user: { id, name, email, avatar, createdAt, updatedAt }
 */
 
export const fetchUserProfile = userId => {
  return async dispatch => {
    // we create a function to fetch the data
    const fetchData = async () => {
      // we call the API
      const response = await fetch(`${API_URL}/v1/users/${userId}`);
      // we check if there's an error
      if (!response.ok) {
        throw new Error('could not fetch any data');
      }
      // if OK then we get the response
      const data = await response.json();
      return data;
    };

    // Once we have the data, we will dispatch it
    try {
      const userData = await fetchData();
      // we dispatch the action to set the user profile
      dispatch(userActions.setCurrenUserProfile({ user: userData.user }));
    } catch (error) {
      console.log(error);
    }
  };
};
```

## Functions

Functions are core part of any language. Learn how to write amazing and well coded functions using this guide. Good functions will not only end up with good performant code, but also, it will make your life easier when it comes to test.

### Functions should do one thing

This is by far the most important rule in good software engineering. When functions do more than one thing, they are harder to compose, test, and reason about. 

**Bad:**

```javascript
function emailClients(clients) {
  clients.forEach(client => {
    const clientRecord = database.lookup(client);
    if (clientRecord.isActive()) {
      email(client);
    }
  });
}
```

**Good:**

```javascript
function emailActiveClients(clients) {
  clients.filter(isActiveClient).forEach(email);
}

function isActiveClient(client) {
  const clientRecord = database.lookup(client);
  return clientRecord.isActive();
}
```

When you can isolate a function to just one action, it can be refactored easily and your code will read much cleaner. If you take nothing else away from this guide other than this, you’ll be ahead of many developers.

### Keep the number of arguments below 2

Avoid writing functions with more than three arguments if possible. It is a pain in the head for testing, it increments the combination of cases when you try to test separate arguments.

If your function has more than one argument, it means it is trying to do too much, in cases where is not, a higher object is better than nothing.

To make it obvious what properties the function expects, you can use the ES2015/ES6 destructuring syntax. This has a few advantages:

When someone looks at the function signature, it’s immediately clear what properties are being used.

1. It can be used to simulate named parameters.
2. Destructuring also clones the specified primitive values of the argument object passed into the function. This can help prevent side effects. Note: objects and arrays that are destructured from the argument object are NOT cloned.
3. Linters can warn you about unused properties, which would be impossible without destructuring.

Bad:

```Javascript
function createPost(title, body, author, status) {
  // ...
}

createPost("Foo", "Bar", "Diego", true);
```

Good:

```Javascript
function createMenu({ title, body, author, status }) {
  // ...
}

createMenu({
  title: "A really nice guide for Javascript",
  body: "Bla bla bla bla",
  author: "Diego",
  status: true
});
```


## Functions must do what they are named for

Naming functions properly is important for code readability and maintainability. Clear, descriptive function names make it easier to understand the purpose and behavior of the code, which in turn helps other developers quickly understand and work with the code. Additionally, well-named functions can also reduce the need for comments, as the function name itself should provide enough information about what the function does. This can make the code more self-documenting, making it easier to understand and maintain in the long run.

Sometimes, code can be misleading due to function names that do not accurately reflect their intended purpose. For example:

```Javascript
function addToDate(date, month) {
  // ...
}

const date = new Date();

// It's hard to tell from the function name what is added
addToDate(date, 1);
```

Good:

```Javascript
function addMonthToDate(month, date) {
  // ...
}

const date = new Date();
addMonthToDate(1, date);
```

### Avoid duplicate code

Strive to eliminate duplicate code as much as possible. Duplicate code is detrimental to the maintainability of your codebase, as it creates multiple points of failure and increases the risk of introducing bugs when making changes or updates to the logic.

Oftentimes you have duplicate code because you have two or more slightly different things, that share a lot in common, but their differences force you to have two or more separate functions that do much of the same things. Removing duplicate code means creating an abstraction that can handle this set of different things with just one function/module/class.

Getting the abstraction right is critical, that's why you should follow the SOLID principles laid out in the Classes section. Bad abstractions can be worse than duplicate code, so be careful! Having said this, if you can make a good abstraction, do it! Don't repeat yourself, otherwise you'll find yourself updating multiple places anytime you want to change one thing.

```Javascript
function showDeveloperList(workers) {
  workers.forEach(worker => {
    const expectedSalary = worker.calculateExpectedSalary();
    const experience = worker.getExperience();
    const githubLink = worker.getGithubLink();
    const data = {
      expectedSalary,
      experience,
      githubLink
    };

    render(data);
  });
}

function showManagerList(managers) {
  managers.forEach(manager => {
    const expectedSalary = manager.calculateExpectedSalary();
    const experience = manager.getExperience();
    const portfolio = manager.getMBAProjects();
    const data = {
      expectedSalary,
      experience,
      portfolio
    };

    render(data);
  });
}
```

A well done:

```Javascript
function showWorkersList(employees) {
  employees.forEach(employee => {
    const expectedSalary = employee.calculateExpectedSalary();
    const experience = employee.getExperience();

    const data = {
      expectedSalary,
      experience
    };

    switch (employee.type) {
      case "manager":
        data.portfolio = employee.getMBAProjects();
        break;
      case "developer":
        data.githubLink = employee.getGithubLink();
        break;
    }

    render(data);
  });
}
```


### Use Object.assign() to set defaults objects

Using `Object.assign()` to set default objects is a way to ensure that the properties of an object are properly initialized and that any missing properties are set to a default value. This can help prevent errors caused by undefined properties and can make the code more readable and maintainable.

`Object.assign()` method copies the values of all enumerable own properties from one or more source objects to a target object. It will return the target object.

Bad:

```Javascript
const menuConfig = {
  title: null,
  body: "Bar",
  buttonText: null,
  cancellable: true
};

function createMenu(config) {
  config.title = config.title || "Foo";
  config.body = config.body || "Bar";
  config.buttonText = config.buttonText || "Baz";
  config.cancellable =
    config.cancellable !== undefined ? config.cancellable : true;
}

createMenu(menuConfig);
```

Good:

```Javascript
const menuConfig = {
  title: "Order",
  // User did not include 'body' key
  buttonText: "Send",
  cancellable: true
};

function createMenu(config) {
  let finalConfig = Object.assign(
    {
      title: "Foo",
      body: "Bar",
      buttonText: "Baz",
      cancellable: true
    },
    config
  );
  return finalConfig
  // config now equals: {title: "Order", body: "Bar", buttonText: "Send", cancellable: true}
  // ...
}

createMenu(menuConfig);
```

See another example:

```Javascript
const options = {};
Object.assign(options, {
  option1: true,
  option2: false,
  option3: 'hello'
});
```

This will ensure that the options object has properties option1, option2, and option3, and that they are set to true, false, and 'hello', respectively. If any of these properties were not defined in the options object, they would be added and set to the specified default values.

This approach is useful, as it guarantees that the object has all the properties that we expect it to have, even if it was not passed to the function or was passed with some properties missing.

It is also possible to use spread operator instead of Object.assign()

```Javascript
const options = {};
options = {...options, option1: true, option2: false, option3: 'hello'}
```

Either way, this approach is a great way to ensure that objects are properly initialized and that all necessary properties are set to default values.

### Don't use flags as function parameters

Using flags as function parameters can make the code harder to read and understand, as well as harder to maintain.

Bad:

```Javascript
function createFile(name, temp) {
  if (temp) {
    fs.create(`./temp/${name}`);
  } else {
    fs.create(name);
  }
}
```

Good:

```Javascript
function createFile(name) {
  fs.create(name);
}

function createTempFile(name) {
  createFile(`./temp/${name}`);
}
```

When a function takes a flag as a parameter, it usually means that the function is doing more than one thing, and the flag is used to control which thing the function is doing. This can make the code harder to understand, as it is not immediately clear what the function is doing without looking at the flag. It can also make the code harder to maintain, as changes to the function may have unexpected consequences when the flag is set to different values.

A better approach would be to create separate functions for each distinct behavior or action, rather than using a flag to control the behavior of a single function. This way, the code is more modular and easier to understand, as each function has a clear and specific purpose.

Also, flags often make the function signature longer and harder to read, it makes the code less explicit, harder to understand and harder to reason about.

In summary, flags as function parameters are not recommended because they can make the code harder to read and understand, and harder to maintain. Instead, it is better to create separate functions for each distinct behavior or action.

### Avoid side-effects in functions

In summary, avoiding side-effects in functions is considered a best practice in software development because it helps to improve the readability, testability, and maintainability of the code. Functions that are free of side-effects are easier to understand, test, and reason about, which leads to more maintainable code.

Bad:

```Javascript
let counter = 0;
function incrementCounter() {
  counter++;
}
```

This function has a side-effect because it modifies the value of a global variable, `counter`. This makes the function's behavior hard to understand, predict and test.

Here's an example of a function without side-effects that could be considered "good" practice:

```Javascript
function incrementCounter(counter) {
  return counter + 1;
}
```

This function takes a parameter counter and returns a new value, it doesn't modify the input parameter and it doesn't have any other side-effects. This makes the function's behavior easy to understand, predict and test.

#### Favor functional programming over imperative programming

Favoring functional programming over imperative programming in JavaScript can lead to more readable, maintainable, and testable code.

Functional programming emphasizes the use of pure functions and immutable data, which can lead to a more predictable and self-contained codebase. Functions that have no side effects, and return a new output based on the input, are much easier to reason about and test.

```Javascript
const programmerOutput = [
  {
    name: "Uncle Bobby",
    linesOfCode: 500
  },
  {
    name: "Suzie Q",
    linesOfCode: 1500
  },
  {
    name: "Jimmy Gosling",
    linesOfCode: 150
  },
  {
    name: "Gracie Hopper",
    linesOfCode: 1000
  }
];

let totalOutput = 0;

for (let i = 0; i < programmerOutput.length; i++) {
  totalOutput += programmerOutput[i].linesOfCode;
}
```


Good:

```Javascript
Good:

const programmerOutput = [
  {
    name: "Uncle Bobby",
    linesOfCode: 500
  },
  {
    name: "Suzie Q",
    linesOfCode: 1500
  },
  {
    name: "Jimmy Gosling",
    linesOfCode: 150
  },
  {
    name: "Gracie Hopper",
    linesOfCode: 1000
  }
];

const totalOutput = programmerOutput.reduce(
  (totalLines, output) => totalLines + output.linesOfCode,
  0
);
```


#### Avoid negative conditionals

Avoiding negative conditionals can make the code more readable and easier to understand. Positive conditionals are more explicit and makes it easier to spot bugs, and it's more maintainable.

Negative conditionals are conditionals that use negation (i.e. "not" or "!") to check for the absence of a certain condition. For example:

```Javascript
if (!isValid) {
  // do something
}
```

This type of conditional can make the code harder to read and understand, as it can be more difficult to understand the intended behavior of the code. It requires the reader to negate the condition in their head to understand what the code is doing, which can lead to confusion and make it harder to spot bugs.

A better approach would be to use positive conditionals, where the condition is stated directly and without negation. For example:

```Javascript
if (isValid === false) {
  // do something
}
```

This makes the code more explicit and easier to read and understand. It also makes it easier to spot bugs as it is clear what the code is doing.

This approach is also really good when used with encapsulated conditionals.

#### Use encapsulated conditionals

Using encapsulated conditionals is good because it helps to improve the readability, maintainability, and testability of the code. It makes the code more explicit, easier to update, refactor and test, and it also reduces the risk of introducing bugs and makes it easy to spot edge cases and untested scenarios.

Imagine this situation:

Bad:

```Javascript
if (fsm.state === "fetching" && isEmpty(listNode)) {
  // ...
}
```

Good:

```Javascript
function shouldShowSpinner(fsm, listNode) {
  return fsm.state === "fetching" && isEmpty(listNode);
}

if (shouldShowSpinner(fsmInstance, listNodeInstance)) {
  // ...
}
```


Encapsulted conditionals are good because they can be used in conjunction with the positive conditionals.

#### Avoid type-checking (part 1)

Avoiding type-checking in JavaScript can lead to more flexible and dynamic code, as it allows for more dynamic code, and makes the code more reusable and easier to understand. However, it is not to say that type-checking should be avoided altogether, it should be used wisely and only when necessary.

It is worth noting that while JavaScript is a dynamically typed language, that doesn't mean that we should not use type-checking at all. Some type-checking can be helpful in certain situations such as catching bugs early, providing clear error messages or providing better autocompletion suggestions.

Bad:

```Javascript
function travelToBarcelona(vehicle) {
  if (vehicle instanceof Bicycle) {
    vehicle.pedal(this.currentLocation, new Location("texas"));
  } else if (vehicle instanceof Car) {
    vehicle.drive(this.currentLocation, new Location("texas"));
  }
}
```

Good:

```Javascript
function travelToTexas(vehicle) {
  vehicle.move(this.currentLocation, new Location("texas"));
}
```

If you are working with basic primitive values like strings and integers, and you can't use polymorphism but you still feel the need to type-check, you should consider using TypeScript.

The problem with manually type-checking normal JavaScript is that doing it well requires so much extra verbiage that the faux "type-safety" you get doesn't make up for the lost readability.

```Javascript
function combine(val1, val2) {
  if (
    (typeof val1 === "number" && typeof val2 === "number") ||
    (typeof val1 === "string" && typeof val2 === "string")
  ) {
    return val1 + val2;
  }

  throw new Error("Must be of type String or Number");
}
```

Good:

```Javascript
function combine(val1, val2) {
  return val1 + val2;
}
```

### Don't over-optimize

Modern browsers do a lot of optimization under-the-hood at runtime. A lot of times, if you are optimizing then you are just wasting your time. There are good resources for seeing where optimization is lacking. Target those in the meantime, until they are fixed if they can be.

Bad:

```Javascript
// On old browsers, each iteration with uncached `list.length` would be costly
// because of `list.length` recomputation. In modern browsers, this is optimized.
for (let i = 0, len = list.length; i < len; i++) {
  // ...
}
```

Good:

```Javascript
for (let i = 0; i < list.length; i++) {
  // ...
}
```


#### Remove dead code

Dead code, like commented code, can be detrimental to the maintainability of your codebase. It serves no purpose and can clutter the code, making it harder to understand and navigate. It is recommended to regularly review your codebase and remove any dead code. If the code is still needed in future, it will be available in the version history.

Bad:

```Javascript
function oldRequestModule(url) {
  // ...
}

function newRequestModule(url) {
  // ...
}

const req = newRequestModule;
inventoryTracker("apples", req, "www.inventory-awesome.io");
```

Good:

```Javascript
function newRequestModule(url) {
  // ...
}

const req = newRequestModule;
inventoryTracker("apples", req, "www.inventory-awesome.io");
```

#### Remove imports you don’t use

Unused imports are not harmful to the performance of the application, but it can make the codebase harder to understand, as it's not clear why those imports are there and if they're still being used or if they're left behind from previous development.

It's a good practice to periodically review your code and remove any imports that are not being used. If you are unsure if an import is being used or not, you can use tools like ESLint or TSLint to check for unused imports.

In general, it's also a good practice to keep the number of imports to a minimum, and only import the functions and modules that are actually needed in your code. This helps to keep the codebase organized, readable, and maintainable.

Unused imports can also increase the bundle size, which can negatively impact the performance of your application.

Bad:

```Javascript
import { functionA, functionB, functionC, functionD, functionE, functionC } from 'utils/validators.js';

const newEmail = functionD('diego@gmail.com');
```

Good:

```Javascript
import { functionD } from 'utils/validators.js';

const newEmail = functionD('diego@gmail.com');
```


### Pipeline functions are your friend

Pipeline functions, also known as "pipe functions" or "flow functions", are a pattern that can be used to improve the readability and maintainability of code.

```Javascript
function somethingA(string) {…}
function somethingB(string) {…}
function somethingC(string) {…}

function mixEverything(string) {
	return somethingA(somethingB(somethingC(string)));
}

mixEverything('hello');
```

Pipeline functions are functions that take a value as input and return a new value as output, which can then be passed as input to the next function in the pipeline. This allows for a more declarative and functional approach to coding, as the code expresses the flow of data and the operations that are being performed on it.

Using pipeline functions can make the code more readable, as it clearly shows the flow of data and the operations being performed on it. It also allows for more modular and reusable code, as the individual functions can be composed and reused in different parts of the application.

Additionally, pipeline functions can make the code more testable, as the individual functions can be tested separately, and the overall flow of the data can be easily tested as well.

In summary, pipeline functions are a good pattern to use because they can improve the readability, maintainability, and testability of code. They allow for a more declarative and functional approach to coding, and they make it easier to understand the flow of data and the operations being performed on it.

Example:

```Javascript
const reverse = async string => string.trim().split('').reverse().join('');

const upperFirst = string => string[0].toUpperCase() + string.substring(1);

const addPeriod = string =>
  string + (string[string.length - 1] === '.' ? '' : '. ');

function reverseSentence = string => upperFirst(addPeriod(reverse(string)));

reverseSentence('yawa dna pu pu'); // 'Up and away.'
```

Pipes functions can be confusing if you make it too deep, but there’s a workaround until we have the native implementation, done with Ramdajs.

```Javascript
import R from 'ramda';

const reverse = async string => string.trim().split('').reverse().join('');

const upperFirst = string => string[0].toUpperCase() + string.substring(1);

const addPeriod = string =>
  string + (string[string.length - 1] === '.' ? '' : '. ');

const reverseSentence = R.pipe(reverse, addPeriod, upperFirst);

reverseSentence('yawa dna pu pu');
```

### Don't use flags as function parameters

This is because flags make the code more complex and harder to understand, as they add another layer of meaning to the function's parameters. Additionally, when multiple flags are used, the number of possible combinations can quickly become overwhelming, making it more difficult to understand how the function works. It's better to use descriptive parameters that clearly express the intent of the function and to use a more descriptive function name if necessary, to help make the code more readable and maintainable.

A bad example:

```Javascript
function processData(data, flag1, flag2) {
  if (data) {
	  // do something
  } else if (flag1) {
	  // do something
  } else {
	  // do something
  }
}
```

A better way to write this function would be to use descriptive parameters that clearly express the intent of the function:

```Javascript
function processData(data, processType) {
  // perform the operation specified by processType
}
```

Or even better:

```Javascript
function processDataAsSum(data) {
  // perform the sum operation
}

function processDataAsProduct(data) {
  // perform the product operation
}
```

### Avoid side-effects

Avoiding side effects in functions is considered a good practice because it makes the code easier to understand, test, and maintain. When a function has side effects, it can affect other parts of the code in unexpected ways, making it more difficult to predict the outcome of the program.

Here's an example of a function with side effects that could be considered a bad practice:

```Javascript
let counter = 0;

function incrementCounter() {
  counter += 1;
}
```

A better way to write this function would be to avoid the side effect and return the new value instead:

```Javascript
function incrementCounter(counter) {
  return counter + 1;
}
```

In JavaScript, some values are fixed and cannot be altered (immutable), while others can be changed (mutable). Objects and arrays are examples of mutable values, therefore, it is crucial to be mindful when passing them as parameters to a function. Functions can modify an object's properties or manipulate the contents of an array, leading to potential bugs in other parts of the code.

## Avoid writing global functions

Writing to global functions is generally considered a bad practice in programming because it can lead to naming conflicts and make the code harder to maintain. Global functions are accessible from anywhere in the code, which means that any part of the code can modify them, making it difficult to predict their behavior. Additionally, if multiple parts of the code write to the same global function, it can result in unexpected behavior and bugs that are difficult to trace.

It's better to avoid writing to global functions and instead use a modular approach, where functions are defined and used within the scope of specific modules or objects. This way, the code is more organized and the relationships between different parts of the code are easier to understand. It also makes it easier to test the code and to prevent unintended interactions between different parts of the code.

Bad:

```Javascript
Array.prototype.diff = function diff(comparisonArray) {
  const hash = new Set(comparisonArray);
  return this.filter(elem => !hash.has(elem));
};
```

Good:

```Javascript
class SuperArray extends Array {
  diff(comparisonArray) {
    const hash = new Set(comparisonArray);
    return this.filter(elem => !hash.has(elem));
  }
}
```

### Favor functional programming over imperative programming

Favor functional programming over imperative programming is considered better in JavaScript for several reasons:

1. **Improved readability:** Functional programming emphasizes immutability and the avoidance of side effects, which can make the code easier to read and understand. This can make it easier to maintain and debug the code.

2. **Better composition:** Functions in functional programming are designed to be composable, which means that they can be easily combined to form more complex operations. This makes it easier to build and scale complex systems in a maintainable way.

3. **Improved testability:** Functions in functional programming are typically small, isolated units of behavior that can be tested in isolation. This makes it easier to test the code and to catch bugs early in the development process.

4. **Improved performance:** Because functional programming avoids shared state and side effects, it can be more efficient than imperative programming, especially in multi-threaded environments.

That being said, functional programming is not the only good approach to programming, and there are certain problems that may be better suited to an imperative or object-oriented approach. It's important to choose the right approach for the specific problem you're trying to solve, rather than always favoring one approach over the others.

Bad:

```Javascript
const programmerOutput = [
  {
    name: "Uncle Bobby",
    linesOfCode: 500
  },
  {
    name: "Suzie Q",
    linesOfCode: 1500
  },
  {
    name: "Jimmy Gosling",
    linesOfCode: 150
  },
  {
    name: "Gracie Hopper",
    linesOfCode: 1000
  }
];

let totalOutput = 0;

for (let i = 0; i < programmerOutput.length; i++) {
  totalOutput += programmerOutput[i].linesOfCode;
}
```

Good:

```Javascript
const programmerOutput = [
  {
    name: "Uncle Bobby",
    linesOfCode: 500
  },
  {
    name: "Suzie Q",
    linesOfCode: 1500
  },
  {
    name: "Jimmy Gosling",
    linesOfCode: 150
  },
  {
    name: "Gracie Hopper",
    linesOfCode: 1000
  }
];

const totalOutput = programmerOutput.reduce(
  (totalLines, output) => totalLines + output.linesOfCode,
  0
);
```


## Concurrency

## Use promises, not callbacks

Using promises instead of callbacks is considered a better practice in JavaScript for several reasons:

1. **Improved readability:** Promises provide a clear and concise way to handle asynchronous operations, making the code easier to read and understand. This can make it easier to maintain and debug the code.

2. **Better error handling:** Promises provide a way to handle errors that are separated from the main flow of the code, making it easier to catch and handle errors in a centralized way.

3. Better composition: Promises can be easily composed and chained, making it easier to build and scale complex systems in a maintainable way. This is particularly useful when dealing with multiple asynchronous operations that depend on each other.

4. Improved performance: Promises can be optimized for performance in certain cases, such as when multiple asynchronous operations are executed in parallel.

That being said, callbacks are still widely used and have their own advantages, such as simplicity and familiarity. The choice between using promises or callbacks depends on the specific requirements of the problem you're trying to solve and your personal preferences as a developer. In general, though, promises are a more modern and flexible approach to handling asynchronous operations in JavaScript.

Bad:

```Javascript
import { get } from "request";
import { writeFile } from "fs";

get(
  "https://en.wikipedia.org/wiki/Robert_Cecil_Martin",
  (requestErr, response, body) => {
    if (requestErr) {
      console.error(requestErr);
    } else {
      writeFile("article.html", body, writeErr => {
        if (writeErr) {
          console.error(writeErr);
        } else {
          console.log("File written");
        }
      });
    }
  }
);
```

Good:

```Javascript
import { get } from "request-promise";
import { writeFile } from "fs-extra";

get("https://en.wikipedia.org/wiki/Robert_Cecil_Martin")
  .then(body => {
    return writeFile("article.html", body);
  })
  .then(() => {
    console.log("File written");
  })
  .catch(err => {
    console.error(err);
  });
```

### Async/Await are even cleaner than Promises

Promises are a very clean alternative to callbacks, but ES2017/ES8 brings async and await which offer an even cleaner solution. All you need is a function that is prefixed in an async keyword, and then you can write your logic imperatively without a then chain of functions. Use this if you can take advantage of ES2017/ES8 features today!

Bad:

```Javascript
import { get } from "request-promise";
import { writeFile } from "fs-extra";

get("https://en.wikipedia.org/wiki/Robert_Cecil_Martin")
  .then(body => {
    return writeFile("article.html", body);
  })
  .then(() => {
    console.log("File written");
  })
  .catch(err => {
    console.error(err);
  });
```

Good:

```Javascript
import { get } from "request-promise";
import { writeFile } from "fs-extra";

async function getCleanCodeArticle() {
  try {
    const body = await get(
      "https://en.wikipedia.org/wiki/Robert_Cecil_Martin"
    );
    await writeFile("article.html", body);
    console.log("File written");
  } catch (err) {
    console.error(err);
  }
}

getCleanCodeArticle()
```

## Error handling

Thrown errors are beneficial! They indicate that the runtime has effectively detected an issue in your program and is stopping the execution of the current stack, terminating the process (in Node), and providing you with a stack trace in the console for better understanding and debugging.

### Don't ignore caught errors

Doing nothing with a caught error doesn't give you the ability to ever fix or react to said error. Logging the error to the console (console.log) isn't much better as often times it can get lost in a sea of things printed to the console. If you wrap any bit of code in a try/catch it means you think an error may occur there and therefore you should have a plan, or create a code path, for when it occurs.

Bad:

```Javascript
try {
  functionThatMightThrow();
} catch (error) {
  console.log(error);
}
```

Good:

```Javascript
try {
  functionThatMightThrow();
} catch (error) {
  // One option (more noisy than console.log):
  console.error(error);
  // Another option:
  notifyUserOfError(error);
  // Another option:
  reportErrorToService(error);
  // OR do all three!
}
```

### Don't ignore rejected promises

For the same reason you shouldn't ignore caught errors from try/catch.

Bad:

```Javascript
getdata()
  .then(data => {
    functionThatMightThrow(data);
  })
  .catch(error => {
    console.log(error);
  });
```

Good:

```Javascript
getdata()
  .then(data => {
    functionThatMightThrow(data);
  })
  .catch(error => {
    // One option (more noisy than console.log):
    console.error(error);
    // Another option:
    notifyUserOfError(error);
    // Another option:
    reportErrorToService(error);
    // OR do all three!
  });
```