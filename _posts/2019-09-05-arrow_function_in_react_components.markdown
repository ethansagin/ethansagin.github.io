---
layout: post
title:      "Arrow Function in React Components"
date:       2019-09-05 16:27:21 +0000
permalink:  arrow_function_in_react_components
---


I just finished up a screenshare on a React Lab and I thought it would be good to clarify two points about the arrow function's scope in React.

1) When written in Javascript, the arrow function implicitly returns the result of the final read line of code, which can help DRY up our functions. Additionally, the arrow does **not** bind its own 'this' value on instantiation; rather it adopts 'this' from the context of wherever it was originally instantiated. This behavoir makes the arrow function better suited for use in React than a normal function declaration or expression because of the node structure of React components. When a function is passed down the node tree, there is a danger that the function declaration or expression will redefine 'this' in their new context of the nested component, which can alter the behavior of the application in ways that are unpredictable and difficult to debug. Because the arrow function will always take 'this' from its original context, we can pass it to any of children of the component containing the arrow function and reliably get a response using the parent as the reference point for 'this'.

2) When passing a function to an event handler in React, we can write something like: `onClick={ this.functionName }`. Notice that we did not provide parentheses at the end of the function. Were we to do this, the app would read this as an invocation of the function and call it every time the page was rendered, rather than only calling it when the element was clicked. However, there are times when we need to pass an argument into a function, which creates the same invocation issue as an empty set of parentheses. When this occurs, we can wrap our function call with an arrow function in order to make it a callback: `onClick={ () => this.functionName(arg) }`. This will assign the arrow function to the onClick event listener without invoking it.
		

