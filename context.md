## Custom Hooks

We talked about how we need to clean up after ourselves when we create effects. This avoids memory leaks and in general any unexpected behavior when an effect is triggered on an unmounted component. Lets look at an example:

```jsx
const WindowListener = () => {
  const [position, setPosition] = useState({ x: 0, y: 0 });

  const mouseMoveListener = e => {
    setPosition({ x: e.clientX - window.innerWidth / 2, y: e.clientY - window.innerHeight / 2 });
  };

  useEffect(() => {
    window.addEventListener('mousemove', mouseMoveListener);
    return () => window.removeEventListener('mousemove', mouseMoveListener);
  }, []);

  return (
    <div>
      <div>
        <b>X: {position.x}</b>
      </div>
      <div>
        <b>Y: {position.y}</b>
      </div>
    </div>
  );
};
```

This works pretty well. Now imagine we want to add another listener on the window, to listen to the `'click'` event anywhere on the screen. We could do this by adding another effect as:

```jsx
// ...
const clickListener = e => {
  console.log('Clicked the window!');
};

useEffect(() => {
  window.addEventListener('click', clickListener);
  return () => window.removeEventListener('click', clickListener);
}, []);
// ...
```

Again this works great. However, you can see that there's a pattern here. Every time we add a listener we have to remember to clean it. To refactor this into a reusable block, we will introduce custom hooks. Custom hooks are essentially traditional JS functions with the following rules:

- name starts with `use`
- can make use of other hooks in its body

### Writing a Custom Hook

```jsx
const useListener = (event, handler) => {
  useEffect(() => {
    window.addEventListener(event, handler);
    return () => window.removeEventListener(event, handler);
  }, [event, handler]);
};
```

And now our listener is much simpler:

```jsx
const WindowListener = () => {
  const [position, setPosition] = useState({ x: 0, y: 0 });

  useListener('mousemove', e => {
    setPosition({ x: e.clientX - window.innerWidth / 2, y: e.clientY - window.innerHeight / 2 });
  });
  
  useListener('click', () => {
    console.log('Clicked the window!');
  });

  return (
    <div>
      <div>
        <b>X: {position.x}</b>
      </div>
      <div>
        <b>Y: {position.y}</b>
      </div>
    </div>
  );
};
```

Lets look at another example of a useful hook that helps us avoid memory leaks:

```jsx
const useTimeout = (handler, milliseconds) => {
  useEffect(() => {
    const t = setTimeout(handler, milliseconds)
    return () => {
      clearTimeout(t);
    }
  })
}
```

