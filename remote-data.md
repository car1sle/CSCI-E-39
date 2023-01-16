## Remote Data - Continued

We have learned how to build applications that require remote data. We used `fetch` to query external APIs and created abstractions as reusable custom hooks. Learning how to build our own libraries is important to understand how things are wired. This allows us to debug issues and create our own libraries or modules as needed. In this lecture, we'll learn about new types of APIs and libraries that provide these abstraction layers out of the box, with additional functionality that comes in handy when working with remote data.

### GraphQL

In the previous lecture, we discussed REST APIs and what they're used for. In simple terms, a REST API exposes different endpoints that behave (usually) a lot like pure functions - given a request with some parameters, it returns the same response. For example:

Request:
```
https://api.chucknorris.io/jokes/random?category=dev
```

Response:
```JSON
{
  "categories": ["dev"],
  "created_at": "2020-01-05 13:42:19.324003",
  "icon_url": "https://assets.chucknorris.host/img/avatar/chuck-norris.png",
  "id": "a1whrz_crhgykfuah1mrmw",
  "updated_at": "2020-01-05 13:42:19.324003",
  "url": "https://api.chucknorris.io/jokes/a1whrz_crhgykfuah1mrmw",
  "value": "Chuck Norris doesn't need sudo, he just types \"Chuck Norris\" before his commands."
}
```

We can make this request several times and even though the joke will be different (be careful, some might be explicit!), the structure will always be the same.

From a client perspective, this creates a great environment to develop against. Having a predictable response allows us to build user interfaces with confidence on the data we're requesting. As long as the API is not changed, we can expect the same result every time. 

However, this has some disadvantages. Imagine you wanted to build app to display song lyrics. The main screen should show a list of 100 songs and the user can select a song to display its lyrics. It woulnd't be wise to fetch the lyrics for all 100 songs at once, since the user might not even need them all. In a REST API, we would probably use two endpoints:

Request songs:
```
https://songs.api/list
```

Response:
```JSON
[
  {
    "id": "433280fc-89c5-4709-87c1-59a00d20a8bc",
    "title": "Imagine"
  },
  {
    "id": "433280fc-89c5-4709-87c1-59a00d20a8bc",
    "title": "Let it be"
  },
...
}]
```

Request lyrics:
```
https://songs.api/song/433280fc-89c5-4709-87c1-59a00d20a8bc
```

Response:
```JSON
{
  "id": "433280fc-89c5-4709-87c1-59a00d20a8bc",
  "title": "Imagine",
  "lyrics": "Imagine there's no heaven...",
  "author": {
    "id": "d2ab822b-d12a-4749-8508-e48ad6b4fa9c",
    "name": "John Lennon"
  }
}
```

Further, if we wanted songs by a specific author, we would need yet another endpoint:

Request author:
```
https://songs.api/author/d2ab822b-d12a-4749-8508-e48ad6b4fa9c
```

Response:
```JSON
{
  "id": "d2ab822b-d12a-4749-8508-e48ad6b4fa9c",
  "name": "John Lennon",
  "songs": [{
    "id": "433280fc-89c5-4709-87c1-59a00d20a8bc",
    "title": "Imagine"
  }]
}
```

As clients need more and more data grouped and arranged differently, the number of endpoints grows exponentially. This is one of the main reasons GraphQL was introduced. It was mainly built by developers frustrated with the limitations or REST. From the client perspective, it allows to query exactly the data needed and nothing more. From the server perspective, it allows exposing types rather than endpoints. It is up to the client to request the needed data from these types.

We're going to create the same application but using GraphQL. 

## Server-Side

On the server-side, we first define our types:

```
type Song {
  id: ID!
  name: String!
  lyrics: String!
  author: Author
}

type Author {
  id: ID!
  name: String!
  songs: [Song]
}
```

This defines the available models but not necessarily the "endpoints" to access them or queries that can be executed against them. For this, every GraphQL service has a `query` type, under which we define what can be queried:

