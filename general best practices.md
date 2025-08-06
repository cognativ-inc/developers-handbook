# General programming best practices

This guide will serve for all languages, always trying to make the best practices to improve your programming skills with these tips.

## No passwords in code

One of the most important security practices is to **never store passwords, API keys, or any sensitive information directly in your code**. Hard-coding credentials or sensitive data in source files exposes them to version control systems and anyone with access to the codebase. This can lead to security breaches and compromised applications. Here’s how you can avoid these issues.

### Use environment variables

Instead of hard-coding sensitive data, use environment variables to manage them safely. This allows you to separate configuration settings from your code, making it easier to manage and keep secure. Most programming languages and frameworks provide support for environment variables.

For example, in **Node.js**, you can use the `process.env` object to access environment variables:

```javascript
// Don't do this:
const apiKey = 'your-api-key';

// Instead, use environment variables:
const apiKey = process.env.API_KEY;
```
In Python, use the `os` module to access environment variables:

```python
import os

# Don't do this:
API_KEY = 'your-api-key'

# Instead, use environment variables:
API_KEY = os.getenv('API_KEY')
```

### Use a configuration management system

In larger applications or projects with multiple environments (e.g., development, staging, production), consider using a dedicated configuration management tool or service to handle sensitive information. Examples include:

* AWS Secrets Manager
* Azure Key Vault
* Google Cloud Secret Manager
* Docker secrets for containerized environments

### Never commit sensitive data

Make sure sensitive files, such as those containing environment-specific configurations, are excluded from your version control system. Use `.gitignore` or similar mechanisms to prevent committing files with credentials.

### Avoid using `eval()` and similar functions

Using `eval()` or similar functions like `setTimeout()`, `setInterval()`, or `new Function()` with user-generated content can expose your application to **code injection attacks**. These functions execute code dynamically, and if improperly handled, they can run malicious code on your system.

#### JavaScript Example:

```javascript
// Don't do this:
const userInput = 'alert("Hello!")';
eval(userInput); // Risky: Could execute malicious code

// Instead, find safer alternatives, such as explicit conditional checks:

const userInput = 'someInput';
if (userInput === 'expectedValue') {
  // Handle input safely
  console.log('Input is valid');
} else {
  console.log('Invalid input');
}

```
### Use HTTPS Everywhere

Ensure your application communicates over **HTTPS** to protect the integrity and confidentiality of data transmitted between the client and server. All sensitive information, including login credentials, should be transmitted securely using TLS (Transport Layer Security).

- Enforce HTTPS by redirecting HTTP requests to HTTPS.
- Use HTTP Strict Transport Security (HSTS) headers to prevent man-in-the-middle attacks.

#### Example:

To enforce HTTPS and prevent insecure connections, add the following HTTP header to your server configuration:

```bash
Strict-Transport-Security: max-age=63072000; includeSubDomains; preload
```

### Sanitize User Inputs

User input must always be treated as potentially malicious. Failing to sanitize input can lead to **Cross-Site Scripting (XSS)**, **SQL Injection**, or other forms of attack. Always sanitize and validate inputs before processing them, especially when dealing with dynamic content or database queries.

#### JavaScript Example:

```javascript
function sanitizeInput(input) {
  const div = document.createElement('div');
  div.appendChild(document.createTextNode(input));
  return div.innerHTML;
}

const userInput = "<script>alert('XSS')</script>";
console.log(sanitizeInput(userInput)); // Outputs sanitized input
```

### Hash and Salt Passwords

Never store passwords as plain text. Always hash and salt passwords before storing them in your database. This ensures that even if your database is compromised, attackers cannot easily retrieve the original passwords. 

Hashing converts passwords into a fixed-length string of characters, while salting adds an extra layer of security by appending a unique value (the salt) to the password before hashing. This makes it much harder for attackers to use precomputed dictionaries (rainbow tables) to crack the passwords.

#### Example using bcrypt in Node.js:

