# Chapter 5: Actions and DOM events

## Modeling state

**Add Post** button always adds a post with the same text. 

Add an input field to change the text:
```js
<h4>hyperposts</h4>
<input type="text" autofocus />
${" "}
<button onclick=${AddPost}>Add Post</button>
```
We added a whitespace between the input and the button to make the UI nicer.

When you start typing text into the input, your state and view will get out of sync. What you're typing is not
reflected in the state object.
One of the tenets of functional UI architecture is the continuous synchronization of state and view. 
View reacting to state changes, and state changes reacting to view actions. 
To make this work, you need some part of your state to model the contents of the input field. 

Create a new state property named `currentPostText`:
```js
const state = {
  currentPostText: "type your text",
  posts: [...]
};
```
Read the new property in your view:
```js
<input type="text" value=${state.currentPostText} autofocus />
```
DOM attribute called `value` sets the text of the input field to the `currentPostText`.

## Accessing DOM events

Input text reflects `currentPostText` from the state object. You want to close the loop with DOM events changing the state.

Add DOM `oninput` attribute to trigger `UpdatePostText` action on input changes:
```js
<input type="text" oninput=${UpdatePostText} value=${state.currentPostText} autofocus />
```

Add a new action next to the `AddPost` action:
```js
const UpdatePostText = (state, event) => ({
    ...state,
    currentPostText: event.target.value
});
```
Compare the `UpdatePostText` signature with the `AddPost` signature.

```
(oldState) => newState
(oldState, event) => newState
```
Hyperapp actions accept either `(oldState)` or `(oldState, event)`. 
With a second attribute provided, Hyperapp will inject both sources of information to your action.
The `event` is a regular DOM event. Therefore we can access `event.target.value` from the DOM Event API. 
As mentioned before, it's all about transferable skills. 

The following figure shows the updated conceptual model of Hyperapp actions with the extra event attribute:

![Figure: Action is a pure function of state and event](images/action-with-event.png)

Try to add a new post with some text. It should still not work. You need to copy the `currentPostText` to the newly added post.

```js
const AddPost = (state) => {
    const newPost = { username: "anonymous", body: state.currentPostText };
    return { ...state, posts: [...state.posts, newPost] };
};
```

With this change, you can start adding custom messages to the list.

![Figure: Adding custom messages to the list](images/custom-messages.png)

## Extracting repetitive event data

All event based actions will follow a similar pattern:
```js
(oldState, event) => {
    const userData = event.target.value;
    ....
}
```
Action code would be cleaner if it didn't know about the DOM Event API.

Create a **custom payload filter** to extract only a part of the event you care about:
```js
const targetValue = event => event.target.value;
```
Eventually, you may move this code to a library but for now, put it somewhere above your view declarations.

Switch `UpdatePostTest` to use the new function:
```js
const UpdatePostText = (state, event) => ({
    ...state,
    currentPostText: targetValue(event)
});
```
The code is still dependent on the `targetValue` function.

Ideally, you'd like the action to accept only the data it needs:
```js
const UpdatePostText = (state, currentPostText) => ({
    ...state,
    currentPostText
});
```
Shape the second argument of your action using the following snippet: 
```js
const withFilter = filter => action => (state, event) => action(state, filter(event));
const withTargetValue = withFilter(targetValue);
<input type="text" oninput=${withTargetValue(UpdatePostText)} value=${state.currentPostText} autofocus />
```
`withFilter` is a curried function that can be first configured with a given filter e.g. `targetValue` and then
applied to a given action e.g. `UpdatePostText`.  It returns a new action that applied filter to the original event.

If you keep using `withTargetValue(UpdatePostText)` over and over, consider creating an alias:
```js
const UpdatePostTextAction = withTargetValue(UpdatePostText);
```
That you can pass directly:
```js
<input type="text" oninput=${UpdatePostTextAction} value=${state.currentPostText} autofocus />
```

## Exercises

According to [modern research](https://en.wikipedia.org/wiki/Desirable_difficulty), testing your knowledge is essential for learning. 
If you want to get the most out of this book, please do the exercises. They are not optional.

### Exercise: cleaning text input

Your application doesn't clear the input text after adding a new post.
1. Modify the `AddPost` action to reset `currentPostText`. 
2. Make sure the initial text is empty.

Try to do it on your own first. 
When you're done, compare with the solution below.

<details>
    <summary id="cleaning_text_input">Solution</summary>

```js
const AddPost = (state) => {
  const newPost = { username: "anonymous", body: state.currentPostText };
  return { ...state, currentPostText: "", posts: [...state.posts, newPost] };
};
```

</details>

### Exercise: checking empty input

After we clean the input, users may accidentally submit the empty text. Your task is to prevent them from doing so.
The application should ignore **Add Post** clicks when the text is empty.

<details>
    <summary id="checking_empty_input">Solution</summary>

```js
const AddPost = (state) => {
  if(state.currentPostText.trim()) {
      const newPost = { username: "anonymous", body: state.currentPostText };
      return { ...state, currentPostText: "", posts: [...state.posts, newPost] };
  }  else {
      return state;
  }
};
```

</details>
