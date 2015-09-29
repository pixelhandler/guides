You can think of a component as a black box of UI functionality. So far,
you've learned about how parent components can pass attributes in to a
component, and how the component can use those attributes from both
JavaScript and its Handlebars template.

But what about the opposite direction? How does data flow back out of
the component to the parent? In Ember, components use something called
**actions** to communicate events and changes.

To the outside world, a component is entirely self-contained: a black
box where attributes go in, and actions come out.

### A Simple Action

Let's look at a simple example of how a component can use an action to
communicate with its parent.

Imagine we're building an application where users can have accounts. We
need to build the UI for users to delete their account. Because we don't
want users to accidentally delete their accounts, we'll build a button
that requires the user to double click it in order to trigger some
action.

Best of all, because components are "black boxes" that aren't coupled to
their parent component, once we create this "double click button"
component we can reuse it all over our application.

Let's call our component `double-click-button`. We can create it by
typing:

```text
ember generate component double-click-button
```

Remember that you can use this component from a template like this:

```hbs
{{double-click-button}}
```

When implementing an action, you need to break it down into two steps:

1. In the outside world, decide how you want to react to the action.
2. In the component, determine when something has happened, and tell the
   outside world.

In this example, that means:

1. In the outside world, interpret double clicks on the button as "the
   user wants to delete their account."
2. In the component, detect that the user has double clicked the button
   and trigger the action.

Let's take it step by step.

In the outside world, let's first define what we want to happen when the
user double clicks the button. In this case, we'll find the user's
account and delete it.

Where do you put code that should be triggered by an action? In Ember,
each component can have an `actions` hash where code that is triggered
by child components can go. Let's look at what the parent component's
JavaScript might look like. In this example, imagine we have a parent
component called `user-profile` that shows the user's profile to them.

```app/components/user-profile.js
export default Ember.Component.extend({
  login: Ember.inject.service(),

  actions: {
    userDidDeleteAccount() {
      this.get('login').deleteUser();
    }
  }
});
```

In this example, we have an action on the parent component called
`userDidDeleteAccount()` that, when called, gets a hypothetical `login`
service and calls the service's `deleteUser()` method. However, we have
not told Ember when we want this action to be triggered, which is the
next step.

What comes next is arguably the most important part of building a
component: deciding what the event that triggers the action should be
called.

In this example, we want something to happen whenever the component is
double clicked. Let's call this event `onDoubleClick`. We can tell Ember
to trigger the `userDidDeleteAccount()` action on our parent whenever
the child component's `onDoubleClick` event is called:

```app/components/user-profile.hbs
{{double-click-button onDoubleClick=(action 'userDidDeleteAccount')}}
```

This snippet says "take the `userDidDeleteAccount` action from the
parent and make it available on the child component as
`onDoubleClick`."

Next, how do we detect that a user has double clicked our component?
Ember has built-in support for common browser events like this. In this
case, in our component's JavaScript, we can implement a method called
`doubleClick()`. Every time the user double clicks this component, Ember
will trigger the `doubleClick()` method.

```app/components/double-click-button.js
export default Ember.Component.extend({
  tagName: 'button',

  // This method is called every time the user double clicks the
  // component.
  doubleClick() {
  }
});
```

Now that we've implemented a method to detect whenever a double click
happens, we still need to tell the outside world to trigger the
`userDidDeleteAccount` action that we've been given.  Let's do that now.

One important thing to know about actions is that they're **just
functions** you can call, like any other method on your component.

In this case, the components `onDoubleClick` method is a function that
will call the parent component's `userDidDeleteAccount` action. Here's
how to call it:

```app/components/double-click-button.js
export default Ember.Component.extend({
  tagName: 'button',

  doubleClick() {
    this.onDoubleClick();
  }
});
```

That's it! Under the hood, `onDoubleClick` is a function that knows how
to break through the normal isolation of components and trigger the
`userDidDeleteAccount` on its parents.

Just like normal attributes, actions set a property on the component;
the only difference is that the property is actually a function that
knows how to trigger behavior in the parent component.

That makes it easy to remember how to add an action to a component. It's
just like passing an attribute, but you use the `action` helper to pass
a function instead. In the future, for example, we may want to be able
to pass a `title` attribute to the `double-click-button` component. You
can see how similar the syntax is:

```app/component/user-profile.hbs
{{double-click-button
  title="Delete Account"
  onDoubleClick=(action 'userDidDeleteAccount')}}
```

You may have noticed that component actions are similar to another
concept in JavaScript: **callbacks**. You can think of actions as being
like callbacks for components.

In JavaScript, you can pass a callback to a function that it can invoke
at a later time. For example, we might have a helper function that takes
a callback and adds it as an event listener on an element:

```js
function onClick(element, callback) {
  element.addEventListener('click', callback);
}

let element = document.getElementById('delete-button');
onClick(element, () => {
  alert("button clicked!");
});
```

In this example, we pass a callback function to the `onClick` function
that will be called whenever the button is clicked. This pattern allows
us to separate the functionality of listening for a click (the `onClick`
function) from what should happen when the click happens (the passed
callback).

Like callbacks in JavaScript, actions in components allow you to
decouple an event happening from how it's handled, leading to modular,
more reusable components.
