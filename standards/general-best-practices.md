# General programming best practices

| Field | Value |
| --- | --- |
| Owner | Engineering |
| Applies to | All Cognativ engineering work, all languages |
| Status | Active |
| Last reviewed | 2026-08-24 |

This guide applies across languages. It collects practices that make code safer and easier to maintain, with examples in the languages Cognativ uses most.

> **Where security requirements live.** This document gives practical, language-level guidance. The mandatory requirements — and the full picture of authentication, authorization, secrets, data handling, dependencies, and everything else — are in the [Secure Software Development Standard](../security/secure-software-development-standard.md). Where the two overlap, the standard governs.

## Contents

- [No passwords in code](#no-passwords-in-code)
- [Avoid dynamic code evaluation](#avoid-dynamic-code-evaluation)
- [Use HTTPS everywhere](#use-https-everywhere)
- [Validate and encode user input](#validate-and-encode-user-input)
- [Hash passwords properly](#hash-passwords-properly)
- [Use a Content Security Policy](#use-a-content-security-policy)
- [Limit request size and rate](#limit-request-size-and-rate)
- [Scan your dependencies](#scan-your-dependencies)
- [Always validate input](#always-validate-input)
- [Always handle errors](#always-handle-errors)
- [Positive conditionals over negative conditionals](#positive-conditionals-over-negative-conditionals)

## No passwords in code

**Never store passwords, API keys, tokens, private keys, or any credential directly in source code.** Hard-coded credentials end up in version control, in CI logs, in forks and clones, and in the hands of anyone who gains read access to the repository — permanently, because Git keeps history.

This is a mandatory requirement, not a preference: see [standard §8 Secrets management](../security/secure-software-development-standard.md#8-secrets-management).

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

Keep files containing credentials out of version control. Add `.env` files, key and certificate files (`*.pem`, `*.p12`, service-account JSON), and local database dumps to `.gitignore`, and commit a `.env.example` with placeholder values so the next engineer knows what to configure.

**If a credential does get committed, rotate it.** Deleting the line, amending the commit, or force-pushing does not undo the exposure — the value must be treated as compromised and replaced, then reported per [standard §26](../security/secure-software-development-standard.md#26-security-incidents). Rotation is the fix.

## Avoid dynamic code evaluation

Executing data as code exposes your application to **code injection**. Never pass anything influenced by user input — request data, URL parameters, `postMessage` payloads, third-party API responses — to a construct that evaluates code:

- `eval()` and `new Function()`;
- `setTimeout()` / `setInterval()` **when the first argument is a string** rather than a function (passing a function is perfectly safe and normal);
- template or expression engines evaluating untrusted templates;
- the equivalents in other languages: `exec`/`eval` in Python, `Invoke-Expression` in PowerShell, dynamic compilation in C#.

The same rule applies to deserializing untrusted data with a format that can construct arbitrary types. See [standard §6](../security/secure-software-development-standard.md#6-input-validation-and-output-encoding).

#### JavaScript example

```javascript
// Don't do this — the input decides what code runs:
const userInput = 'alert("Hello!")';
eval(userInput);

// Don't do this either — string arguments to setTimeout are evaluated:
setTimeout('doSomething(' + userInput + ')', 1000);

// Passing a function is fine:
setTimeout(() => doSomething(userInput), 1000);
```

When behavior must be selected at runtime, dispatch from an allow-list instead of evaluating code:

```javascript
const actions = {
  publish: publishPost,
  archive: archivePost,
  restore: restorePost,
};

function runAction(name, post) {
  const action = actions[name];
  if (!action) {
    throw new Error(`Unsupported action: ${name}`);
  }
  return action(post);
}
```

The set of possible outcomes is now fixed by the code, not by the input.

## Use HTTPS everywhere

All network communication carrying application data must use TLS — browser traffic, API calls, service-to-service calls, and webhooks. See [standard §10](../security/secure-software-development-standard.md#10-encryption).

- Redirect HTTP to HTTPS; do not serve application content over plain HTTP.
- Send an HTTP Strict Transport Security (HSTS) header so browsers refuse to downgrade.
- Never disable certificate validation to make something work locally. Code that skips verification must not reach a shared branch.

#### Example

To enforce HTTPS and prevent insecure connections, add the following HTTP header to your server configuration:

```bash
Strict-Transport-Security: max-age=63072000; includeSubDomains; preload
```

## Validate and encode user input

Treat every input from outside your trust boundary as hostile: request bodies, query strings, headers, cookies, uploaded files, webhook payloads, and third-party API responses.

Two distinct jobs, both required:

1. **Validate on input** — reject anything that is not the expected type, length, range, or format, on the server, before use. Prefer an allow-list (a schema, an enum, a bounded pattern) over trying to strip out bad values.
2. **Encode on output** — escape values for the context they enter: HTML body, HTML attribute, JavaScript, URL, CSS, SQL identifier, shell argument.

Do not write your own escaping. Use the mechanism designed for the context:

| Context | Use |
| --- | --- |
| SQL | Parameterized queries or a safe query API — never string concatenation ([SQL standards](sql.md)) |
| HTML rendering | The framework's automatic escaping (React/JSX, template auto-escaping) |
| Untrusted HTML that must be rendered | A maintained sanitizer such as DOMPurify, then render |
| OS commands | An argument-array API with no shell, arguments validated against an allow-list |
| File paths | Resolve the path, then verify it stays inside the intended directory |

#### JavaScript example — validate with a schema, let the framework encode

```javascript
import { z } from 'zod';

const CommentInput = z.object({
  postId: z.string().uuid(),
  body: z.string().min(1).max(2000),
});

export async function createComment(req, res) {
  const parsed = CommentInput.safeParse(req.body);
  if (!parsed.success) {
    return res.status(400).json({ error: 'Invalid comment' });
  }
  // Store the raw text. React escapes it automatically when rendering:
  //   <p>{comment.body}</p>
  const comment = await comments.insert(parsed.data);
  return res.status(201).json({ id: comment.id });
}
```

Store user input in its original form and encode it at render time. Encoding on the way into the database corrupts the data and still fails the moment the value is rendered in a different context.

If untrusted HTML genuinely has to be rendered — a rich-text field, for example — sanitize it with a maintained library:

```javascript
import DOMPurify from 'dompurify';

const safeHtml = DOMPurify.sanitize(untrustedHtml);
```

Never hand-roll an HTML sanitizer, and never feed untrusted content to `innerHTML` or `dangerouslySetInnerHTML` unsanitized. See [standard §21](../security/secure-software-development-standard.md#21-frontend-security).

## Hash passwords properly

Never store passwords in plain text or in any reversible form. Store only a salted hash produced by a **password-hashing function** — one deliberately designed to be slow and memory-hard:

- **Argon2id** — preferred for new work.
- **scrypt** or **bcrypt** — acceptable and widely available.

General-purpose hashes (MD5, SHA-1, SHA-256, SHA-512) must **not** be used for passwords, salted or not: they are fast, which is exactly what an attacker wants.

Rules:

- Use the library's own `hash` and `verify` functions. They generate the salt and encode the parameters for you — do not manage salts by hand.
- Compare with the library's verify function, never with `==`, and never in a SQL `WHERE password = …` clause ([standard §7](../security/secure-software-development-standard.md#7-database-security)).
- Keep work factors in line with current [OWASP Password Storage](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html) guidance and revisit them periodically as hardware gets faster.
- Better still, do not store passwords at all: delegate authentication to an identity provider where the project allows it ([standard §4](../security/secure-software-development-standard.md#4-authentication)).

#### Example using Argon2id in Node.js

```javascript
import argon2 from 'argon2';

async function hashPassword(password) {
  return argon2.hash(password, { type: argon2.argon2id });
}

async function verifyPassword(hash, password) {
  return argon2.verify(hash, password);
}
```

#### Example using bcrypt in Node.js

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

`bcrypt` remains acceptable, but note its 72-byte input limit and choose a cost factor appropriate to current hardware.

## Use a Content Security Policy

**Content Security Policy (CSP)** is an HTTP response header that limits where the browser may load resources from. It is a strong second line of defense against **XSS** and other injection attacks: even if a payload reaches the page, a good policy stops it from executing or from exfiltrating data.

CSP is a defense in depth measure. It does not replace output encoding and input validation.

#### A reasonable starting policy

```text
Content-Security-Policy: default-src 'self'; script-src 'self'; style-src 'self'; img-src 'self' data:; object-src 'none'; base-uri 'self'; frame-ancestors 'none'
```

#### Allowing a trusted third-party origin

```text
Content-Security-Policy:
    default-src 'self';
    script-src 'self' https://trusted-scripts.example.com;
    style-src 'self' https://trusted-styles.example.com;
    img-src 'self' data: https://trusted-images.example.com;
    connect-src 'self' https://api.example.com;
    object-src 'none';
    base-uri 'self';
    frame-ancestors 'none'
```

#### What to avoid

- **`'unsafe-inline'` in `script-src`** defeats most of the protection CSP offers. Where inline scripts are genuinely required, use a per-response **nonce** or a hash instead:

  ```text
  Content-Security-Policy: script-src 'self' 'nonce-r4nd0mV4lu3PerRequest'
  ```

- **`'unsafe-eval'`** — remove the need for it rather than allowing it.
- **Broad wildcards** such as `script-src https://*` or `default-src *`, which allow any host.
- Deploying a policy without testing it. Roll out with `Content-Security-Policy-Report-Only` first, review the violation reports, then enforce.

Most frameworks can generate the header and nonces for you; prefer that over assembling the string by hand in several places. See [standard §19](../security/secure-software-development-standard.md#19-secure-configuration).

## Limit request size and rate

Rate limiting and request size limits protect availability and make abuse expensive. Apply them to authentication, password reset, registration, search, export, and any expensive or unauthenticated endpoint ([standard §19](../security/secure-software-development-standard.md#19-secure-configuration)).

Also bound the *shape* of what you accept: body size, array lengths, string lengths, page size, JSON nesting depth, and upload size.

```javascript
const rateLimit = require('express-rate-limit');

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // Limit each IP to 100 requests per windowMs
});

app.use(limiter);
```

## Scan your dependencies

Outdated and vulnerable libraries are one of the most common ways applications get compromised. Most of an application's code comes from its dependencies, and Cognativ is responsible for all of it. The full requirements are in [standard §13](../security/secure-software-development-standard.md#13-dependency-and-supply-chain-security); the essentials:

- **Commit the lockfile** and install from it in CI (`npm ci`, `pnpm install --frozen-lockfile`, `dotnet restore --locked-mode`). A build that resolves fresh versions on every run is not reproducible.
- **Scan regularly** — on pull requests and on a schedule, not once at the start of a project.
- **Justify every dependency.** Prefer the standard library or a few lines of local code over a package for trivial functionality, and check the exact package name and publisher before installing: typosquats and lookalike packages are a routine attack.
- **Triage what the scanner reports** ([standard §18](../security/secure-software-development-standard.md#18-vulnerability-management)). Severity is a starting point, not a verdict — judge by whether the vulnerable path is actually reachable and exposed.

Common commands by ecosystem:

| Ecosystem | Audit / scan |
| --- | --- |
| Node.js | `npm audit`, `pnpm audit`, `yarn npm audit` |
| Python | `pip-audit` |
| .NET | `dotnet list package --vulnerable --include-transitive` |
| Java | OWASP Dependency-Check, `mvn versions:display-dependency-updates` |
| Go | `govulncheck ./...` |
| Ruby | `bundle audit` |
| PHP | `composer audit` |
| Containers | An image scanner as part of the build |

Which platform-level scanner is enabled is a per-project decision ([standard §17](../security/secure-software-development-standard.md#17-cicd-security)). Running the ecosystem command locally before opening a pull request costs seconds.


## Always validate input

Input validation prevents both security vulnerabilities and ordinary bugs. The questions to answer are when to check, what to check, and how.

Failing to validate leads to security breaches, corrupted data, and crashes. Validate at the point where data enters your code — and, for anything crossing a trust boundary, validate on the server ([standard §6](../security/secure-software-development-standard.md#6-input-validation-and-output-encoding)).

### When should we check?

Validate arguments wherever a function can be called with something unexpected. Take this function:

```Javascript
function sumAllNumbers(number1, number2) {
	return number1 + number2
}
```

Called with `'2'` and `'3'` it returns `'23'`. Called with `undefined` it returns `NaN`. Neither failure is visible until something downstream breaks.

Decide first whether the function accepts numbers only, or also numeric strings, then enforce exactly that:

```Javascript
function sumAllNumbers(number1, number2) {
  if (!Number.isFinite(number1) || !Number.isFinite(number2)) {
    throw new TypeError('sumAllNumbers expects two finite numbers');
  }

  return number1 + number2;
}
```

Note what changed: `Number.isFinite` rejects strings, `null`, `undefined`, `NaN`, and `Infinity`, and it **accepts `0`**. A `!number1` check would have rejected `0` — a real bug that falsy checks introduce whenever `0` or `''` is a valid value.

If the caller may legitimately pass a numeric string, convert explicitly at the boundary and keep the core function strict:

```Javascript
function parseNumericInput(value, field) {
  const parsed = typeof value === 'string' ? Number(value.trim()) : value;
  if (!Number.isFinite(parsed)) {
    throw new TypeError(`${field} is not a valid number`);
  }
  return parsed;
}

// At the edge of the system:
const total = sumAllNumbers(
  parseNumericInput(req.body.a, 'a'),
  parseNumericInput(req.body.b, 'b'),
);
```

Converting once, at the boundary, keeps the rest of the code free of type-guessing. For anything larger than a couple of fields, use a schema validator rather than hand-written checks.


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

Plan for and handle errors and exceptions appropriately to prevent the application from crashing. Security matters here too: errors must fail closed, and they must not leak stack traces, SQL, internal paths, or sensitive values to end users — see [standard §11](../security/secure-software-development-standard.md#11-error-handling).

In C# and Java, you can use try-catch blocks to catch exceptions and handle them gracefully. In Javascript and Typescript, you can use try-catch statements and return error messages to the user.

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
