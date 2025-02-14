# Dispatch

**_Definition:_**

> The **dispatch** function controls Hyperapp's core dispatching process which executes [actions](actions.md), applies state transitions, runs [effects](effects.md), and starts/stops [subscriptions](subscriptions.md) that need it.

You can augment the dispatcher to tap into the dispatching process for debugging/instrumentation purposes. Such augmentation is loosely comparable to middleware used in other frameworks.

**_Signature:_**

```elm
DispatchFn : (Action, Payload?) -> void
```

---

## Dispatch Initializer

The dispatch initializer accepts the default dispatch as its sole argument and must give back a dispatch in return. Hyperapp's default dispatch initializer is equivalent to:

```js
const boring = (dispatch) => dispatch
```

In your own initializer you'll likely want to return a variant of the regular dispatch.

---

## Augmented Dispatching

A dispatch function accepts as its first argument an [action](actions.md) or anything an action can return, and its second argument is the default [payload](actions.md#payloads) if there is one. The payload will be used if the first argument is an action function.

The action will then be carried out and its resulting state transition will be applied and then any effects it requested to be run will be run.

```js
// DispatchFn : (Action, Payload?) -> void
const dispatch = (action, payload) => {
  // Do your custom work here.
  // ...

  // Hand dispatch over to built-in dispatch.
  dispatch(action, payload)
}
```

## Dispatch recursion

Dispatch is implemented in a recursive fashion, such that if the action dispatched does not represent the next state (or next state with effects), it will use the dispatched action and payload to resolve the next thing to dispatch. 

A call to `dispatch([ActionFn, payload])` will recurse `dispatch(ActionFn, payload)`, which will recurse to `dispatch(ActionFn(currentState, payload))`. 

---

## Example 1 - Log actions

Let's say you need to debug the order in which actions are dispatched. An augmented dispatch that logs each action could help with that, rather than having to add `console.log` to every action.

```js
const mwLogActions = dispatch => (action, payload) => {

  if (typeof action === 'function') {
    console.log('DISPATCH: ', action.name)
  }

  //pass on to original dispatch
  dispatch(action, payload)
}

```

---

## Example 2 - Log state

If you instead are more interested in just logging each state transformation, an augmented dispatch such as this will work:

```js
const mwLogState = dispatch => (action, payload) => {

  if (Array.isArray(action) && typeof action[0] !== 'function') {
    console.log('STATE:', action[0])
  }
  if (!Array.isArray(action) && typeof action !== 'function') {
    console.log('STATE:', action)
  }

  //pass on to original dispatch
  dispatch(action, payload)
}

```



## Usage

The [`app()`](../api/app.md) function will check to see if you have a dispatch initializer assigned to the [`dispatch:`](../api/app.md#dispatch) property while instantiating your application. If so, your app will use it instead of the default one.

The only time the dispatch initializer gets used is once during the instantiation of your app.

Only one dispatch initializer can be defined per app. Consequently, only one dispatch can be defined per app.

Extending the example from above, the dispatch initializer would be used like this:

```js
import { mwLogState } from "./middleware.js"

app({
  // ...
  dispatch: mwLogState
})
```

And if you wanted to use both custom dispatches together, you can chain them like this:

```js
import { mwLogState, mwLogActions } from "./middleware.js"

app({
  // ...
  dispatch: dispatch => mwLogState(mwLogActions(dispatch))
})
```


---

## Other Considerations

- [`app()`](../api/app.md) returns the dispatch function to allow [dispatching externally](../api/app.md#instrumentation).

- If you're feeling truly adventurous and/or know what you're doing you can choose to have your dispatch initializer return a completely custom dispatch from the ground up. For what purpose? You tell me! However, a completely custom dispatch won't have access to some important internal framework functions, so it's unlikely to be something useful without building off of the original dispatch.
