# Styling
When the web was born, websites were simply text documents and rendered in monospace font. Fast forward to today, and we can now add color, shapes, animations, gradients, blur, filters, fonts and an nearly infinite amount of style to our websites, making the web a more fun place to be.

In traditional websites and web-apps, we could use inline styles and CSS to change how things looked. Further, in many cases styling is very coupled with application logic. For example, if condition X is met then a button should be red. In React world, this is no different. Because our components are in the end simply HTML tags, we can use selectors to style them as usual. One minor thing to know is that the `class` prop that you would normally use in HTML is instead `className` in React:

```html
<!-- In standard HTML you would do -->
<button class="btn-primary">Click Me</button>
```

```jsx
// but in React (or JSX) this becomes:
const MyButton = () => {
  return <button className="btn-primary">Click Me</button>
}
```

When this react component is rendered to the DOM, it will be converted to HTML as:

```html
<button class="btn-primary">Click Me</button>
```

So if we wanted to style the button using CSS, we would simply use a stylesheet with:

```css
.btn-primary {
  background-color: blue;
}
```

## Inline-Styles
Styling in React using CSS is no different than a website so we're going to skip that and cover something that is React specific, which is inline-styles. 

To style a native HTML tag inline one could do:

```html
<button style="background-color: blue;">Click Me</button>
```

In React, this is achieved using the `style` prop, but instead of passing a string, we pass a JSON object where the keys are the camel-cased CSS names as follows:

```jsx
const MyButton = () => {
  const style = {
    // Note that this will then be converted to background-color
    backgroundColor: 'blue'
  }
  
  return <button style={style}>Click Me</button>
}
```

Of course it does not have to be a separate variable and one can simply use an anonymous object:

```jsx
const MyButton = () => {
  return (
    <button style={{
      backgroundColor: 'blue'
    }}>
      Click Me
    </button>
  )
}
```

This has several advantages because it allows us to:
- create re-usable styles, similar to CSS but using JS
- change style based on application rules
- use a single language JS

However, it does come with some disadvantages:
- no separation of concerns (style/function in the same place)
- not everything is supported (for example animations)

