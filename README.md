# React-Intermediate-Reference
React Intermediate Reference

If you are just starting out with React, start by familiarising yourself with the React documentation, and check out these other readmes first:

- [React Basic Reference](https://github.com/edgarfinn/React-Basic-Reference)

- [Redux Basic Reference](https://github.com/edgarfinn/Redux-Basic-Reference)

This Readme will explain the basic principals of how to:

- Route different content to different endpoint requests within a URL.

- Post information to an endpoint.

- Load data from a back-end API based on users current API.

- Validate user input in react forms.

React Router
---
React router essentially handles requests to different endpoints. One of these ways of handling requests might be to serve a specific component (ie a contact form, or a detailed view of a specfic blog post). This seems similar(ish) to conditional rendering, but far better equipped for breaking down the different sections to a user's journey.

For example:

- /newpost

    might return a form for writing a post to a blog.

- /posts

    might return all recent posts

- /posts/123

    might return a specific post


## Installation:
```npm i react-router-dom```

Then in the parent-most component / module that you want to handle routes:

#### src/App.js:
```js
// based on create-react-app package
import React, { Component } from 'react';
import './App.css';

// react-router set-up
import {BrowserRouter, Route} from 'react-router-dom';

class Hello extends React.Component {
  render() {
    return (<h3>Hello!!</h3>)
  }
}

class Goodbye extends React.Component {
  render() {
    return (<h3>Goodbye!!</h3>)
  }
}

class App extends Component {
  render() {
    return (
      <div className="App">
        <h1 className="App-title">Welcome to React</h1>
        <BrowserRouter>
          <div>
            // return Hello component for requests to /hello
            <Route path="/hello" component={Hello} />
            // return Goodbye component for requests to /goodbye
            <Route path="/goodbye" component={Goodbye} />
          </div>
        </BrowserRouter>
      </div>
    );
  }
}

export default App;

```

localhost:3000/
-> Welcome to React

localhost:3000/hello
-> Welcome to React
-> Hello!!

localhost:3000/goodbye
-> Welcome to React
-> Goodbye!!

All changes in URL within a react app are first passed into the History library, which forwards the new URL to React Router. The ```{BrowserRouter}``` method from react-router-dom interacts with the History library and determines what to do based on any changes to the URL.

The ```{Route}``` object is a React component that can be rendered inside any other react component. Route is used to configure different components to be rendered based on the URL entered in the browser.

The Route component takes at least two props: ```path```, which determines which URL should trigger the rendering of that component, and ```component```, which determines the component to be rendered based on the path.

```js
// for requests to /home, serve up the HomePage component
<Route path="/home" component={HomePage} />
```

### Handling varying endpoints

Using a colon within a path introduces a variable wild-card. The wild-card references a dynamic endpoint, allowing the value to vary depending on which specific instance of an endpoint you request (for example, a specific blog post, identified by its ```:id```).


```js
<Route path="/posts/:id" component={PostsShow} />
```

#### LODASH TIP:

Converting arrays of objects to Objects with lodash's ```_.mapKeys```

When ```get``` requesting arrays of data from APIs, lodash's ```_.mapKeys``` method can be useful for converting the array into an object of referenceable objects.

For example:

```js
// Turning this array:
[{id:1,value: "One"},{id:2,value: "Two"},{id:3,value: "Three"}]

// Into this object, each child object referenceable by an internal key of your choice:
{
  '1': {id:1,value: "One"},
  '2': {id:2,value: "Two"},
  '3': {id:3,value: "Three"}
}

```

The ```_.mapKeys``` method takes an array of uniform arguments as the first argument, and a specified key as the second argument. The returned value is an object of the same uniform objects from the array, each referenced using the key passed as the second argument.

This can be really useful when handling arrays of data objects from an API ```get``` request.

NOTE: The id key will need to be unique, otherwise former duplicates will be overwritten and deleted.

```js
const posts = [
  {id: 4, title: "Hi"},
  {id: 36, title: "Hows it going?"},
  {id: 25, title: "Bye"}
]

const state = _.mapKeys(posts, 'id')

console.log(state)
/*
{
  4: { id: 4, title: 'Hi' },
  25: { id: 25, title: 'Bye' },
  36: { id: 36, title: 'Hows it going?' }
}
*/
```