And now a more complicated one to review, `usePrevious`. Note that the source for this is from [usehooks.com](https://usehooks.com/usePrevious/). There are lots of unofficial hooks that people have written and open sourced that are extremely handy. However, it's always recommended to look at the source and try to learn how it works to increase your own skill and understanding of React.

## `usePrevious`

The requirement is to write a component that accepts user input and allows undoing the last action. Our first attempt is using a `ref` as follows:

```jsx
const DiffInput = () => {
  const [value, setValue] = useState('');
  const previous = useRef(value);

  useEffect(() => {
    previous.current = value;
  }, [value]);

  return (
    <div>
      <input type="text" value={value} onChange={e => setValue(e.target.value)} />
      <div>
        <b>Previous:</b> {previous.current}
      </div>
      <br />
      <button
        onClick={() => {
          console.log('previous.current', previous.current);
          setValue(previous.current);
        }}
      >
        Undo
      </button>
    </div>
  );
};
```

This kind of works - lets follow the order of operations after typing `ab`:

- type `a`
  - `input` triggers `onChange` and sets `value` to `a`
  - component re-renders with state:

    ```js
    value = 'a'
    previous.current = ''
    ```

  - effect triggers -> `previous.current` is set to `a`

- type `b`
  - `input` triggers `onChange` and sets `value` to `ab`
  
  ```js
    value = 'ab'
    // This is the important bit - previous.current is always trailing one render behind
    previous.current = 'a'
  ```

  - effect triggers -> `previous.current` is set to `ab`

But we're not quite there yet. If we click the **Undo** button we can see that the value is not set to the previous value but to the current one. This is because the `onClick` handler of the button is fetching the value by reference (i.e. reading the value from **location** of `previous.current`) and by the time we click the button, it already has the new value. Lets refactor this a bit:

```jsx
const usePrevious = value => {
  const previous = useRef();

  useEffect(() => {
    previous.current = value;
  }, [value]);

  return previous.current;
};

const DiffInput = () => {
  const [value, setValue] = useState('');
  const previous = usePrevious(value);

  return (
    <div>
      <input type="text" value={value} onChange={e => setValue(e.target.value)} />
      <div>
        <b>Previous:</b> {previous}
      </div>
      <br />
      <button
        onClick={() => {
          setValue(previous);
        }}
      >
        Undo
      </button>
    </div>
  );
};
```

Now, our `usePrevious` hook hides the `ref` from us and only gives us access to its value. When the render occurs and the `onClick` handler for the button is set, it uses the **value** that the hook is returning, which happens to be the old one.

## Context

Component state allows us to make a component react to changes and upates. A parent component can pass state to children as props, including current values and setters:

```jsx
const GrandParent = () => {
  const [count, setCount] = useState(0);
  return <Parent count={count} setCount={setCount}/>
}

const Parent = ({ count, setCount }) => {
  return <div>
    <div>Parent Count: {count}</div>
    <Button count={count} setCount={setCount}/>
  </div>
}

const Button = ({ count, setCount }) => {
  return <button onClick={() => setCount(count + 1)}>Current {count}</button>;
}
```

As an application grows, so does the number of components and the amount of state that we need to manage. Passing state down to every child component is not scalable and is known as "prop drilling". There's no industry standard as to how many levels are acceptable but personally 2 or 3 is where it starts getting hard to manage. To avoid prop drilling, we have to introduce a new concept called **Context**. Context allows sharing state with an entire component sub-tree without explicitly passing it down through `props`. Lets see how it works:

```jsx
// First we'll create our context
export const AppContext = React.createContext({});

const GrandParent = () => {
  const [count, setCount] = useState(0);
  
  // Then we wrap all children components in a Provider,
  // which will "provide" access to a value to all descendants
  return <AppContext.Provider value={{
    count,
    setCount
  }}>
    <Parent />
  </AppContext.Provider>
}

const Parent = () => {
  const { count } = useContext(AppContext);

  return <div>
    <div>Parent Count: {count}</div>
    <Button />
  </div>
}

const Button = () => {
  const { count, setCount } = useContext(AppContext);
  return <button onClick={() => setCount(count + 1)}>Current {count}</button>;
}
```

Lets abstract this a bit more by creating a little Blog app. First we'll create our provider in a separate file:

`BlogProvider.js`
```jsx
const BlogProvider = ({ children }) => {
  const [posts, setPosts] = useState([]);
  const [selectedPostId, setSelectedPostId] = useState(null);

  return (
    <BlogContext.Provider
      value={{
        selectedPostId,
        setSelectedPostId,
        posts,
        createPost: ({ title, body }) => {
          const id = posts.length + 1;
          setPosts([...posts, { id, title, body }]);
        },
        retrievePost: ({ postId }) => posts.find(post => `${post.id}` === `${postId}`),
      }}
    >
      {children}
    </BlogContext.Provider>
  );
};
```

Now create a component that we can use to create new posts:

`NewPost.js`
```jsx
const NewPost = () => {
  const { createPost } = useContext(BlogContext);
  const [title, setTitle] = useState('');
  const [body, setBody] = useState('');

  return (
    <PostWrapper>
      <Title>
        <Input placeholder="Title..." value={title} onChange={e => setTitle(e.target.value)} />
      </Title>
      <TextArea value={body} onChange={e => setBody(e.target.value)} />
      <Button
        onClick={() => {
          createPost({ title, body });
          setTitle('');
          setBody('');
        }}
      >
        Submit
      </Button>
    </PostWrapper>
  );
};
```

Then we will create another component to render an existing Post. 

`Post.js`
```jsx
const InnerPost = ({ post }) => {
  const { updatePost, setSelectedPostId } = useContext(BlogContext);

  const [values, setValues] = useState({
    title: post.title,
    body: post.body,
  });

  // Most applications have an read/edit mode but in our example
  // you can always edit the post
  useEffect(() => {
    updatePost({
      ...post,
      ...values,
    });
  }, [values]);

  return (
    <PostWrapper>
      <FakeLink onClick={() => setSelectedPostId(null)}>All Posts</FakeLink>
      <Title>
        <Input
          value={post.title}
          onChange={e => {
            setValues(v => ({ ...v, title: e.target.value }));
          }}
        />
      </Title>
      <Body>
        <TextArea
          value={post.body}
          onChange={e => {
            setValues(v => ({ ...v, body: e.target.value }));
          }}
        />
      </Body>
    </PostWrapper>
  );
};

const Post = ({ postId }) => {
  const { retrievePost } = useContext(BlogContext);
  const post = retrievePost({ postId });

  // Note that we cannot do this in a single component because we 
  // would violate the non-conditional rendering rule of hooks. This
  // is one of the main criticisms of hooks
  if (!post) return <div>Not Found</div>;

  return <InnerPost post={post} />;
};
```

Lastly, we'll glue all of this together:

`Blog.js`
```jsx
const Blog = () => {
  const { selectedPostId, setSelectedPostId, posts } = useContext(BlogContext);
  if (selectedPostId) return <Post postId={selectedPostId} editable />;

  return (
    <div style={{ display: 'flex' }}>
      <div>
        {posts.map(post => (
          <PostWrapper key={post.id} onClick={() => setSelectedPostId(post.id)}>
            <Title>{post.title}</Title>
          </PostWrapper>
        ))}
      </div>
      <NewPost />
    </div>
  );
};

// Again, note here that we broke this up into two components because <Blog/>
// needs to use useContext. 
const Wrapped = () => {
  return (
    <BlogProvider>
      <BlogStyle />
      <Blog />
    </BlogProvider>
  );
};

export default Wrapped;
```

Lastly, we'll create a custom hooks so that the content of our blog is persisted at least locally.

```jsx
const usePersistedState = (storageKey, fallbackValue) => {
  const [value, setValue] = useState(() => {
    const storedValue = window.localStorage.getItem(storageKey);

    if (storedValue === null || !storedValue) {
      return fallbackValue;
    }

    try {
      return JSON.parse(storedValue);
    } catch (e) {
      console.log('Error parsing stored value', e);
      return null;
    }
  });

  useEffect(() => {
    if (value) {
      window.localStorage.setItem(storageKey, JSON.stringify(value));
    } else {
      window.localStorage.removeItem(storageKey);
    }
  }, [value]);

  return [
    value,
    setValue,
    () => {
      setValue(fallbackValue);
    },
  ];
};
```