```javascript
const bcrypt = require('bcrypt');
const saltRounds = 10;

// Hashing a password
const hashPassword = async (password) => {
  const salt = await bcrypt.genSalt(saltRounds);
  return await bcrypt.hash(password, salt);
};

// Verifying a password
const verifyPassword = async (inputPassword, hashedPassword) => {
  return await bcrypt.compare(inputPassword, hashedPassword);
};

// Example usage
const password = 'mySecurePassword';
const hashedPassword = await hashPassword(password);

const isPasswordCorrect = await verifyPassword('userInputPassword', hashedPassword);
console.log(isPasswordCorrect ? 'Password is correct' : 'Invalid password');
```
### Use Content Security Policy (CSP)

**Content Security Policy (CSP)** is an HTTP header that helps protect your website from **Cross-Site Scripting (XSS)** and other code injection attacks by controlling which resources can be loaded on your web pages. It acts as a whitelist, allowing only trusted sources of content (such as scripts, styles, or media) to be executed by the browser.

#### Example of a basic CSP:

```bash
Content-Security-Policy: default-src 'self'; img-src https://*; child-src 'none';
```

#### More complex example:

```bash
Content-Security-Policy: 
    default-src 'self'; 
    script-src 'self' 'unsafe-inline' https://trusted-scripts.com; 
    style-src 'self' 'unsafe-inline' https://trusted-styles.com;
    img-src 'self' https://trusted-images.com;
    object-src 'none';
```

#### Example of a basic CSP:

```bash
Content-Security-Policy: default-src 'self'; img-src https://*; child-src 'none';
```

#### Why CSP is important:

* Mitigates XSS attacks: By restricting the sources from which scripts can be loaded, CSP can help prevent malicious scripts from being executed in your web application.
* Reduces attack surface: By limiting where content can be loaded from, CSP reduces the potential avenues an attacker can exploit to inject malicious content into your site.

## Limit Request Size and Rate

Rate limiting and request size limiting help prevent denial-of-service (DoS) attacks or abuse by controlling how much and how often data can be sent to your servers.

```javascript
const rateLimit = require('express-rate-limit');

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // Limit each IP to 100 requests per windowMs
});

app.use(limiter);
```

## Use Dependency Scanners

Outdated or vulnerable libraries are a common security threat. Always use automated tools to scan your dependencies for vulnerabilities and regularly update them.

* Node.js: Use npm audit or yarn audit.
* Python: Use pip-audit.
* Java: Use tools like OWASP Dependency-Check.


## Always Input validate

Input validation is a crucial aspect of programming in any language. It is essential to check the inputs before processing the outputs to prevent potential security vulnerabilities and unexpected behavior. The questions that arise are when to check, what to check, and how to check.

Failing to properly validate inputs can result in critical issues such as security breaches, data loss, and unexpected application crashes. Therefore, it's essential to validate inputs at the appropriate stages of processing, verifying the data type, range of values, and any other necessary conditions.

### When we should check?

Each time we have arguments, we must check them. For example:

```Javascript
function sumAllNumbers(number1, number2) {
	return number1 + number2
}
```

In this simple example, we have the option to do several checks and act accordingly. For example, we first must check if `number1` and `number2` are available before going to the next one, that must be if both props are the type of `number`.

```Javascript
function sumAllNumbers(number1, number2) {
  if (!number1 || !number2) {
    throw new Error('Number 1 or Number 2 is missing');
  }

  const regex = /^[0-9]+$/;
  if (!regex.test(number1) || !regex.test(number2)) {
    throw new Error('Number1 and Number2 cannot be converted into numbers.');
  }

  if (typeof number1 === 'string' || typeof number2 === 'string') {
    number1 = parseInt(number1);
    number2 = parseInt(number2);
    return number1 + number2;
  }

  return number1 + number2;
}
```


### Always check for nulls, undefined before running the entire function

Commonly referred to as "null checking" or "null/undefined validation". This helps to ensure that the code runs as expected and avoid potential errors or unexpected behavior that could occur when working with variables that have not been properly initialized or assigned a value. The specific approach to null checking may vary depending on the programming language and development practices, but it is a common and important step in the overall process of software development.

```Javascript
function printName(name) {
  if (name === null || name === undefined) {
    console.log("Name is not provided");
    return;
  }

  console.log("Name is: " + name);
}
```

