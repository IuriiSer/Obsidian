## React JS problems with performance

React itself boasts about its high performance, which is true, yet, there are cases where careful actions need to be taken.

### The concept of virtual DOM

Before we start discussing how to deal with React performance problems we need to first understand how React UI is rendered. 

The key concepts here are DOM and virtual DOM. 

“**DOM**” stands for Document Object Model which is the structure of a webpage composed of a tree of objects. In layman’s terms, it means the HTML elements of the webpage. 

“**Virtual DOM**” is the representation of the DOM kept in the memory and synchronized with the real DOM when it needs to be updated. Thanks to this approach developers are able to abstract out the attribute manipulation, event handling, and manual DOM updating.

![[ReactProblemsPerfomans.jpg]]

The image shows that the virtual DOM (kept in memory) computes the difference when a state change occurs, and only then it is applied to the real DOM.