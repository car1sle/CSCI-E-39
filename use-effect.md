## Review

Last class we covered how to provide state management to components. We introduced the `useState` hook:

```jsx
const Counter = () => {
  const [count, setCount] = useState(0);

  return (
    <div className="main-panel">
      <div className="display">Current {count}</div>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}
```

Lastly, we talked about `useRef`, which gives us a way to store values over time and modify them without triggering a re-render. It also gives us a way to bind to the DOM without querying the DOM directly:

```jsx
const Counter = () => {
  const [trigger, setTrigger] = useState(true);
  const count = useRef(0);

  return (
    <div className="main-panel">
      <div className="display">Current {count.current}</div>
      <button
        onClick={() => {
          count.current += 1;
        }}
      >
        Increment
      </button>
      <button
        onClick={() => {
          setTrigger(!trigger);
        }}
      >
        Re-render
      </button>
    </div>
  );
};
```

## `useEffect`

In any given system, making a change can have what are called "side-effects", which is simply causing a change to some state after a modification. These changes can be intentional or unintentional but in all cases should be controlled or at least accounted for. In React, we can create side effects *after* a change is made to a component by listening to changes in any variable. It's important to note the *after* keyword here, which means the effect runs once changes are flushed to the DOM and not before.

```jsx
const Counter = () => {
  const [counter, setCounter] = useState(0);

  useEffect(() => {
    console.log('run update', counter);
  });

  return (
    <div className="main-panel">
      <div className="display">Counter {counter}</div>
      <button onClick={() => setCounter(counter + 1)}>+</button>
    </div>
  );
};
```

In this case, we have created an effect that is fired any time the component re-renders. We can also specify a dependency array with the list of variables we want to listen to exclusively.

```jsx
const Counter = () => {
  const [counter, setCounter] = useState(0);
  const [other, setOther] = useState(0);

  useEffect(() => {
    console.log('run update', counter);
  }, [counter]);

  return (
    <div>
      <div className="main-panel">
        <div className="display">Counter {counter}</div>
        <button onClick={() => setCounter(counter + 1)}>+</button>
      </div>
      <div className="main-panel">
        <div className="display">Other {other}</div>
        <button onClick={() => setOther(other + 1)}>+</button>
      </div>
    </div>
  );
};
```

It is very common to need to execute something after a components get mounted but never again, for example to fetch some data required by the component. We can easily do this by specifying an empty dependency array, which will effectively run the effect once after mount and never again:

```jsx
const Counter = () => {
  const [counter, setCounter] = useState(null);

  useEffect(() => {
    // Simulate an async operation
    setTimeout(() => {
      setCounter(0);
    }, 3000)
  }, []);

  // When it mounts, its not ready since counter has not been set
  if (counter === null) {
    return <div>Counter is not ready...</div>
  }

  return (
    <div>
      <div className="main-panel">
        <div className="display">Counter {counter}</div>
        <button onClick={() => setCounter(counter + 1)}>+</button>
      </div>
    </div>
  );
};
```

### Cleaning Up after side-effects

JS is a high level language that manages garbage collection for us so we don't need to worry about allocating/deallocating memory, etc. However, it's very important that any time we set up an effect, we clean up after its no longer needed to avoid memory leaks. This is useful for example if we have set up subscriptions (like subscribing to an API), listeners to events, intervals or timeouts, etc. The clean-up function should be returned from `useEffect` as follows:

```jsx
const App = () => {
  
  useEffect(() => {
    API.subscribe();

    return () => {
      API.unsubscribe();
    }
  }, []);

  return (
    <div>API Subscription</div>
  );
};
```

So when does `useEffect` run? It's important to understand the order of operations when a component renders:

- render
- run clean up functions of previous `useEffect` (note that we still have old values here because the effect was declared in the previous render)
- run `useEffect` (here we have the new values)

