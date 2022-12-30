### Wasted rerenders

Wasted renders are one of the most common React JS problems. 

What’s causing them?

To understand it, we need to first comprehend how data is passed in React. 

There are many ways, but one of the simplest is through “props” (data passed directly to the component).

Say there’s an element such as “Item” which may represent a single item on the list. If there is a list of two items and developers want to pass a value that is computed in the list to each one of the cards, they can do it this way:

![Image](https://global-uploads.webflow.com/622fa4d65a5fab0c3465af07/627938c357a245c5d1d46aac_react-js-problems-renderers.png)

The problem here is that each time “getNewValue” is called, **each item of the list is rerendered**, which is a waste of resources as the value is always the same. 

This example here is a dummy one in order for the example not to look too complicated. 

In real life, developers could deal with complex data structures being rerendered which visibly slow down the performance, e.g. messages in a group chat – a large collection of elements that contain text, images, and quotes of other messages.

Do end-users notice “wasted renders” or maybe is it simply a flaw for developers?

The answer is: this is often a **problem for the end-user** as well. 

The end-user will notice:

-   a waste of CPU, as the processing unit needs to recalculate the same value(s), thus, resulting in faster battery drainage,
-   slow down of the application if the rerenders happen often (dozens of times per minute) and require repeatable heavy computations – often perceived by users as “app lagging”.

A tool that greatly eases the pain of manually checking every rerender is “[Why did you render](https://www.npmjs.com/package/@welldone-software/why-did-you-render)”.

How can developers use it?

By launching it in the app initialization and adding a whyDidYouRender static to the components they want to track, just like this:

```
class BigListPureComponent extends React.PureComponent {  
	static whyDidYouRender = true  
	
	// a lot of logic and handlers  
	
	render() {  
		return (  
			// a list of 1000000000000 entries  
		)  
	}  
}
```

If anything goes wrong a notification will pop up in the console:

![Image](https://global-uploads.webflow.com/622fa4d65a5fab0c3465af07/627938c372c119329b105b1f_react-js-problems-renderers-tutorial.png)

[Image source](https://miro.medium.com/max/1384/1*RrPj34bpTesLXKwoPvBrbA.png)

The fix for that is easy to implement, however, one needs to be careful not to eliminate rerenders when they really need to happen.

Developers can use “PureComponent” which performs a shallow comparison of arguments passed to a component with the previous values and then decides if the component needs to rerender. 

Quite often, however, this is not enough and developers need to manually tell the components if they need to rerender by providing custom comparison functions. 

If the components are arranged in a way that observable data is set at the very top of the screen’s hierarchy component then this practice may accidentally cause a wasted rerender which happens each time a character is added to or deleted from the input field that is located on the screen and connected to a data provider such as Redux.

PureComponent or a React’s new feature of “[React.memo](https://reactjs.org/docs/react-api.html#reactmemo)” lets developers prevent such situations.

**Key take-away:** to improve the app’s performance and avoid unnecessary rerenders use a detection tool called “[Why did you render](https://www.npmjs.com/package/@welldone-software/why-did-you-render)” and implement necessary fixes. Try “PureComponent”, a tool that determines if the component needs a rerender and either lets it pass, or stops it.