```
type Query {
  songs: [Song]
  authors: [Author]
}
```

That's it - we have now defined how a client can query our models.

## Client-side

The client-side is where GraphQL becomes most useful. Continuing with our song application, lets see how we can get a list of songs:

Request GraphQL query:
```
query {
  songs {
    id
    name
  }
}
```

Response:
```json
{
  "data": {
    "songs": [
      {
        "id": "ckwi8wua0btfg0a30j601uf7p",
        "title": "Imagine"
      },
      {
        "id": "ckwi8y0psbyzr0a76goa5iam8",
        "title": "Let it Be"
      },
      ...
    ]
  }
}
```

Now if we want to request a specific song:

Request GraphQL query:
```
query {
  songs(id: "433280fc-89c5-4709-87c1-59a00d20a8bc") {
    id
    name
    lyrics
    author {
      id
      name
    }
  }
}
```

Response:
```json
{
  "data": {
    "songs": [
      {
        "id": "ckwi8wua0btfg0a30j601uf7p",
        "title": "Imagine",
        "lyrics": "Imagine there's no heaven..."
      },
    ]
  }
}
```

As you can see, we haven't created a different endpoint. We simply added an `id` parameter and requested additional data (`lyrics` and `author`). All it takes is adjusting the query to what we need.

## React and GraphQL

Making a GraphQL request is as simple as making a POST request, passing our `query` as the `body`. We could use the same techniques we covered (using `fetch`) but instead we'll learn about a new module called Apollo (apollographql.com/docs/react/).

First, we'll wrap our App in a Provider, exported from Apollo:

```jsx
const AppWrapper = () => {
  const apolloClient = new ApolloClient({
    uri: Config.GRAPHCMS, // this is the endpoint of our Graph
    cache: new InMemoryCache(),
  });

  return (
    <ApolloProvider client={apolloClient}>
      <App />
    </ApolloProvider>
  );
};
```

That's it! We're now ready to start using our client to make queries. Lets create the list of songs:

```jsx
import { useQuery } from '@apollo/client';

// First define the query:
export const GET_SONGS = gql`
  query getSongs {
    songs {
      id
      title
    }
  }
`;

// The useQuery hook gives us a nice api, telling us when it's loading, or if there are errors
const SongList = () => {
  const { data, loading, error } = useQuery(GET_SONGS);

  if (loading) return <div>Loading...</div>;

  if (error) return <div>Failed</div>;

  return data.songs.map(p => (
    <div key={p.id}>{p.title}</div>
  ));
};
```

What if we want a specific post? We can of course provide parameters to our queries as follows.

First we need our server to accept the params, so the signature for our query could be:

```
type Query {
  songs: [Song]
  authors: [Author]
  song(id: ID!): Song
}
```

The last query here says that we can ask for a song with a specific `id`, and the `!` is telling us it is a required parameter. Note that this query does not return a list of `Songs` but a single `Song`. Now on the client we would use:

```jsx
import { useQuery } from '@apollo/client';

// First define the query including params:
export const GET_SONG_BY_ID = gql`
  query getSong($id: ID!) {
    song(id: $id) {
      id
      title
      lyrics
    }
  }
`;

const Song = ({ id }) => {
  const { data, loading, error } = useQuery(GET_SONG_BY_ID, {
    variables: {
      id
    }
  });

  if (loading) return <div>Loading...</div>;

  if (error) return <div>Failed</div>;

  return <div>
    <div>{data.song.title}</div>
    <div>{data.song.lyrics}</div>
  </div>
};
```

## Mutations

You can think of a query as a GET request in a purely RESTful API (even though all GraphQL requests are http type POST). So what about modifying data?

GraphQL allows us to mutate data by sending a mutation request with the required params in a similar way to queries:

## Server-side
```
type Mutation {
  createSong(id: ID!, title: String!): Song
}
```

