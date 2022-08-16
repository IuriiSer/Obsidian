[Docs](https://reactjs.org/docs/react-component.html#gatsby-focus-wrapper) -> React lets you define components as classes or functions. Components defined as classes currently provide more features which are described in detail on this page.

### Component on ES6
```
const React = require('react');

class Board extends React.PureComponent {
  render() {
    return (
	    <Square value={i} />
    );
  }
}

module.exports = Board
```
```
const React = require('react');

class Square extends React.Component {
  render() {
    return (
      <button className="square">
        {this.props.value}
	  </button>
    );
  }
}
```
###  Without ES6
```
const React = require('react');
const ReactDOMServer = require('react-dom/server');

const renderTemplate = (_reactEl, props, res) => {
	const reactEl = React.createElement(_reactEl, props);
	return reactEl; <- this code is a part of React SSR
};
```

```
var createReactClass = require('create-react-class');
var Greeting = createReactClass({
  render: function() {
    return <h1>Hello, {this.props.name}</h1>;
  }
});
```
#### Minus
##### 1 -> 
With `createReactClass()`, you need to define `getDefaultProps()` as a function on the passed object:

```
var Greeting = createReactClass({
  getDefaultProps: function() {
    return {
      name: 'Mary'
    };
  },

  // ...

});
```
##### 2 -> 
With `createReactClass()`, you have to provide a separate `getInitialState` method that returns the initial state:

```
var Counter = createReactClass({
  getInitialState: function() {
    return {count: this.props.initialCount};
  },
  // ...
});
```
##### 3 -> 
In React components declared as ES6 classes, methods follow the same semantics as regular ES6 classes. This means that they don’t automatically bind `this` to the instance. You’ll have to explicitly use `.bind(this)` in the constructor. With `createReactClass()`, this is not necessary because it binds all methods:
```
var SayHello = createReactClass({
  getInitialState: function() {
    return {message: 'Hello!'};
  },

  handleClick: function() {
    alert(this.state.message);
  },

  render: function() {
    return (
      <button onClick={this.handleClick}>
        Say hello
      </button>
    );
  }
});
```

**This means writing ES6 classes comes with a little more boilerplate code for event handlers, but the upside is slightly better performance in large applications.**