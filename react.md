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