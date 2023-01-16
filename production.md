# React App Hardening

As we mentioned in our very first lecture, Javascript is not a strictly typed language. That gives developers quite a bit of freedom, at the expense of allowing bugs to slip through that could easily be caught by enforcing types. By default, Create React App enforces some rules through ESLint, but this still leaves us exposed. In this lecture we'll explore alternatives to harden our applications and catch bugs before they happen or at least be notified after they do.

### TypeScript

TypeScript, as its website describes, is "... a strongly typed programming language that builds on JavaScript...". In short, it allows us to define types and forces us to adhere to them. Most editors support TypeScript either natively or through plugins, so similarly to ESLint, we are warned as we write our code if any rule is not followed. Futher, TypeScript does not require support from the clients where our code will run as it converts to JavaScript before. 

We will not cover TypeScript in this course but you it's very common and you surely will encounter it in production applications so it's worth learning it. You can read more [here](https://www.typescriptlang.org/). We encourage you to run at least some Hello World examples to get familiar with it.

### Prop-Types

As we know, React is a very opinionated framework. If we don't follow its rules, it will make sure we know about it with verbose and usually gnarly logs. However, when we define components, we have complete freedom in terms of the `props` they can receive (except for some protected names like `children`. This makes sense as it would be quite limiting otherwise. However, it does provide us with a module called PropTypes (`prop-types`, installed separately) to enforce types for the `props` our component receives. This is very useful when our components are to be used by other developers since we can protect them (and us!) from passing incorrect arguments. Let's see how this works:

```jsx
import PropTypes from 'prop-types';

const SimpleButton = ({ label, onClick }) => {
  return (
    <button onClick={onClick}>
      {label}
    </button>
  );
}

// Define the type of properties that we expect
RadioButton.propTypes = {
  label: PropTypes.string,
  onClick: PropTypes.func,
};

const App = () => {
  return <div>
    <SimpleButton
      label="Test 1"
      onClick={() => {
        console.log('clicked 1');
      }}
    />
    <SimpleButton
      label={1} 
      onClick={() => {
        console.log('clicked 2');
      }}
    />
  </div>
}
```

Now if we look at our console, we can notice we have a wordy warning:

<div style="color: red; background: white; padding: 20px; margin: 10px 0px;">
Warning: Failed prop type: Invalid prop `label` of type `number` supplied to `SimpleButton`, expected `string`.
    at SimpleButton (http://localhost:3001/main.299a5a5df3e9c775d348.hot-update.js:25:5)
    at div
    at App
</div>

That's it! We can now start "forcing" devs to comply to our prop-types unless they want to have a noisy console. Note that components will still render, but with possible unexpected behavior. This makes it even more important to always have our dev tools open and making sure we take care of warnings. Let's explore some more options:
<br/>
<br/>

```jsx
MyComponent.propTypes = {
  optionalArray: PropTypes.array,
  optionalBool: PropTypes.bool,
  optionalFunc: PropTypes.func,
  optionalNumber: PropTypes.number,
  optionalString: PropTypes.string,
  optionalSymbol: PropTypes.symbol,
  optionalObject: PropTypes.object,
};

MyComponent.propTypes = {
  optionalNode: PropTypes.node, // anything that can be rendered
  optionalElement: PropTypes.element, // specifically a react element
  optionalElementType: PropTypes.elementType, // a string which is the name of a native HTML element's type ('textearea', 'div') or the actual class {MyClass}
  optionalInstance: PropTypes.instanceOf(MyClass), // an instance of a specific component
};

// Allowing more than 1 type
MyComponent.propTypes = {
  optionalEnum: PropTypes.oneOf(['News', 'Photos']),

  // An object that could be one of many types
  optionalUnion: PropTypes.oneOfType([PropTypes.string, PropTypes.number, PropTypes.instanceOf(Message)]),
};

// Validating arrays
MyComponent.propTypes = {
  // An array of a certain type
  optionalArrayOf: PropTypes.arrayOf(PropTypes.number),
};

// Validating objects
MyComponent.propTypes = {
  // An object with all property values of a certain type
  optionalObjectOf: PropTypes.objectOf(PropTypes.number),
  // Object with shape (allows +- keys)
  optionalObjectWithShape: PropTypes.shape({
    a: PropTypes.string,
    b: PropTypes.number,
  }),
  // Object with specific keys (you can have less keys than these, but not more)
  optionalObjectWithShape: PropTypes.exact({
    a: PropTypes.string,
    b: PropTypes.number,
  }),
};

// So far we haven't made anything required, just specified the type
// Any proptype can be made a required field
Avatar.propTypes = {
  requiredString: PropTypes.objectOf(PropTypes.string).isRequired,
  // Anything is allowed as long as its provided
  requiredAnything: PropTypes.any.isRequired,
};


// Default values
Avatar.defaultProps = {
  // if p1 is not provided, it will be set to 1
  p1: 1,
  // note that in this case, if a user is provided without a fullName,
  // if will NOT be set to 'Default User'. This will only happen if the
  // user prop is not provided at all
  user: {
    fullName: 'Default User',
  },
};

// Its useful to create our own PropTypes so we
// don't have to repeat the definition everywhere
const MyPropTypes = {
  user: PropTypes.exact({
    fullName: PropTypes.string,
    email: PropTypes.string,
  }),
};

// Then use as:
Avatar.propTypes = {
  user: MyPropTypes.user,
};
```

## Client-Side Error Reporting

Ugh, unfortunately we got to the point were a bug slipped through and our application failed. So how does React handle a runtime error? Not gracefully unfortunately. If we are in development mode, it will display an overlay on our app with useful information for debugging and fixing the issue. In production, however, users will simply see a white screen, which is quite a frustrating experience as they are left thinking they made a mistake and have no clue on what to do next. To solve this problem, that is the experience but not the bug, React provides a simple solution yet slightly dated implementation called [Error Boundaries](https://reactjs.org/docs/error-boundaries.html). 

```jsx
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    // Update state so the next render will show the fallback UI.
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    // Handle error, maybe send it to a logging service
  }

  render() {
    if (this.state.hasError) {
      // You can render any custom fallback UI
      return <h1>Something went wrong.</h1>;
    }

    return this.props.children; 
  }
}

// Then we simply need to wrap our application (or single component) with it as:
const Wrapped = () => {
  return <ErrorBoundary>
    <App />
  </ErrorBoundary>;
}
```

As you can see, this implementation uses a `class` component which React has moved away from two major version ago. Further, this implementation has limitations that React explicitly mentions:

Error boundaries do not catch errors for:

- Event handlers (learn more)
- Asynchronous code (e.g. setTimeout or requestAnimationFrame callbacks)
- Server side rendering
- Errors thrown in the error boundary itself (rather than its children)

Because of this, we recommend using a third-party module called [`react-error-boundary`](https://www.npmjs.com/package/react-error-boundary). After installing it, we can use it as:

## react-error-boundary

This module is quite simple to use and adheres quite closely to React's implementation, but fixes most of its caveats. The simplest example (from their website):

```jsx
import { ErrorBoundary } from 'react-error-boundary'

const ErrorFallback = ({ error, resetErrorBoundary }) => (
  <div role="alert">
    <p>Something went wrong:</p>
    <pre>{error.message}</pre>
    <button onClick={resetErrorBoundary}>Try again</button>
  </div>
);

const Wrapped = () => {
  return <ErrorBoundary FallbackComponent={ErrorFallback} onError={(error, errorInfo) => {
    // Handle error, maybe send it to a logging service
  }}>
    <App />
  </ErrorBoundary>;
}
```

## Error Logging
Logging errors is great in development mode, its the easiest and probably most used tool in a programmer's toolbelt. However, once we put our application in the hands of our users, we won't be able to see their consoles (unless you are next to them). There are different ways to address this. 

### Screen Recording
There are services out there that allow us to record user sessions as they use our application. This is useful not only to find errors but also to understand behavior. Some notable ones are [Crazyegg](https://www.crazyegg.com/) or [HotJar](https://hotjar.com/).

### Error Logging
Another approach is to capture any errors that occur in our app, post the error information to a third-party service that will alert us. Hopefully the information we have will be useful enough so we can debug it and fix it. A popular service is [Sentry](https://sentry.io), which provides a very simple React implementation:

```jsx
import React from "react";
import ReactDOM from "react-dom";
import * as Sentry from "@sentry/react";
import { BrowserTracing } from "@sentry/tracing";
import App from "./App";

Sentry.init({
  dsn: '<your dsn here>',
  integrations: [new BrowserTracing()],

  // Set tracesSampleRate to 1.0 to capture 100%
  // of transactions for performance monitoring.
  // We recommend adjusting this value in production
  tracesSampleRate: 1.0,
});

ReactDOM.render(<App />, document.getElementById("root"));
```

Thats it! Any time our application blows up, an error will be reported to our Sentry application. Note that this still requires us to implement our Error Boundaries if we want to avoid the WSOD (white screen of death!).

# Performance

We have talked quite a bit about a component's lifecycle, order of operations, rendering, effects. A simple component does not have a lot complexity, but as soon as we add state, dependencies, changing props, etc, we can easily make mistakes that can degrade performance significantly. 

## Infinite Loops
It is actually very easy to cause infinite rendering loops in React. The two most common are:

### `useEffect` loops
```jsx
const App = () => {
  const [counter, setCounter] = useState(0);

  // Do something in here that updates the counter. this effect triggers itself and will cause an infinite loop. In this case, the problem is easy to spot since the dependencies are explicit
  useEffect(() => {
    setCounter(c => c + 1);
  }, [counter])

  // In this case, the effect also will trigger infinitely, but is harder to debug since the dependencies are not explicit
  useEffect(() => {
    setCounter(c => c + 1);
  })

  return <div>{conuter}</div>;
}
```

### Infinite re-renders
```jsx
const App = () => {
  const [counter, setCounter] = useState(0);

  // Updating a state variable in the main component function will cause it to re-render infinitely.
  setCounter(counter + 1);

  return <div>{conuter}</div>;
}
```

## Optimizing Rendering
Re-rendering a component is not something we should be scared of. After all, React's promise is to make it super efficient to do so, as it will only update the parts of the real DOM that have changed. However, diffing is not free and when are components are large and complex, re-rendering can affect performance and consequently the user experience. Let's explore some tools that we can use to help React and avoid re-rendering when its not needed.

`memo`
In simple terms, a "memoized" value is a cached value that don't gets recalculated unless we say so. In React, wrapping a component with `memo` makes sure that it doesn't re-render unless it's `props` have changed. For example:

```jsx
const Toggle = ({ name, changeToggle }) => {
  console.log('Rendering Toggle:', name);
  return (
    <div>
      <button onClick={changeToggle}>Toggle</button>
    </div>
  );
};

const MemoizedToggle = memo(Toggle);

const App = () => {
  const [count, setCount] = useState(0);
  const [toggle, setToggle] = useState(false);

  const increment = () => {
    setCount(c => c + 1);
  };

  const changeToggle = () => {
    setToggle(!toggle);
  };

  return (
    <div>
      <pre>{JSON.stringify({ count, toggle }, null, 2)}</pre>
      <button onClick={increment}>Increment</button>
      <Toggle name="not using memo"/>
      <MemoizedToggle name="using memo"/>
    </div>
  );
};
```

Note that a memoized component WILL re-render if its internal state (including state coming from out Context) changes. So should you always wrap your components in `memo`? Not really. In most cases its not needed since the performance gains are negligible. This should be used sparsely, only for components that re-render too often and are complex (i.e. the diffing would take considerable time).

`useCallback(func, deps)`
In the above example, we did not provide a handler to the `Toggle` component. If we now pass the `changeToggle` handler, we can see that even the memoized version `MemoizedToggle` re-renders. This is because when `App` re-renders, it creates a new version of the `changeToggle` function. Even though the function itself hasn't changed, its reference has. `MemoizedToggle` sees the new reference as a `prop` change, so it re-renders. To avoid this, we can also memoize our handler using `useCallback` as follows:

```jsx
const changeToggle = useCallback(() => {
  setToggle(!toggle);
}, [toggle]);
```

Now, `changeToggle` will only be regenerated if `toggle` changes.