## Styled Components
CSS and inline styles are two ways to style our applications and are available "out of the box" when we create a new React App using Create React App. However, if we want to avoid some of the disadvantages mentioned above and get the best of both worlds, we can install an external dependency called [`styled-components`](https://www.npmjs.com/package/styled-components). Below is an example on how this works:

```jsx
// Bring in the third-party library
import styled from 'styled-components';

// Here we create a new React Component (simply a div) that is styled with the provided CSS:
const StyledPanel = styled.div`
  background-color: red;
  border: 1px solid gray;
  padding: 10px;
  margin: 10px;
`;

const App = () => {

  // As you can see here, StyledPanel can be used like any regular React component

  return <div>
    <StyledPanel>I'm Panel 1</StyledPanel>
    <StyledPanel>I'm Panel 2</StyledPanel>
  </div>
};
```

### Dynamic Styling
One of the main disadvantages of CSS in React is it's difficult to bake in application logic to style our components differently. However, `styled-components` does allow us to pass in `props` into these styled components easily:

```jsx
import styled from 'styled-components';

// Notice how we return the style for the CSS attribute 
// and can change it based on props
const StyledPanel = styled.div`
  background-color: ${props => {
    if (props.type === 'primary') return 'blue';
    else return 'gray';
  }};
  border: 1px solid gray;
  padding: 10px;
  margin: 10px;
`;

const App = () => {
  // We can pass in props to styled components:
  return <div>
    <StyledPanel type="primary">I'm Panel 1</StyledPanel>
    <StyledPanel type="default">I'm Panel 2</StyledPanel>
  </div>
};
```

# State Management

So far we have built pretty static components, which receive props and render something. However, apps are dynamic and the user interface needs to display change based on user inputs, data updates, integrations, etc. There are lots of different approaches to handling application state and keeping the UI in sync with it, each with its own pros and cons. There are hundreds of libraries that cover this single topic as it's probably one of the most crucial things to get right on a user interface. 

## Component State

We will start by learning how state is managed at a component level and then move upwards to managing state at an application level. In the end a component is kind of an application by itself - usually it renders a user interface, users can do something with it, and it reacts to change. 

First, we will write a little function that will help us keep the value of a single variable:

```jsx
const useState = initialValue => {
  let _innerValue = initialValue;

  const setter = newValue => {
    _innerValue = newValue;
  };

  const getter = () => {
    return _innerValue;
  };

  return [getter, setter];
};

const App = () => {
  const [getCount, setCount] = useState(1);

  console.log('Initial Value:', getCount());

  const run = () => {
    setCount(getCount() + 1);
    console.log('Current Value:', getCount());
  };

  return (
    <div>
      <button onClick={run}>Increment {getCount()}</button>
    </div>
  );
};
```

Of course our UI doesn't react to the changes in our variable because it doesn't know about them. We would need to somehow notify it that there was a change. We could spend some time doing that, but instead we're going to introduce hooks. 

### Hooks
 - Functional components have been around for a while, but they weren't too
 popular because you couldn't do much (state, lifecycle, etc). But then hooks
 were introduced.
 - These allow additional functionality into functional components
 - Allow you to reuse stateful logic without changing your component hierarchy
 - Avoid using `this`
 - Classes don't minify well
 - They work side by side so adoption can be gradual
 - Hooks are functions that let you “hook into” React state and lifecycle features from function components.


## `useState`
One of the simplest yet most useful hooks. It returns an array with the current value and a setter function for a single variable for which we can provide an initial value.

```jsx
import { useState } from 'react';

const App = () => {
  const [count, setCount] = useState(1);

  console.log('Initial Value:', count);

  const run = () => {
    setCount(count + 1);
    console.log('Current Value:', count);
  };

  return (
    <div>
      <button onClick={run}>Increment {count}</button>
    </div>
  );
};
```

### Rules about Hooks
1. only at the top level (not inside conditions, loops, etc)
2. only in functional components (or custom hooks)

```jsx
const Invalid = ({ type }) => {
  if (type === 'number') {
    const [value, setValue] = useState(0);
  } else {
    const [value, setValue] = useState('');
  }

  return <div>Nope</div>;
};
```

## `useRef`

Lets imagine we need to use a plain old local variable in our component, we could do something like:

```jsx
const Counter = () => {
  const [counter, setCounter] = useState(0);

  let myVariable = 'initial value';

  return (
    <div>
      <div className="main-panel">
        <div className="display">Counter {counter}</div>
        <div>
          Variable: {myVariable}
        </div>
        <button
          onClick={() => {
            console.log('Updating my variable');
            myVariable = 'updated value';
            console.log('myVariable', myVariable);
            setCounter(counter + 1);
          }}
        >
          +
        </button>
      </div>
    </div>
  );
};
```

This doesn't work, because when the component re-renders, the variable gets redeclared with the initial value. We could use a new `useState` hook to store a new variable, but then any time we update this variable it would cause a re-render, which is not what we want. Instead, we can use a `ref`, which is a variable that can hold a mutable value in it's `.current` property and does not get updated on re-render:

```jsx
const Counter = () => {
  const [counter, setCounter] = useState(0);
  const myVariable = useRef('initial value');

  return (
    <div>
      <div className="main-panel">
        <div className="display">Counter {counter}</div>
        <div>
          Variable: {myVariable.current}
        </div>
        <button
          onClick={() => {
            console.log('Updating my variable');
            myVariable.current = 'updated value';
            console.log('myVariable', myVariable);
            // Note that the only reason the component is re-rendering is because we're calling this setter which mutates the state. If we comment this out then it will not re-render
            setCounter(counter + 1);
          }}
        >
          +
        </button>
      </div>
    </div>
  );
};
```

Refs can also be used to access the DOM elements by binding them as props:

```jsx
const FocusInput = () => {

  const inputRef = useRef()

  useEffect(() => {
    inputRef.current.focus()
  }, [])

  return <input ref={inputRef}/>
}
```

```jsx
const Map = () => {
  const myMap = useRef();

  useEffect(() => {
    new mapboxgl.Map({
      container: myMap.current,
      style: 'mapbox://styles/mapbox/streets-v11', // style URL
      center: [-71.1167, 42.377], // starting position [lng, lat]
      zoom: 14, // starting zoom
    });
  }, []);

  return (
    <div
      className="main-panel"
      ref={myMap}
      style={{ height: 300, width: 300, padding: 0, margin: 0, overflow: 'hidden' }}
    />
  );
};
```