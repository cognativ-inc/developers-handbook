# React and Next.js

React, through Next.js, is our defacto frontend library to work on the frontend side. We may have projects were we only do React, but in general terms, Next.js will be the base setup. Each new project, unless exceptions, should be done on the latest Next.js built. Typescript is the base language for coding any frontend application.

In order to have a good development time with Next.js, we recommend you to bookmark the [Next.js Docs](https://nextjs.org/docs).

## Components

To make good components in Next.js, we should respect certain rules:

### Use Functional Components

Prefer functional components over class components whenever possible. They are easier to read, write, and maintain.

### Don't use arrow functions

When creating components, do not use arrow functions, instead, use the standard way to declare functions and export them by default.

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

When creating components, use meaningful names. Names should descrive what the component does: "Card", "Label", "Title", etc. Sometimes you may find in need to use a composed name, for such cases, avoid descriptive names.

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

### Remove import React from 'react';

Since we're using Next, we don't need to use `import React from 'react';`.

### "use client" and React Hooks

Leverage React Hooks such as useState, useEffect, useContext, etc., for managing state, side effects, and context within your components. As a rule, in Next.js you must add `"use client"`.


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

Refrain from creating monolithic components with excessive logic. Break them down into smaller, more manageable pieces to improve readability and maintainability.

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

  const handleCalculate = () => {
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
###  Ensuring Type Safety in React/Next.js with PropTypes or TypeScript

Learn how to enhance the reliability and maintainability of your React and Next.js applications by implementing type safety measures using either PropTypes or TypeScript. By enforcing strict type checks, you can catch bugs early in development and streamline collaboration within your team. This tutorial provides guidance on integrating PropTypes for prop validation or leveraging TypeScript for static type checking, empowering you to build robust and error-resistant components.

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

Discover best practices for handling side effects in React components to ensure a clean and maintainable codebase. Avoid triggering side effects directly within components' render methods, as this can lead to unexpected behavior and performance issues. Instead, utilize the useEffect hook to manage side effects effectively, ensuring they are executed at the appropriate times during the component lifecycle. This tutorial provides step-by-step guidance on implementing useEffect to manage side effects and ensure proper cleanup when the component unmounts, promoting code clarity and robustness.

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

Learn how to improve the flexibility and maintainability of your React components by avoiding hardcoding data directly within them. Hardcoded data can lead to inflexibility and difficulty in managing changes over time. Instead, opt for fetching data from external sources or passing it down as props to ensure components remain adaptable and reusable. This tutorial provides guidance on strategies for retrieving data dynamically, promoting better code organization and long-term maintainability.

The following example shows a hardcoded code that can be improved:

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