In this example, we explicitly check for `null` and `undefined`, but we can extend this with the `!` operator in JavaScript will return `true` if the value on the right-hand side is falsy, which includes `null`, `undefined`, `0`, `NaN`, `''` (empty string), and `false`. In this case, the code checks if name is *falsy*, and if so, logs a message indicating that the name is not provided and exits the function.

In Typescript:

```Javascript
function printName(name: string | null | undefined) {
  if (name === null || name === undefined) {
    console.log("Name is not provided");
    return;
  }

  console.log("Name is: " + name);
}
```

In this example, the `name` parameter is declared as a union type of `string`, `null`, and `undefined`. This allows the function to accept any of these three types as a valid value for the name parameter. The function then checks if the name variable is `null` or `undefined` before attempting to use it. If the variable is `null` or `undefined`, a message is logged indicating that the name is not provided, and the function exits. If the variable is defined and not `null` or `undefined`, the function logs a message indicating the `name`.

Or another way:

```Javascript
function printName(name) {
  if (!name) {
    console.log("Name is not provided");
    return;
  }

  console.log("Name is: " + name);
}
```

Here’s an example in C#:

```C#
using System;

namespace NullCheckExample
{
    class Program
    {
        static void Main(string[] args)
        {
            string name = null;

            if (string.IsNullOrEmpty(name))
            {
                Console.WriteLine("Name is not provided");
                return;
            }

            Console.WriteLine("Name is: " + name);
        }
    }
}
```

Or in Java:

```Java
public class NullCheckExample {
    public static void printName(String name) {
        if (name == null) {
            System.out.println("Name is not provided");
            return;
        }

        System.out.println("Name is: " + name);
    }

    public static void main(String[] args) {
        String name = null;
        printName(name);
    }
}
```

Null checking should be a number one priority when writing code. Even if you think your code will be perfectly executed by your application, you’re wrong. Your code will be executed by automation tests, unit tests, CLI, etc.

### Data Type Validation

In programming, it's important to ensure that data is of the correct type. Incorrect data types can result in unexpected behavior and errors in your application. Some common data types include numbers, strings, booleans, arrays, and objects.
Javascript
```Javascript
// Check if variable is a number
if (typeof variable !== 'number') {
  console.error('Variable must be a number');
}

// Check if variable is a string
if (typeof variable !== 'string') {
  console.error('Variable must be a string');
}
```

In C#

```C#
// Check if variable is an int
if (!(variable is int)) {
  Console.WriteLine("Variable must be an integer");
}

// Check if variable is a string
if (!(variable is string)) {
  Console.WriteLine("Variable must be a string");
}
```

In TypeScript, data type checking is performed at compile time, not runtime. This means that TypeScript will catch type mismatches before the code is executed, allowing you to fix them early in the development process. However, it's still good practice to validate inputs and check for correct data types to prevent potential issues and improve the overall quality and maintainability of your code.

In TypeScript, you can use type annotations and interfaces to explicitly specify the expected data type of a variable, and the TypeScript compiler will check for type mismatches at compile time. Additionally, you can use type guards to perform type checking within your code.

For example:

```Typescript
function add(a: number, b: number): number {
  return a + b;
}

const result = add(1, 2);
console.log(result); // 3
```

In this example, the function add expects two arguments of type number, and the TypeScript compiler will raise an error if an argument of a different type is passed to the function.

So while data type checking is not strictly necessary in TypeScript, it is still a recommended best practice to validate inputs and ensure that data is of the correct type.

### Check data ranges

Checking the range of values and verifying that user input meets certain conditions are also important practices in programming. This helps ensure the validity and consistency of data in your application.

JavaScript:

```Javascript
function validateAge(age) {
  if (age >= 18 && age <= 65) {
    console.log('Age is valid');
  } else {
    console.log('Age is not valid');
  }
}

validateAge(30); // Output: Age is valid
validateAge(70); // Output: Age is not valid
```

In Java:

```Java
public static void validateAge(int age) {
  if (age >= 18 && age <= 65) {
    System.out.println("Age is valid");
  } else {
    System.out.println("Age is not valid");
  }
}

public static void main(String[] args) {
  validateAge(30); // Output: Age is valid
  validateAge(70); // Output: Age is not valid
}
```


