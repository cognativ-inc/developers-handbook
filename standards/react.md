# React and Next.js

| Field | Value |
| --- | --- |
| Owner | Engineering |
| Applies to | Cognativ frontend engineering |
| Status | Active |
| Last reviewed | 2026-08-24 |

React, specifically through Next.js, is our default frontend library for building user interfaces. Some projects use React alone, but Next.js is the foundation for most setups. Unless otherwise specified, each new project should be built on a current, supported version of Next.js, and TypeScript is the primary language for frontend work.

Bookmark the [Next.js documentation](https://nextjs.org/docs) — the App Router, caching, and server/client component rules change between major versions, so check the docs for the version your project is on rather than relying on memory.

> **Security:** frontend code is never the security boundary. Requirements for XSS, CSP, browser storage, cookies, CSRF, and sensitive state are in [Secure Software Development Standard §21](../security/secure-software-development-standard.md#21-frontend-security). A short summary is at the [end of this document](#frontend-security-essentials).

## Components

To create high-quality components in Next.js, we should adhere to certain guidelines:

### Use Functional Components

Prefer functional components over class components whenever possible, as they are easier to read, write, and maintain.

### Don't use arrow functions

When creating components, avoid using arrow functions. Instead, use the standard function declaration, and export the component by default.

To illustrate, consider the following example:

```typescript
// BadComponent.tsx

import React from 'react';

const BadComponent: React.FC = () => {
  const handleClick = () => {
    console.log('Button clicked!');
  };

  return (
    <button onClick={handleClick}>Click me</button>
  );
};

export default BadComponent;
```

Instead, this is how it should be done:

```typescript
function handleClick() {
  console.log('Button clicked!');
}

export default function WellDoneComponent(){
  return (
    <button onClick={handleClick}>Click me</button>
  );
};
```

### Use meaningful component names

When creating components, ensure that the names are meaningful and clearly describe the component’s purpose, such as “Card,” “Label,” or “Title.” If you need to use a composed name, avoid overly descriptive or verbose names. Instead, aim for concise yet clear names that still convey the component’s functionality. For example, “UserCard” or “ProductLabel” are preferred over excessively detailed names like “CardForUserInformation.”
To illustrate, consider the following example:

```typescript
// WhiteCard.tsx
function WhiteCard(props: {children: React.ReactNode}) {
  const { children } = props;
  return (
    <div className="white-bg">{children}</div>
  );
}
```

Instead make a generic component and accept variables colors:

```typescript
// ColoredCard.tsx
interface ColoredCardProps {
  color: string;
  children: React.ReactNode;
}

export default function ColoredCard({ color, children }: ColoredCardProps) {
  return (
    <div className={`card ${color}`}>
      {children}
    </div>
  );
}
```

### You don't need `import React from 'react'`

With the modern JSX transform (used by Next.js and by React 17 and later), JSX no longer requires `React` to be in scope. Import only the hooks and APIs you actually use:

```typescript
import { useState, useEffect } from 'react';
```

Import types explicitly where you need them (`import type { ReactNode } from 'react'`).

### Server and client components, and `"use client"`

In the Next.js App Router, components are **server components by default**. Add `"use client"` at the top of a file only when that component needs something the server cannot do:

- React state or lifecycle hooks (`useState`, `useReducer`, `useEffect`, `useLayoutEffect`);
- context consumed with `useContext`, or refs;
- browser-only APIs (`window`, `document`, `localStorage`);
- event handlers attached to DOM elements.

`"use client"` marks a boundary: everything imported into that subtree is bundled for the browser. Adding it unnecessarily makes bundles larger, gives up server-side data access, and — most importantly — **puts code and data into the client bundle where the user can read them**.

Keep it as far down the tree as possible: fetch data in a server component, then pass only what is needed into a small interactive client component.

Two rules that follow from this:

- **Never put secrets in a client component.** Anything reachable from a `"use client"` file ships to the browser, as does any environment variable prefixed `NEXT_PUBLIC_`.
- **Don't over-fetch on the server.** Data serialized into a client component's props is visible in the page payload. Select only the fields the UI renders ([standard §21](../security/secure-software-development-standard.md#21-frontend-security)).

The example below is a client component because it uses `useState` and `useEffect`. In current Next.js, this particular pattern — fetching on mount — is usually better expressed as a server component that awaits the data, or with a data-fetching library that handles caching, retries, and error states.


```typescript
// MyComponent.tsx
"use client"

import { useState, useEffect } from 'react';

interface Data {
  // Define your data type here
}

export default function MyComponent() {
  const [data, setData] = useState<Data | null>(null);
  const [loading, setLoading] = useState<boolean>(true);

  useEffect(function fetchData() {
    async function fetchData() {
      try {
        const response = await fetch('https://api.example.com/data');
        const jsonData: Data = await response.json();
        setData(jsonData);
        setLoading(false);
      } catch (error) {
        console.error('Error fetching data:', error);
        setLoading(false);
      }
    }

    fetchData();
  }, []);

  return (
    <div>
      {loading ? (
        <p>Loading...</p>
      ) : data ? (
        <div>
          <h1>Data:</h1>
          <pre>{JSON.stringify(data, null, 2)}</pre>
        </div>
      ) : (
        <p>No data available.</p>
      )}
    </div>
  );
}
```

### Separation of concerns

Follow the principle of single responsibility and break down your components into smaller, reusable pieces. Each component should ideally do one thing and do it well.

```typescript
// ParentComponent.tsx
import Title from './Title';
import Content from './Content';
import Footer from './Footer';

export default function ParentComponent() {
  return (
    <div>
      <Title />
      <Content />
      <Footer />
    </div>
  );
}
```


```typescript
// Title.tsx
export default function Title() {
  return (
    <div>
      <h1>Welcome to my website</h1>
    </div>
  );
}
```

```typescript
// Content.tsx
export default function Footer() {
  return (
    <footer>
      <p>Contact information</p>
    </footer>
  );
}
```

```typescript
// Footer.tsx
export default function Footer() {
  return (
    <footer>
      <p>Contact information</p>
    </footer>
  );
}
```

### Large Components

Refrain from creating monolithic components with excessive logic. Instead, break them down into smaller, more manageable pieces to enhance readability and maintainability. This approach allows each component to focus on a specific task, making the code easier to understand, test, and reuse.

### Mixing UI Logic with Business Logic

Keep your components focused on presentation logic and avoid mixing business logic within them. Extract business logic into separate utility functions or hooks.

```typescript
// Calculator.ts
export function calculateSum(a: number, b: number): number {
  return a + b;
}
```

And:

```typescript
// CalculatorComponent.tsx

import React, { useState } from 'react';
import { calculateSum } from './Calculator';

export default function CalculatorComponent() {
  const [num1, setNum1] = useState<number>(0);
  const [num2, setNum2] = useState<number>(0);
  const [result, setResult] = useState<number>(0);

  function handleCalculate(){
    const sum = calculateSum(num1, num2);
    setResult(sum);
  };

  return (
    <div>
      <h2>Calculator</h2>
      <div>
        <label>Number 1:</label>
        <input type="number" value={num1} onChange={(e) => setNum1(Number(e.target.value))} />
      </div>
      <div>
        <label>Number 2:</label>
        <input type="number" value={num2} onChange={(e) => setNum2(Number(e.target.value))} />
      </div>
      <button onClick={handleCalculate}>Calculate</button>
      <div>
        <label>Result:</label>
        <span>{result}</span>
      </div>
    </div>
  );
}
```

### Ensuring type safety with PropTypes or TypeScript

Learn how to enhance the reliability and maintainability of your React and Next.js applications by implementing type safety measures using either PropTypes or TypeScript. Enforcing strict type checks allows you to catch bugs early in development and streamline collaboration within your team. This tutorial provides guidance on integrating PropTypes for runtime prop validation or leveraging TypeScript for static type checking, empowering you to build robust and error-resistant components. By incorporating these practices, you can ensure your code is more predictable, easier to debug, and better suited for team collaboration.​⬤

```typescript
// Example using TypeScript for static type checking
interface Props {
  name: string;
  age: number;
}

export default function MyComponent({ name, age }: Props): JSX.Element {
  return (
    <div>
      <h1>{`Name: ${name}`}</h1>
      <p>{`Age: ${age}`}</p>
    </div>
  );
}
```

```typescript
// Bad example without using any type safety measures
export default function MyComponent({ name, age }) {
  return (
    <div>
      <h1>{`Name: ${name}`}</h1>
      <p>{`Age: ${age}`}</p>
    </div>
  );
}
```


### Avoid uncontrolled Side Effects

Discover best practices for handling side effects in React components to maintain a clean and maintainable codebase. Avoid triggering side effects directly within a component’s render method, as this can lead to unexpected behavior and performance issues. Instead, utilize the useEffect hook to manage side effects effectively, ensuring they are executed at the appropriate times during the component lifecycle.

This tutorial provides step-by-step guidance on implementing useEffect to manage side effects, including proper cleanup when the component unmounts. By following these practices, you can promote code clarity and robustness, making your components more predictable and easier to maintain.

```typescript
import { useEffect, useState } from 'react';

export default function SideEffectComponent() {
  const [data, setData] = useState(null);

  useEffect(() => {
    // Fetch data when the component mounts
    fetchData();

    // Clean up the effect when the component unmounts
    return () => {
      // Cleanup code here (if necessary)
    };
  }, []); // Empty dependency array ensures the effect runs only once

  const fetchData = async () => {
    try {
      const response = await fetch('https://api.example.com/data');
      const jsonData = await response.json();
      setData(jsonData);
    } catch (error) {
      console.error('Error fetching data:', error);
    }
  };

  return (
    <div>
      {/* Render data here */}
    </div>
  );
}
```

### Avoid hardcoding

Learn how to improve the flexibility and maintainability of your React components by avoiding hardcoding data directly within them. Hardcoded data can lead to inflexibility and make it difficult to manage changes over time. Instead, opt for fetching data from external sources or passing it down as props to ensure components remain adaptable and reusable. This tutorial provides guidance on strategies for retrieving data dynamically, promoting better code organization and long-term maintainability.

The following example shows a hardcoded component that can be improved:

```typescript
import React from 'react';

export default function BadExampleComponent() {
  return (
    <div>
      <h2>List of Items</h2>
      <ul>
        {/* Hardcoded data directly within the component */}
        <li>Item 1</li>
        <li>Item 2</li>
        <li>Item 3</li>
      </ul>
    </div>
  );
}
```
This is how you can improve it:

```typescript

// Hardcoded data
const items = [
  { id: 1, name: 'Item 1' },
  { id: 2, name: 'Item 2' },
  { id: 3, name: 'Item 3' }
];

// Child component receiving data as props
function ItemList({ items }) {
  return (
    <ul>
      {items.map(item => (
        <li key={item.id}>{item.name}</li>
      ))}
    </ul>
  );
}

export default function ParentComponent() {
  return (
    <div>
      <h2>List of Items</h2>
      <ItemList items={items} />
    </div>
  );
}
```

### Improving Component Design: Avoid Prop Chaining in React

Learn how to enhance the readability and maintainability of your React components by avoiding prop chaining. Prop chaining, where props are passed down through multiple levels of components, can lead to code that is difficult to understand and maintain. Instead, consider using alternative techniques like context API or component composition to pass data to components more efficiently while maintaining a clear and concise component hierarchy.

This tutorial provides guidance on identifying and refactoring instances of prop chaining, promoting cleaner and more scalable component design.

```typescript
// ParentComponent.js
import React, { useState } from 'react';
import ChildComponent from './ChildComponent';

export default function ParentComponent() {
  const [data, setData] = useState('Some data');

  return (
    <div>
      <h2>Parent Component</h2>
      <ChildComponent data={data} />
    </div>
  );
}

// ChildComponent.js
import React from 'react';
import GrandchildComponent from './GrandchildComponent';

export default function ChildComponent(props) {
  return (
    <div>
      <h3>Child Component</h3>
      <GrandchildComponent {...props} />
    </div>
  );
}

// GrandchildComponent.js
import React from 'react';

export default function GrandchildComponent(props) {
  return (
    <div>
      <h4>Grandchild Component</h4>
      <p>Data: {props.data}</p>
    </div>
  );
}
```

---

## Frontend security essentials

Full requirements: [Secure Software Development Standard §21](../security/secure-software-development-standard.md#21-frontend-security). The rules that come up most often in React and Next.js work:

- **The frontend is not the security boundary.** Hidden menus, disabled buttons, filtered lists, and client-side route guards are UX. Every one of them needs a corresponding server-side authorization check ([standard §5](../security/secure-software-development-standard.md#5-authorization)).
- **Let JSX escape for you.** `{value}` is safe. `dangerouslySetInnerHTML` is not — never give it untrusted content. If untrusted HTML must be rendered, sanitize it with a maintained library such as DOMPurify first.
- **No secrets in the bundle.** `NEXT_PUBLIC_*` variables are public. API keys, tokens, and private configuration belong on the server, accessed through a route handler or server action.
- **Validate on the server too.** Client-side form validation is for user experience; the server validates independently ([standard §6](../security/secure-software-development-standard.md#6-input-validation-and-output-encoding)). Server actions and route handlers are public endpoints — validate their input and check authorization inside them.
- **Be careful with browser storage.** `localStorage` and `sessionStorage` are readable by any script on the origin. Prefer `HttpOnly`, `Secure`, `SameSite` cookies for session material; don't store long-lived tokens or personal data in the browser.
- **Validate user-controlled URLs** before using them in `href`, `src`, or a redirect — reject `javascript:` and `data:` schemes. Use `rel="noopener noreferrer"` on links opened in a new tab.
- **Third-party scripts run with full access to the page.** Justify each one, and keep them off pages that handle credentials or sensitive data.
- **Don't ship sensitive data to error reporting.** Client-side logging and monitoring tools must not receive tokens or personal data.
