---
layout: post
title: From React to React-Redux in a few steps
category: blog
comments: true
tags: [react, redux]
header: /assets/images/from-react-to-redux/from-react-to-redux-header.jpeg
excerpt: In this post I'm going to focus on connecting `React` components with `Redux`. If you are just starting out with `React + Redux` or you have already worked with these before, but want to make this concept a bit clearer then feel invited to read this post till the end ;)
---

![image](/assets/images/from-react-to-redux/from-react-to-redux-header.jpeg)

In this post I'm going to focus on connecting `React` components with `Redux`. If you are just starting out with `React + Redux` or you have already worked with these before, but want to make this concept a bit clearer then feel invited to read this post till the end ;)

If you would like to get a bit more understanding of the flow in Redux, you can take a look at my previous [post about how I understand the Redux architecture](http://eliaszsawicki.com/story-of-redux/).

## Before we introduce Redux

Before we dive into `Redux` let's take a look at simple `React` component. What does it look like?
Just to make it a bit clearer - let's use TypeScript with interfaces to show what `props` (input data) do we expect in the component.

``` jsx
interface Props {
    title: string
    numbers: number[]
}

interface State {}

export class SimpleComponent extends React.Component<Props, State> {

  render() {
    return <div>
      <h1>{this.props.title}</h1>
      {this.props.numbers.map(number => <p>{number}</p>)}
    </div>
  }
}
```

This component takes two input parameters - `title` and `numbers`. If we want to display it in our application, we need to pass these manually. For example:

``` jsx
<SimpleComponent title='Test' numbers={[1,2,3,4]}/>
```

## Introducing Redux

I guess that in every developer's life there comes a time when one wants to make something more complex for no reason, so let's introduce `Redux` to our example. (Disclaimer: it was a joke).

Do we really need `Redux`? Let's take a look at an example of an app without `Redux` first.

``` jsx
interface Props {}
interface State {}
export class FatComponent extends React.Component<Props, State> {
    render() {
        return <div>
            <SimpleComponent title='Test' numbers={[1,2,3,4]}/>
            <SimpleComponent title='Test' numbers={[1,2,3,4]}/>
            <SimpleComponent title='Test' numbers={[1,2,3,4]}/>
            <SimpleComponent title='Test' numbers={[1,2,3,4]}/>
        </div>
    }
}

```

The `FatComponent` displays 4 `SimpleComponents` with the same numbers.
Let's imagine that we would like to share the same numbers across all of our `SimpleComponents`.
A good way to do it would be to move this data to one place (a parent).
In this case our `FatComponent` is a good candidate for this.

``` jsx
interface Props {}
interface State {
    numbers: number[]
}
export class FatComponent extends React.Component<Props, State> {

    constructor(props) {
        super(props)
        this.state = { numbers: [1, 2, 3, 4] }
    }

    render() {
        const { numbers } = this.state
        return <div>
            <SimpleComponent title='Test' numbers={numbers}/>
            <SimpleComponent title='Test' numbers={numbers}/>
            <SimpleComponent title='Test' numbers={numbers}/>
            <SimpleComponent title='Test' numbers={numbers}/>
        </div>
    }
}

```

But what if our `FatComponent` does not have this data instantly and would need to download it?
Let's use a `fetchNumbers` method in `componentDidMount`.

``` jsx
interface Props {}
interface State {
    numbers: number[]
}
export class FatComponent extends React.Component<Props, State> {

    constructor(props) {
        super(props)
        this.state = { numbers: [] } // initially we have an empty numbers array
    }

    // async await - https://javascript.info/async-await
    async componentDidMount() {
        const numbers = await fetchNumbers() // this is my imaginary function that will provide me with numbers
        this.setState({ numbers })
    }

    render() {
        const { numbers } = this.state
        return <div>
            <SimpleComponent title='Test' numbers={numbers}/>
            <SimpleComponent title='Test' numbers={numbers}/>
            <SimpleComponent title='Test' numbers={numbers}/>
            <SimpleComponent title='Test' numbers={numbers}/>
        </div>
    }
}

```

Ok, so we have a component that knows how to fetch numbers and display them in `SimpleComponents`. Great!
But what if we want to reuse our `FatComponent` and present numbers from different source?
What if we do not want to fetch the data everytime our component mounts? - After all we can fetch this data once and we can use it in future.
What if we want to use a different initial array?
In order to do this we could add parameters to `FatComponent` and pass them from a parent that renders our `FatComponent`.

``` jsx

interface Props {
    // we moved numbers from State to Props as our FatComponent will not control the source of the numbers
    numbers: number[]
    // this is a method that FatComponent will call when it decides that it wants to refresh its numbers
    // we expect that if we call it, then FatComponent's parent will handle fetching the data and pass it to our component,
    // that's why we use "() => void" type
    refreshNumbers: () => void
}
interface State {
}
export class FatComponent extends React.Component<Props, State> {

    // async await is no longer needed here as we tell our parent to load data for us.
    componentDidMount() {
        this.props.refreshNumbers()
    }

    render() {
        const { numbers } = this.props // we no longer have numbers in state - we need to change it to props
        return <div>
            <SimpleComponent title='Test' numbers={numbers}/>
            <SimpleComponent title='Test' numbers={numbers}/>
            <SimpleComponent title='Test' numbers={numbers}/>
            <SimpleComponent title='Test' numbers={numbers}/>
        </div>
    }
}

interface BigBossProps {}
interface BigBossState {
    numbers: number[]
}
export class BigBossParent extends React.Component<BigBossProps, BigBossState> {

    constructor(props) {
        super(props)
        this.state = { numbers: [] }
    }

    async onFetchNumbers() {
        // if we fetched numbers before, then we won't do it again
        const hasDataOrPendingRequest = // check pending request && data existence
        if (!hasDataOrPendingRequest) {
            const numbers = await fetchNumbers() // this is my imaginary function that will provide me with numbers
            this.setState({ numbers })
        }
    }

    render() {
        return <FatComponent numbers={this.state.numbers} // we present numbers from BigBossParent's state in FatComponent
            refreshNumbers={this.onFetchNumbers.bind(this)}/> // Bind with `this` in order to use BigBossParent component as `this` in `onFetchNumbers` method
    }
}

```

Now if render logic in our `BigBossParent` changes and it will conditionally render `FatComponent` we will run into a situation where `onFetchNumbers` will be called multiple times. The catch here is that our `BigBossParent` is pretty smart, so it won't download any new data but reuse the old array.
But then again. If at some point we decide to `unmount` `BigBossParent`, then we will lose the state that is kept there and we will have to fetch it once again.
If we want to avoid this, we could move the state to... You guessed it! Another parent.
And this is where `Redux` comes with help to us. `Redux` provides us with a way to keep our application's state in one unified "parent" called `Store` that will provide it to the components that we render.
With `Redux` you will be able to:
- Keep your application state in one place - `Store`
- Write tests for your application's state changes in an easier way as you can test it decoupled from the UI part.
- Use a unified way of changing this state (via `Actions` and `Reducers`), which comes in handy when the project grows and you need to move around it.

Keep in mind that `Redux` is not a must and you do not need to use it for your application if you don't feel that you need it! - [You Might Not Need Redux](https://medium.com/@dan_abramov/you-might-not-need-redux-be46360cf367). But let's assume that we would like to introduce `Redux` to our example and keep numbers in this unified `Store`.
There are many approaches to how we can do it. The approach that is widely used and I personally like is connecting your main parent `components` with `Store` (in our case this would be `BigBossParent`) and then pass the required data to its children via their `props`. This way the rendered children are not aware of any `Redux` magic and if we decide to drop `Redux` at some point, then our all "dumber" (not connected to store) components would not require any changes.

How would we approach connecting our `BigBossParent` to store (Place in `Redux` where data is kept)?
First of all, we need to specify the input props of `BigBossParent` just as we did with `FatComponent`.
Just as before, we move the things that we do not want to control to `BigBossProps` and we hope that a thing that renders this component will
take care of them and give it to use.

``` jsx

interface BigBossProps {
    numbers: number[] // numbers will be provided to BigBossParent
    refreshNumbers: () => void // BigBossProps will not fetch the data on its own (just as FatComponent)
}
interface BigBossState {}
// we do not export this method anymore
// Please remember that we COULD do it and use this component as any other React component
class BigBossParent extends React.Component<BigBossProps, BigBossState> {

    render() {
        // If FatComponent decides to refreshNumbers, our BigBossParent will pass this request to its parent.
        return <FatComponent numbers={this.props.numbers}
            refreshNumbers={this.props.refreshNumbers()}/>
    }
}

export const connectedComponent = ... // we will get to this later

```

But what will be in charge of rendering our `BigBossParent`? We will render it in our applications "root" which will be connected to `Redux`.
Let's imagine that this `div` here is the root of our app. The first thing that will be presented here is `Provider`.
Provider, `createStore` is available through `react-redux` package and it will be responsible for providing components rendered inside it with a way
to connect with the main application `store`. We will be able to get the state from it and apply changes to it (Let's focus on "getting" the state now).
Provider will receive one parameter - a store which will be created with a `reducer` (let's not focus on them right now).
``` html
    <div>
        <Provider store={createStore(reducer)}>
            <BigBossParent /> // Where are the props that we normally would pass here?
            // Why don't we do it in as before?
            // <BigBossParent numbers={} refreshNumbers={}/>
        </Provider>
    </div>
```

Just before we move to our `BigBossParent` component, let's define an interface for our state in the application.
What I mean is that every time that we get the state from the store (that we created with `createStore(reducers)`), we expect that it will be of `ApplicationState` type.

``` js
interface ApplicationState {
    numbers: number[]
}

```

Instead of passing the props to BigBossParent in a usual way, we will use the `connect` that is available from `react-redux` package.

``` jsx

interface BigBossProps {
    numbers: number[] // numbers will be provided to BigBossParent
    refreshNumbers: () => void // BigBossProps will not fetch the data on its own (just as FatComponent)
}
interface BigBossState {}
// we will not export the old component
class BigBossParent extends React.Component<BigBossProps, BigBossState> {

    render() {
        // If FatComponent decides to refreshNumbers, our BigBossParent will pass this request to its parent.
        return <FatComponent numbers={this.props.numbers}
            refreshNumbers={this.props.refreshNumbers()}/>
    }
}

// This method will receive the application state in a first parameter
// its job is to take the part of the application state that BigBossParent is interested in and return it
// In this method we would like to exactly match the props that BigBossParent expects, however, we will not care about
// methods. (We will not provide refreshNumbers method through mapStateToPros)
function mapStateToProps(state: ApplicationState) {
    // this method will return object has "numbers" with a value of numbers that are kept in our application state
    return {
        numbers: state.numbers
    }
}

// This method will receive dispatch method as a first parameter
// The dispatch will allow us to send actions to the store.
// (if this concept is unfamiliar to you, please take a look at Redux documentation or my previous post - http://eliaszsawicki.com/story-of-redux/ )
function mapDispatchToProps(dispatch: Redux.Dispatch) {
    return {
        refreshNumbers: () => dispatch({
            type: 'UPDATE_NUMBERS',
            payload: { numbers: [1, 2, 3, 4, 5]}
        })
    }
}
// instead we will export the component that is connected to our application store.
// this means that the props that the BigBossParent component needed will be provided via our mapping functions
// functions through mapDispatchToProps and variables through mapStateToProps
export const connectedComponent = connect(mapStateToProps, mapDispatchToProps)(BigBossParent)
```

Let's take a quick look at `Reducer`. Do you remember that we have passed it as our `createStore` parameter?
`Reducer` is a function that takes in two parameters - `state` and `action` and returns a new `state`.

``` js
const DefaultState = { numbers: [] } // if we do not have a state yet (start of the app), we need to provide a default one
function reducer(state: ApplicationState = DefaultState, action: Action): ApplicationState {
    switch (action.type) {
        case 'UPDATE_NUMBERS': // This is the action type that we sent from our BigBossParent component.
            const newState = { numbers: action.payload.numbers }
            return newState
    }
    return state
}
```

In really simplified case we will have one reducer that handles our whole state, but in bigger apps we will have combined reducers that only take a part of the application state as a first parameter. The part that they know how to handle.
`UPDATE_NUMBERS` is the action type that we sent from our BigBossParent component. Let's take a look at `mapDispatchToProps` once again:

``` js
//the dispatch parameter is in fact way to call `store.dispatch()`.
function mapDispatchToProps(dispatch: Redux.Dispatch<ApplicationState>) {
    return {
        refreshNumbers: () => dispatch({
            type: 'UPDATE_NUMBERS',
            payload: { numbers: [1, 2, 3, 4, 5]}
        })
    }
}
```
What does this map do? At the time that we call `refreshNumbers` from `BigBossParent` component. What in fact happens is:
``` js
store.dispatch({
            type: 'UPDATE_NUMBERS',
            payload: { numbers: [1, 2, 3, 4, 5]}
        })
    }
```
This way we send an action to our store. Store receives the action and then passes both application state and this action to reducers (In our case this is a reducer mentioned above). It sees that the action type matches the one it handles - `UPDATE_NUMBERS` and creates
a new state accordingly. In our case it will apply the numbers sent as an action's payload. After it's done, the new state is returned and applied to the `store`.
This will now be the new state of our application. At the time that we receive this new state, our `BigBossParent` will be updated (mapping functions will be invoked again).

And this is how you go from a `React` to `React-Redux` ;)
If you have any comments, please share them below!


*This article is cross-posted with [my company blog](http://blog.brightinventions.pl/)*