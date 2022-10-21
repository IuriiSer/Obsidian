### Overview
[Class Components](https://reactjs.org/docs/react-component.html) - React lets you define components as classes or functions. Components defined as classes currently provide more features which are described in detail on this page. To define a React component class, you need to extend `React.Component`:
```
class Welcome extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}
```

### The Component Lifecycle

#### Mounting

These methods are called in the following order when an instance of a component is being created and inserted into the DOM:

-   [**`constructor()`**](https://reactjs.org/docs/react-component.html#constructor)
-   [`static getDerivedStateFromProps()`](https://reactjs.org/docs/react-component.html#static-getderivedstatefromprops)
-   [**`render()`**](https://reactjs.org/docs/react-component.html#render)
-   [**`componentDidMount()`**](https://reactjs.org/docs/react-component.html#componentdidmount)

#### Updating

An update can be caused by changes to props or state. These methods are called in the following order when a component is being re-rendered:

-   [`static getDerivedStateFromProps()`](https://reactjs.org/docs/react-component.html#static-getderivedstatefromprops)
-   [`shouldComponentUpdate()`](https://reactjs.org/docs/react-component.html#shouldcomponentupdate)
-   [**`render()`**](https://reactjs.org/docs/react-component.html#render)
-   [`getSnapshotBeforeUpdate()`](https://reactjs.org/docs/react-component.html#getsnapshotbeforeupdate)
-   [**`componentDidUpdate()`**](https://reactjs.org/docs/react-component.html#componentdidupdate)

#### Unmounting

This method is called when a component is being removed from the DOM:

-   [**`componentWillUnmount()`**](https://reactjs.org/docs/react-component.html#componentwillunmount)

#### [](https://reactjs.org/docs/react-component.html#error-handling)Error Handling

These methods are called when there is an error during rendering, in a lifecycle method, or in the constructor of any child component.

-   [`static getDerivedStateFromError()`](https://reactjs.org/docs/react-component.html#static-getderivedstatefromerror)
-   [`componentDidCatch()`](https://reactjs.org/docs/react-component.html#componentdidcatch)

### [](https://reactjs.org/docs/react-component.html#other-apis)