A common pattern is to have a mutation that receives some parameters and returns the updated object, which is what the signature of our mutation defines.

## Client-side

```js
// First define the mutation - note how we can specify what fields we want in the returned object too
export const UPDATE_POST = gql`
  mutation updatePost($id: ID!, title: String!) {
    updatePost(id: $id, title: $title) {
      id
      title
    }
  }
`;

const SongCreator = ({ song }) => {
  const [createSong, { loading }] = useMutation(UPDATE_POST);
  const [title, setTitle] = useState(song.title);

  return <div>
    <input value={title} onChange={e => setTitle(e.target.value)}/>
    <button onClick={() => {
      createSong({
        variables: {
          id: song.id,
          title,
        }
      })
    }}>{loading ? 'Saving...' : 'Save'}
  </div>
};
```

Now lets put everything together:

```jsx
const App = () => {
  return <div>
    <SongList/>
    <SongCreator/>
  </div>
}
```

If we try to create a song, even if it succeeds on the server, the list of songs doesn't yet show the new song. This is because our cache doesn't know about server changes unless we tell it. There are three approaches to solving this:

### Refresh Queries

In most cases, when we mutate data, we know which queries need to be refetched. If this is the case, we can tweak our mutation to specify what to refetch as follows:

```jsx
const [createSong, { loading }] = useMutation(UPDATE_POST, {
  refetchQueries: [{
    query: GET_SONGS,
  }]
});
```

This way, any time the `createSong` mutation succeeds, it will fetch the list of songs again.

### Polling

Using refresh queries works when these conditions are met:

- we trigger the mutation
- we know what queries need to be refetched

However, if there were multiple client-applications mutating the data on the server, our application would never know until the next refresh. We can solve this by simply telling the Apollo Client that some queries need to be refetched periodically. All we need to do is set the `pollInterval` on the queries:

```jsx
const { data, loading, error } = useQuery(GET_SONGS, {
  pollInterval: 5000, // re-run this query every 5 seconds
});
```

This makes sure that our song list will at most be lagging by 5 seconds behind the server data. Note that this can affect performance. Its easy to think that all our queries should be refetched as frequently as possible so we're always in sync with the server, but this would overload the server with a lot of unnecessary requests. Consequently, this technique should be limited to specific use cases.

### Fetch-Policy

By default, Apollo uses the in-memory cache we specified when we created the client to store all data retrieved. Cached entries are only cleared when we explicitly refetch a query. However, when needed, we can tell the client to skip the cache altogether and always pull the data from the "network" (i.e. the server). We can do this using the `fetchPolicy` param:

```jsx
const { data, loading, error } = useQuery(GET_SONGS, {
  fetchPolicy: 'network-only', // always pull this data from the server
});
```

In this case, every time we call the `songs` query, we'll be pulling a fresh copy from the server. The advantage over polling is we're not firing requests periodically. However, we lose performance by sending network requests unnecessarily when the cached data could be correct too. 

### Subscriptions

GraphQL supports `subscriptions`, which work similarly websockets. This allows the server to "push" changes to our client, without us having to send a new request each time. We're not going to cover this in this course but you can read more about it [here](https://dgraph.io/docs/graphql/subscriptions/).

### So what technique should I use?

There's no right answer here as every use case and application is different. A chat application might require a subscription while a blog might do just fine with in-memory caching. Other apps might require hybrid approaches, with some queries needing refetching with higher frequencies than others. Use whatever works best for your use case and users.

## What about traditional REST APIs?

When Apollo came out to work with GraphQL services, it became very popular because it was very developer-friendly, had a very good API, documentation and the functionality it provided out of the box was very useful (caching, refetching, etc). So much so that a few years later an almost exact copy came out to deal with traditional REST APIs using the same interface. It's called `react-query` and it's gaining a lot of popularity as well. You can read more about it [here](https://react-query-v3.tanstack.com/)