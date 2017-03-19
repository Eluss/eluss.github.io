---
layout: post
title: A story of Redux
---

When you meet Redux for the first time, it often seems a bit overwhelming at first. However, if you want to work with redux [check]effectively, you have to understand how it works, and what are it's core elements. State... Actions... Reducers... Store... In today's post I'd like to introduce you to Redux in not so techy way, so that you grasp the idea of how it works.

/// introduce


Archives (State)
---

If I asked you, how the state in your application looks like, then it would be really hard to explain it right away. Some values are stored in view models, there are some repositories that also store some values... Oh! And that class right there, that knows what user is currently logged in. But what if after a question like this, we could present our application's state right away? Wouldn't it be easier for us to reason about an app ordered in such way? In this case, Redux comes with a solution for us.
Imagine, that your state is like archives, you exactly know where to look for things, and it's stored in one structure. Of course having a big structure does not indicate that it will be a mess. You can store your state in archives's drawers that will represent the state of your login screen, settings screen etc.

Letters (Actions)
---

So... now we have a neatly structured state in our archives, everything is clean, and easy to reason about. But as you might expect, there is a high possibility that our users would like to change this state in some way (I guess that's not anything new). How do we handle that? Well, we can't directly access our app's state and change what we want. That's not how you work with clean archives...  Let's imagine, that I'd like to change my address. I can't just walk into a city hall and play with their archives. I need to write an official `letter`. 

^ Title: John Doe - Address Change
^ Hello, I've just moved to another house. My current address is Art Street 10.

Don't worry, when we change state in our application, we only need crucial information. So it might look like this:

```
{
    type: 'UPDATE_ADDRESS'
    address: {
        street: Art
        number: 10
    }
}
```

This king of letter could be sent at the time when user presses a button, and fields could be populated from the form that user has just filled.


Accountants (Action creators)
---

However, sometimes we can't prepare such letters right away, as we do not have all required data yet. When I need to perform a http request, I do not know the response yet. For cases like this, we have to hire our accountants. They will perform all the work, and then send a letter on our behalf. For example,after receiving response from the server, they will fill a letter with needed information and send it for us.
```
function updateUserName() {
    return (dispatch) => {
        getUserNameFromTheServer()
        .then((userName)=>{
            dispatch({
                type: 'UPDATE_USER_NAME',
                name: userName
            })
        })
    } 
}
```

City Hall (Store)
---

After your letter is ready, you have to send it to the city hall. There are many people working there, who take care of various things. Some handle your address information, some take care of your car registration etc... After they update archives with data from your letter, they will inform you about it, so that you can for example, update your todo list and cross "Change address" out... Or update your facebook status. The same goes with reducers inside an app. If new state is created, then all places interested in this update will be notified.

City Hall worker (Reducers)
---

As I said, in the City Hall, there are many people who take care of various things. These are reducers in our application. Let's imagine, that after our letter is received, it is given to cityHallDirector who presents it to every worker in this city hall. Some of them will take a look at it, and simply ignore it, as they see that title is not familiar to them. But at some point, there will be a person who says: "Oh! I'm responsible for address changes, I have to handle this!"

```
function cityHallDirector(archives, letter) {
    return {
        carsArchives: carsArchives(archives.cars, letter),
        addressesArchives: addressesArchives(archives.adresses, letter)
        ...
    }
}
```

```
function addressesArchivesWorker(addressesArchives, letter) {
    switch letter.title {
        case 'UPDATE_ADDRESS':
            return {
                ...addressesArchives, 
                address: letter.address
            }
        case 'REMOVE_ADDRESS':
            return {
                ...addressesArchives, 
                address: null
            }
        default:
            return state
    }
}
```

After your letter is handled, new archives are returned to director, who then gives it back to the city hall. Important thing is that city hall workers are only able to handle one letter at once, so you have to be sure that they work as fast as possible. After our city hall director receives all responses from his workers, he will provide new archives to the city hall. City hall will get rid of old archives and replace them with the new ones.

So these are the core elements of the Redux architecture.
Archives - state in our application
Sending letters - dispatch actions with payload, that will cause state change.
Accountants - action creators
City Hall - Application store that contains current state and reducers that will create a new state.
City Hall workers - reducers, that know how to create a new state by using current state and action.

I hope that this will help some newcomers to understand how redux works and what are the responsibilites of it's elements.

