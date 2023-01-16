### Routing

If you recall our mini Blog app, the `Blog` component render the selected `Post` if one had been selected (state variable kept in the `BlogContext`) or the list of posts otherwise. This worked well, but has some limitations:

1. we need to keep track of the selected post
2. we have to check the application state to choose what "page" to render
3. we cannot link to a post page

One could say that all of these have solutions. First, keeping track of a selected post is not complicated with a simple state variable. Second, the conditional is just a simple `if` statement. Third, we could add an effect that encodes the application state and stores it in the URL. When starting the application again, if there's state encoded in the URL we can pre-load it into our context and pick up where we left off. 

However, even though it can be done as we have shown in the simple blog app, as the application grows it becomes tedious and error prone. A better approach is to use routing so we will leverage [React Router](https://reactrouter.com/en/main) to do so.

## React Router

The easiest way to learn how to use this is with an example so lets refactor our Blog to user routing. First, we will wrap our application in a `BrowserRouter`. This allows us to use navigation within the application, you can think of it as a "provider of navigation". Inside it, we can use `Routes` to define the different paths that we can navigate to and the React element each one should render:

```jsx
const App = () => {
  return <BrowserRouter>
    <Routes>
      <Route path="/" element={<Blog />} />
      <Route path="/create" element={<NewPost />} />
      <Route path="/post/:id" element={<Post />} />
    </Routes>
  </BrowserRouter>
}
```

Now when we navigate to `/`, we will see the `Blog` element rendered, etc. Note how the last route has `:id`, which represents a param. We can access this param in the `Post` component using the `useParams` hook as follows:

```jsx
import { useParams } from 'react-router-dom';

const Post = () => {
  const { id } = useParams();
  const { retrievePost } = useContext(BlogContext);
  const post = retrievePost({ postId: id });

  if (!post) return <div>Not Found</div>;

  return <InnerPost post={post} />;
};
```

### Linking

To navigate within our application, we should avoid using standard anchor tags (`<a href="/">Home</a>`). Using these links would cause a full page refresh which will restart our app, downloading all assets (js, css, etc) again. In most cases the client (browser) will have some sort of caching, but we cannot be certain of this. Instead, we should use the `Link` component provided by `react-router-dom` as:

```jsx
import { Link } from 'react-router-dom';

...
  <Link to="/">Home</Link>
...
```

This will navigate without refreshing the page but will still push into the browser history so Back/Forward actions will still work.

### Navigating

Sometimes in our applications we want to be able to navigate to different parts of the application after an event occurs. For example, after a form is submitted correctly, without requiring the user to click a link. To do this, we use the `useNavigate` hook as follows:

```jsx
import { useNavigate } from 'react-router-dom';

const Form = () => {

  const submit = () => {
    fetch('some api') // <-- endpoint to create a new post in some API
      .then(r => r.json())
      .then(r => {
        // After the post is created, we can navigate to it
        navigate(`/post/${r.id}`)
      })
  }

  return <div>
    <div>Some Content</div>
    <button onClick={submit}>Submit</button>
  </div>
}
```

We have covered the basics of `react-router` but barely scratched the surface as it can do a lot more. If you want to read more please check out the [documentation](https://reactrouter.com/).