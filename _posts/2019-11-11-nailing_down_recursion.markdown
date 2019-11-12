---
layout: post
title:      "Nailing Down Recursion"
date:       2019-11-12 00:02:40 +0000
permalink:  nailing_down_recursion
---


I've been playing around with some educational materials on data structures for several weeks now, and while the concept of recursion is not difficult to explain on the surface, understanding how to implement it has proven more challenging. I came across a decent explanation over the weekend that I'm going to try to break down below.

**Recursion - What Is It?**
Recursion is the process by which the solution to a problem is found by conducting the same operation over and over again on a progressively diminishing object. In programming, a recursive function actually calls itself as a method within its body, which essentially creates a loops that only ends when the argument being transformed and passed to each new method call meets a conditional requirement. For example:

  `function recursiveFn(arg) {
      if(arg < 1) return arg
	    return arg + recursiveFn(--arg)
   }`
	 
In the sample function above, recursiveFn will continue to call itself on an argument that is decrementing on each progress method assignment. Once the argument evalutes to less than one, it will break the loop and reduce the last line of each method into a final returned value.

If we want to write an algorithm that requires a recursive solution then, we know that the function will require 1) a base case to break the loop, 2) a method that interacts with some other variable and is returned in the event that the base case doesn't evaluate to true, and 3) a transformation to the argument to pass to the method call. 


**JS Compiling**
To get a clearer picture of how this recursive process plays out, let's give our sample function an argument of 4 and spread it out line-by-line:

```
function recursiveFn(4) {
      if(4 < 1) return 4
	    return 4 + recursiveFn(--4) {
			      if(3 < 1) return 3
	          return 3 + recursiveFn(--3) {
			           if(2 < 1) return 2
	               return 2 + recursiveFn(--2) {
			                if(1 < 1) return 1
	                    return 1 + recursiveFn(--1) {
			                      if(0 < 1) return 0			
			                }
			           }
			     }
			}
 }
```

For the purposes of legibility, we're going to begin eliminating some lines. We can remove the test for base case in the first four iterations of the function because we know that they all evaluate to false, leaving us with:

```
function recursiveFn(4) {
	    return 4 + recursiveFn(--4) {
	          return 3 + recursiveFn(--3) {
	               return 2 + recursiveFn(--2) {
	                    return 1 + recursiveFn(--1) {
			                      if(0 < 1) return 0			
			                }
			           }
			     }
			}
 }
```

Now, the important thing to note as we step through this process is that the when JS compiles a recursive function, it can't return the result of the topmost function until it has evaluated the methods nested within the tree. This is the part that tripped me up because the order on which the return values are operated is a bit counter-intuitive. If I were writing this algorithm from scratch, my first instinct would be to conceptualize the final result like this:

`function recursiveFn(4){
     4 + 3 + 2 + 1 + 0
}`

This seems to make sense when I'm reading the function laid out line-by-line because our brains are used to reading from left to right, top to bottom of the page. However, if we ran this function and stepped through the call stack, we would immediately see that this order isn't really correct! Because JS has to evaluate method within those return lines before it returns a value, we get something that's halfway in between:

	    return 4 + recursiveFn(--4) {
	          return 3 + recursiveFn(--3) {
	               return 2 + recursiveFn(--2) {
	                    return 1 + recursiveFn(--1) {
			                      return 0			
														
becomes...

			return 4 + recursiveFn(--4) {
	          return 3 + recursiveFn(--3) {
	               return 2 + recursiveFn(--2) {
	                    return 1 + 0

becomes...

	    return 4 + recursiveFn(--4) {
	          return 3 + recursiveFn(--3) {
	               return 2 + 1
								 
becomes...
								 
	    return 4 + recursiveFn(--4) {
	          return 3 + 3

becomes...
								 
	    return 4 + 6
			
			
As you can see, the most deeply nested method is actually the FIRST to be fully evaluated within the call stack, meaning that we actually step backwards through our order of operations. This doesn't have much effect on our sample function, so let's look at one more example which better illustrates the issue:

```
let array = [ ]

function createReverseArray(num){
     if(num < 1) return array.push(num)
		 return array.push(createReverseArray(--num))
}

console.log(array)
```

In this example, we pass in a number to the function whose value is subsequently decremented and pushed onto the end of an array (Note: array must be outside the function body, otherwise it's value would be reassigned to [ ] with each new method call). Reading this function from left to right, we might assume our order of operation to be as follows for an argument of 4:

```
array.push(3), array.push(2), array.push(1), array.push(0)

array = [3, 2, 1, 0]
```

Remember though, our compiler isn't going to evaluate return if there is an unresolved method within the line. Therefore, our call stack is going to push the deepest level to the array first, giving us:

`array = [0, 1, 2, 3]`


**Wrapping Up**
As coders, we have to regularly remind ourselves that the rules of reading are much less linear than in non-programming languages. Don't worry if you're still fuzzy on the mechanics of a recursive function, it can take some time to wrap your brain around. The important concept that I hope you'll take away from this post is that when you have any sort of recursive loop in your code, the first line to be resolved is the most deeply nested one.
When developing an algorithms that utilizes recursion, I'd recommend starting from your base case and stepping backwards up your code tree to make sure you're getting the results in the order that you want; you'll be able to catch and fix any issues your code may have regarding the order of your results, and you'll avoid the trap of confusing yourself with the more traditional reading order of English.