And this goes along with using functions as positive conditionals:

```Javascript
let age = 1;

function invalidAgeRequirement(age) {
  if (!age) return false;
  // returns true if age is less or equal than 18 or greater than 65
  if (age <= 18 || age > 65) return true;
}

if (invalidAgeRequirement(age)) {
  return console.error('Invalid age value: ' + age);
}

console.log(`the age of ${age} is valid`);
```


## Always handle errors

Plan for and handle errors and exceptions appropriately to prevent the application from crashing. In C# and Java, you can use try-catch blocks to catch exceptions and handle them gracefully. In Javascript and Typescript, you can use try-catch statements and return error messages to the user.

Try-catch is the most clean way to do it, specially in Javascript because of the use of callbacks and promises.

Javascript:

```Javascript
try {
  console.log(a);
} catch (error) {
  console.error('Error:', error);
}
```

Java:

```Java
try {
  int result = 100 / 0;
} catch (ArithmeticException e) {
  System.out.println("Error: " + e.getMessage());
}
```

C#:

```C#
try {
  int result = 100 / 0;
} catch (DivideByZeroException ex) {
  Console.WriteLine("Error: " + ex.Message);
}
```

Although in the examples we use `console.log` as an action our advice is avoiding that. 

### Handle caught errors effectively

Ignoring caught errors hinders your ability to detect and resolve them. Simply logging the error to the console with `console.log` may not provide a lasting solution as it can easily get buried among other console output. Whenever you wrap code in a `try/catch` block, it implies that you expect an error to occur and thus, it is imperative to have a well-defined plan or alternative code path for handling such situations.

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

## Positive conditionals over negative conditionals

Using positive conditionals instead of negative ones can improve the readability and clarity of your code. When writing conditionals, it's ideal to have as few of them as possible, as this makes the functions simpler and easier to understand. Positive statements, which assert what should be true, are more straightforward than negative statements, which assert what should not be true. This results in code that is easier to follow and debug.

When you want to use a conditional, try to make the invalid case positively, example:

```Javascript
if (!person.age > 18) {
  console.log('You are not allowed to enter the bar');
}
```

The expression `!person.age > 18` is not correct.

The `!` operator has higher precedence than the `>` operator, so the expression `!person.age > 18` is equivalent to `!(person.age > 18)`, which means "not greater than 18". This would not produce the expected results.

A correct expression would be:

```Javascript
if (person.age <= 18) {
  return 'You are not allowed to enter the bar';
}
```

### Abstract conditions into functions

abstracting conditions into functions is a good practice. By doing so, you can:

1. **Improve code readability:** By breaking down complex conditions into smaller, reusable functions, you make your code more understandable and maintainable.
2. **Encourage code reuse:** By defining a condition as a separate function, you can easily reuse it in multiple parts of your code, reducing the amount of duplicated code.
3. **Promote modularity:** Functions can be thought of as building blocks that can be combined to solve larger problems. By abstracting conditions into functions, you can build more modular and flexible systems.

Here is a simple example to illustrate the concept:

```Javascript
function isMinor(person) {
  return person.age < 18;
}

if (isMinor(person)) {
  console.log('You are not allowed to enter the bar');
}
```

In this example, the condition `person.age < 18` is abstracted into a separate function, `isPersonIsMinor`, making the code more readable and reusable. This also makes it easier to test and maintain the condition, as it can be modified in one place without affecting the rest of the code.

You can apply this principle to most programming languages, for example in .NET:

```C#
public class Person
{
    public int Age { get; set; }
}

public static class AgeChecker
{
    public static bool IsMinor(Person person)
    {
        return person.Age < 18;
    }
}

class Program
{
    static void Main(string[] args)
    {
        var person = new Person { Age = 17 };

        if (AgeChecker.IsMinor(person))
        {
            Console.WriteLine("You are not allowed to enter the bar");
        }
        else
        {
            Console.WriteLine("You are allowed to enter the bar");
        }

        Console.ReadKey();
    }
}
```
