---
layout: post
title:      "Targeting State in Your Reducer"
date:       2019-10-29 17:08:09 +0000
permalink:  targeting_state_in_your_reducer
---


Hey there! I've officially declared my job search to Flatiron's career services team, so in the coming weeks I'll be writing regularly as I navigate the path to getting my first position as a software engineer. The posts will be a mix of reflections on various aspect of the search process as well as discussions of new information I stumble across and problems I encounter when building projects. For my first post however, I want to look back at the final app I built for the Flatiron curriculum a pick apart a small-but-important bug I encountered.

SETTING THE STAGE:
My app is a social calendar where you can assign each friend a time interval (in months) in which you'd like to regularly get together with them. If you do not schedule a future meetup within that span of months, the app will alert you that you are overdue to hang out with your friend and calculate the number of days by which you are overdue based on the date of the last scheduled meetup. The app is built using a React front-end and a Ruby-on-Rails API.

PROBLEM:
I had an index page which listed all of the user's friends as links to individual profile pages, where the user could view the friend's personal info and manage their list of meetups. On the profile pages, I wanted to add a button which allowed the user to edit the friend's personal information. My app manages a store for global state using Redux, so I set up my action/reducer to target the friend object by its id and update its attributes using local state connected to my edit form.

The result was a little odd: my api was receiving and updating the correct friend object in the database, but my component did not register the change to the object when it re-rendered during the componentDidUpdate cycle. It was only once I returned to the index page that my app would register the change and begin using my friend's updated information. This obviously was not ideal.

SOLUTION:
As you might have guessed, the root of the issue was in the reducer function I had written. My initial code was along the following lines:

        case 'FRIEND_UPDATED':
            return {
                friends: [...state.friends, action.payload],
                loading: false
            }

My assumption had been that this would work along the same lines as my FRIEND_ADDED case, which is identical. The key difference is that when the payload in FRIEND_ADDED is added to the array value, Redux recognizes the change as an addition of a new object and updates the store accordingly.

The problem with trying to incorporate this logic into FRIEND_UPDATED was that since the object already existed, Redux did not register a change was being made to the object itself. The elements being updated were key-value pairs WITHIN that object. What the above code was essentially asking Redux to do was somehow identify the object that matched the payload I had given it with no instructions on how to go about that process, and THEN target those internal differences and change them. In short, this is above Redux's pay grade.

A potential fix would have been to find a way to target the payload's corresponding object within the array, open a subarray and use the spread operator so that I could drop in the payload's changed key-value pairs directly. This would have gotten convoluted pretty quickly; luckily there was a much easier solution:
						
						case 'FRIEND_UPDATED':
                let newFriends = state.friends.filter(f => f.id !== action.payload.id).concat(action.payload)
                return {
                    ...state, 
                    friends: newFriends,
                    loading: false
                }

Because the friend object were listed in an array, the most straightforward fix was to pass the friends key an entirely new array object. I created this by taking the original array and filtering out the target whose id matched the id of the action payload, and immediately concating the payload into the array as its replacement.

One final loose string to tie up: why was the index able to register the change when the individual page could not? Well, in short, when we returned to the index page, the component had to be remounted, which triggered a call to the API for a fresh set of data to populate the global store. Since our code was able to successfully update the API from the start, the component received our updated friend's information and subsequently passed it down to the friend component to use as a prop the next time we opened our friend's profile page.
