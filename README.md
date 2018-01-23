# React-Intermediate-Reference
React Intermediate Reference

If you are just starting out with React, start by familiarising yourself with the React documentation, and check out these other readmes first:

- [React Basic Reference](https://github.com/edgarfinn/React-Basic-Reference)

- [Redux Basic Reference](https://github.com/edgarfinn/Redux-Basic-Reference)

This Readme will explain the basic principals of how to:

- Use [React Router](#react-router) to route different content to different endpoint requests within a URL.

- Post information to a specific endpoint.

- Load data from a back-end API based on users current API.

- Handle, validate and update redux state with a user's form input using [Redux Form](#redux-form) .

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

Avoid loading multiple components for similar paths.
---
Router will load **all** components whos paths match the first part of an endpoint request. So if two components are routed to path endpoints that partly match one another (ie **/users** and **/users**/edgarfinn ), then components matched to **both** of these endpoints will be mounted and rendered.

This can be avoided using react-router-dom's ```Switch``` component.

```js
import { BroswerRouter, Route, Switch } from 'react-router-dom';

<BrowserRouter>
  <div>
    <Switch>
      <Route path="/users/:id" component={UserProfile} />
      <Route path="/users" component={UserList} />
      <Route path="/" component={HomePage} />
    </Switch>
  </div>
</BrowserRouter>

```
Now, the ```UserList``` or ```HomePage``` components will not be rendered for requests to "/users/edgarfinn".

**NOTE:** The Switch component will render ONLY the first Route that matches the current URL being requested, THEREFORE you must list your Routes in order from most specific path, to least specific path (as above).

Navigation
---
React-router-dom's ```Link``` component acts similarly to an anchor tag.

Syntax:

```js
import { Link } from 'react-router-dom';

<Link to="/posts/new">
  Add New Post
</Link>

//will render to the DOM as:

<a href="/posts/new">Add New Post</a>

```

A Link component essentially renders as an anchor tag, but with a couple of pre-written event handlers to prevent the browser from performing certain default actions such as sending further requests to the server.

LODASH TIP:
---

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

Redux Form
---
Redux form saves you having to set up event handlers, action creators and input validation.  It is automatically connected to redux-form and passes input values straight to your formReducer reducer. You should always allow **one** Field component for **each** piece of state to be updated. Redux form handles changes in form fields, taking care of event listeners, state changes and input validation.

```$ npm install redux-form --save```

### Setup:

Before implementation, you should first make sure the redux-form reducer function is mapped in your root reducer.

The default name of this function is ```reducer```, so it helps to import it and assign it to a less generic reference such as ```formReducer```, to avoid erroneous references.

src/reducers/index.js
```js
//import reducer, assigning it to reference formReducer
import { reducer as formReducer } from 'redux-form'

const rootReducer = combineReducers({
  // formReducer MUST be mapped to a key named 'form'
  form: formReducer
})

```
### Implementation:

On your form component (in this case src/components/posts_new.js):

```js
class PostsNew extends Component {

  // the 'field' argument allows the Field component to pass in a few event handlers
  // that allows the Field component below to map its state and reducer interactions
  // with this exact input field.
  renderTitleField(field) {
    return (
      <div>
        <input
          // remember to declare the input type!
          type="text"
          {...field.input}
         />
      </div>
    )
  }

  render () {
    return (
      <form>
        <Field
          // describes which bit of state is being edited
          name="title"
          // specify a component function to determine this Field's appearance
          component={this.renderTitleField}
        />
      </form>
    )
  }
}

// notice how implementation here is similar to {connect}
export default reduxForm({
  // always make sure the value of form here is unique,
  // to prevent conflict with any other forms on your application.
  form: 'PostsNewForm'
})(PostsNew);
```

The ```Field``` component is essentially a react component, which is used to represent a distinct input that will be visible on-screen to your users. It has no concept of appearance, and therefore no JSX or presentational value until you give it some. The Field component is declared, specifying a ```name``` property, which identifies the piece of state the field will produce, and a ```component``` property, which should be used to reference the JSX that will determine its appearance. For example the ```title``` name property above, maps this field's input data to the ```title``` state property. And the ```renderTitleField``` function is passed in to the Field component's ```component``` property determine the appearance of the Field, by identifying a function that will be responsible for the ```Field```'s presentational value.


The presentational component (```renderTitleField```) is passed a ```field``` argument, which allows redux-form to map user interactions between the presentational component (```renderTitleField```) and the ```Field``` component, and therefore pass user input data on to redux's state. So it is vital this argument is declared when declaring the Field's presentational component.

This ```field``` argument also allows access to any useful props values you may pass to it from the ```Field``` component. These are passed in like normal props: ```<Field label="Title" ... />```, are referenced using ```field.example``` (confusingly, **NOT ```field.props.example```** as one might assume)


Once passed in, you must then declare the ```{...field.input}``` object. This object contains a number of different event handlers and props, such as ```onChange```, ```onFocus``` and ```onBlur```, which can all be abbreviated into this one (destructured?) declaration.

The ```reduxForm``` invocation at the bottom is a function very similar to {connect} from react-redux. It allows your components to communicate with the ```formReducer``` reducer that we just mapped to our ```rootReducer``` above.

### Modularising multiple Fields in one form.

The above example works great, but is not optimised for scaling up and elaborating easily if you want to render more then one input field.

If you wish to use the same presentational function in order to render several inputs for several Field components, you can modularise the presentational component by making use of the fields props like so:

```js
// generic renderField presentational component
renderField(field) {
  return (
    <div className="form-group">
    // access fields label prop to dynamically access the label property
      <label>{field.label}</label>
      <input
        className="form-control"
        type="text"
        {...field.input}
       />
    </div>
  )
}

// now this renderField function can be used to render multiple inputs for multiple fields,
// so long as each declares a label property like so:

render () {
  return (
    <form>
      <Field
        // sets a label property for accessing in the presentational component
        label="Title"
        // describes which bit of state is being edited
        name="title"
        component={this.renderField}
      />

      <Field
        // sets a label property for accessing in the presentational component
        label="Categories"
        // describes which bit of state is being edited
        name="categories"
        component={this.renderField}
      />

      <Field
        // sets a label property for accessing in the presentational component
        label="Post Content"
        // describes which bit of state is being edited
        name="content"
        component={this.renderField}
      />
    </form>
  )
}
```

Submitting form data
---

Redux form only handles form data by updating state and validating it with its own event handlers. What you do with the data once it has been passed into state and validated is largely up to you. Therefore, you need to write your own onSubmit handler, and declare exactly what you want it to do with your user's input.

HOWEVER - Redux form does run a check on the validity of your user's input before passing it over to your submit handler, but it requires some configuration in order to do so:

- First you need to declare an onSubmit handler helper function. When configured correctly, redux form will automatically pass this handler a ```values``` object, containing the values of your user's input.

```js
onSubmit(values) {
  console.log(values)
}
//{title: "user's title data", categories: "user's cat data", content: "user's content data"}
```

- Then at the start of your render method, access redux-form's handleSubmit method from reduxForm's props, and assign it to a reference:

```js
render () {
  const { handleSubmit } = this.props;
  // same as: const handleSubmit = this.props.handleSubmit;
  // ...
  // more code
  // ...
}
```

```handleSubmit``` is a pre-written handler provided by reduxForm, which is set to handle submissions for you by combining things like validation automatically.

All this function does is handles the state and validation of the input, after data is passed to redux state and validated, you need to tell ```handleSubmit```  what to do next, by passing your ```onSubmit``` function in as a callback argument...so...

- On your form element, pass in your onSubmit handler **via an invocation of the handleSubmit function** like so:

```js
<form onSubmit={handleSubmit(this.onSubmit.bind(this))}>
```

Make sure you ```bind(this)``` to onSubmit to this when passing in as a callback, so that it still has access to your component.

Form validation with redux-form
---

Firstly, in order to setup form validation with redux form, create a ```validate``` helper function and pass this into our reduxForm invocation at the bottom of the document:

```js

const validate = (values) =>  {
  // values.title accesses the user input data in title input.
  // values.categories accesses th euser input data in the categories input and so-on ...
  const errors = {};
  // If errors object is empty, the form is fine to submit.
  // If errors has any properties, redux assumes the form is invalid.
  return errors;
}

export default reduxForm({
  // always make sure the value of form here is unique, to prevent conflict with any other forms on your application.
  validate,
  form: 'PostsNewForm'
})(PostsNew);

```

The validate function should declare an (initially empty) 'errors' object, and return this object.

When ```validate``` passes this object into ```reduxForm```, the object is evaluated. If it is still an empty object, then the form is assumed to be valid, and is therefore submitted. If - however - the ```errors``` object contains any properties, it will block the submission of the form and throw errors instead.

```js

const validate = (values) =>  {
  const errors = {};

  // if user did not enter a title value:
  if (!values.title) {
    // then add this error message to the errors object
    errors.title = "Enter a title!";
  }
  // if user enters a title shorter than 3 characters:
  if (values.title.length < 3) {
    // then add this error message
    errors.title = "Title must be at least 3 characters";
  }
  if (!values.categories) {
    errors.categories = "Enter some categories!";
  }
  if (!values.content) {
    errors.content = "Enter some content!";
  }
  // If errors object is empty, the form is fine to submit.
// If errors has any properties, redux assumes the form is invalid.
return errors;
}



```

### Displaying error messages to the user:

The ```name``` property in the Field component is not only responsible for mapping the input's data to state, it is also used to map validation error messages back to the correct presentational component.

For example:
- ```errors.title = "Title must be at least 3 characters";``` will only be mapped back to the input field with the ```title``` name property.

Simlarly:
- ```errors.categories = "Enter some categories!";```  will only be mapped back to the input field with the ```categories``` name property.

So these error messages can now be referenced from the presentational component using ```{field.meta.error}```, but the relevant error property **MUST** match the string assigned to the name property of the correct Field component:

```js
renderField(field) {
  return (
    <div className="form-group">
      <label>{field.label}</label>
      <input
        className="form-control"
        type="text"
        {...field.input}
       />
       {field.meta.error}
    </div>
  )
}
```

The only problem with the above code is that all error messages ```{field.meta.error}``` will render as soon as the page loads.

Redux form breaks the Field's life-cycle into three chapters:

- Pristine

  When the field has just mounted, and not yet been interacted with.

- Touched

  After the field has been interacted with (but not necessarily submitted yet)

- Invalid

  Once the input has been validated and found to be invalid (ie after a form submission)

So, using the ```{field.meta.touched}``` property, you can alert users to invalid data input **before** they've tried to submit it. This property returns a falsy value until the user has clicked or interacted with the Field and moved focus away again (ie clicked in, and then clicked out of an input).

```js
// show error message only after the field has been touched.
{field.meta.touched ? field.meta.error : ''}
```

### Conditional styling for error warnings

You might want to colour an error message to grab the user's attention in the event of erroneous submissions or data input.

Here, you only want the error-related styling to appear if the form data is both **touched** and **has errors**, so you can render a className string conditionally, depending on the truthiness of these two factors like so:

```js
renderField(field) {
  const className = `form-group ${field.meta.touched && field.meta.error ? 'has-danger' : ''}`;
  return (
    <div className={className}>
    // ...
    </div>
  )
}
```

### Destructuring tip:
The ```renderField``` function is now making quite a few references to ```field.meta.someProperty```, which adds up to be wordy.

```js
renderField(field) {
  const className = `form-group ${field.meta.touched && field.meta.error ? 'has-danger' : ''}`;
  return (
    <div className={className}>
      <label>{field.label}</label>
      <input
        className="form-control"
        type="text"
        {...field.input}
       />
       {field.meta.touched ? field.meta.error : ''}
    </div>
  )
}

```

[ES6 destructuring assignments](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment) can be a powerful tool for cleaning up your code, they offer a smart way to abbreviate value assignments from various data types, in order to the reduce the overall word count in your code base.

This simple line...
```js
const { meta } = field
// same as:
// const meta = field.meta
```
...means that you can now abbreviate ```field.meta.touched``` to ```meta.touched```, and ```field.meta.error``` to ```meta.error```

```js
renderField(field) {
  const { meta } = field;
  const className = `form-group ${meta.touched && meta.error ? 'has-danger' : ''}`;
  return (
    <div className={className}>
      <label>{field.label}</label>
      <input
        className="form-control"
        type="text"
        {...field.input}
       />
       {meta.touched ? meta.error : ''}
    </div>
  )
}
```

This is an improvement, but there's still a lot of references to meta. To go one step further, we can further stipulate ```const { meta: {touched, error} } = field;``` in order to identify ```touched``` and ```error``` as properties that belong inside the ```field.meta``` key. So we can now abbreviate ```meta.touched``` and ```meta.error``` to simply ```touched``` and ```error```:

```js
renderField(field) {
  const { meta: {touched, error} } = field;
  const className = `form-group ${touched && error ? 'has-danger' : ''}`;
  return (
    <div className={className}>
      <label>{field.label}</label>
      <input
        className="form-control"
        type="text"
        {...field.input}
       />
       {touched ? error : ''}
    </div>
  )
}
```

Destructuring assignments can be powerful help in cleaning up your code.


Reference code in repositories:
---

- [React Redux Blog Template](https://github.com/edgarfinn/react-redux-blog-template)