```jsx
const Counter = () => {
  const [counter, setCounter] = useState(0);

  useEffect(() => {
    console.log('Running useEffect with no dependencies');

    return () => {
      console.log('Running clean-up for useEffect with no dependencies');
    }
  }, []);

  useEffect(() => {
    console.log('Running useEffect - counter is', counter);

    return () => {
      console.log('Running clean-up - counter is', counter);
    }
  }, [counter]);

  console.log('Rendering...');

  return (
    <div>
      <div className="main-panel">
        <div className="display">Counter {counter}</div>
        <button onClick={() => setCounter(counter + 1)}>+</button>
      </div>
    </div>
  );
};
```

Lets look at another example:

```jsx
const Even = ({ number }) => {
  useEffect(() => {
    console.log('updated Even');
    return () => {
      console.log('unmounted Even');
    };
  });

  return `Even ${number}`;
};

const Odd = ({ number }) => {
  useEffect(() => {
    console.log('updated Odd');
    return () => {
      console.log('unmounted Odd');
    };
  });

  return `Odd ${number}`;
};

const App = () => {
  const [counter, setCounter] = useState(0);

  return (
    <div className="main-panel">
      <div className="display">{counter % 2 ? <Odd number={counter} /> : <Even number={counter} />}</div>
      <button onClick={() => setCounter(counter + 1)}>Inc</button>
    </div>
  );
};
```

What if we forget to clean up? In many cases, we won't notice, which is why it's even more important to remember to clean up. One common example of forgetting to clean up is trying to update a component that has already been unmounted:

```jsx
const TroubleMaker = () => {
  const [trouble, setTrouble] = useState(false);
  
  useEffect(() => {
    const t = setTimeout(() => {
      console.log('Updating TroubleMaker');
      
      // This will cause an update on an unmounted component so react will warn us but note that if there was no update, this would go unnoticed
      setTrouble(true);
    }, 5000)

    // If we do clean up, there's no problem
    // return () => clearTimeout(t);
  }, [])

  return <div>I'm about to cause trouble...{trouble}</div>
}

const App = () => {
  const [mounted, setMounted] = useState(true);

  useEffect(() => {
    setTimeout(() => {
      console.log('Unmounting TroubleMaker');
      setMounted(false);
    }, 2000)
  }, []);

  console.log('Rendering...');

  return (
    <div>
      <div>Trouble Maker mounted? {mounted}</div>
      {mounted && <TroubleMaker/>}
    </div>
  );
};
```

Lets try an interesting example using timers:

```jsx
const Counter = () => {
  const [count, setCount] = useState(0);

  useEffect(() => {
    setTimeout(() => {
      setCount(count + 1);
    }, 5000);
  }, [])

  return (
    <div className="main-panel">
      <div className="display">Current {count}</div>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}
```

As you can see, clicking the button increments the count but when the timeout function fires, it has an old value. This is because when we *registered* the timeout function on mount (no dependencies in the array), the value was 0 and that's what it will use when it fires. The callback passed to `setTimeout` is not being updated. To fix this we can use a different version of the `setter` returned by `useState`. This receives a function which is passed the previous value of the state and returns the new value:

```jsx
const Counter = () => {
  const [count, setCount] = useState(0);

  useEffect(() => {
    setTimeout(() => {
      setCount(prevCount => {
        return prevCount + 1
      });
    }, 5000);
  }, [])

  return (
    <div className="main-panel">
      <div className="display">Current {count}</div>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}
```

This works well, but sometimes we want to use updated values in our callback functions without using a setter function. 

```jsx
const Counter = () => {
  const [trigger, setTrigger] = useState(true);
  const [var1, setVar1] = useState('');

  const var2 = useRef('');

  useEffect(() => {
    setTimeout(() => {
      console.log('Var1', var1);
      console.log('Var2', var2);
    }, 5000);
  }, []);

  return (
    <div className="main-panel">
      <input
        value={var1}
        onChange={v => {
          setVar1(v.target.value);
          var2.current = `the value of var1 is ${var1}`;
        }}
      />
    </div>
  );
};
```

Something interesting happened here - as you can see, `var2.current` logged the value of `var1` but missed the last change. If we typed `abc`, it will log `ab`. This is because `useState` is *asynchronous*. When we set the value, its not immediately available which is why we can only rely on `useEffect` to get the updated value.