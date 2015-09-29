You can think of a component as a black box of UI functionality. So far,
you've learned about how parent components can pass attributes in to a
component, and how the component can use those attributes from both
JavaScript and its Handlebars template.

But what about the opposite direction? How does data flow back out of
the component to the parent? In Ember, components use
**actions** to communicate events and changes.

## A Simple Action

Let's look at a simple example of how a component can use an action to
communicate with its parent.

Imagine we're building an application where users can have accounts. We
need to build the UI for users to delete their account. Because we don't
want users to accidentally delete their accounts, we'll build a button
that requires the user to confirm in order to trigger some
action.

Once we create this "button with confirmation"
component, we want to be able to reuse it all over our application.

### Creating the Component

Let's call our component `button-with-confirmation`. We can create it by
typing:

```shell
ember generate component button-with-confirmation
```

We'll plan to use the component in a template something like this:

```app/templates/components/user-profile.hbs
{{button-with-confirmation text="Click OK to delete your account."}}
```

We'll also want to use the component elsewhere, perhaps like this:

```app/templates/components/send-message.hbs
{{button-with-confirmation text="Click OK to send your message."}}
```

### Designing the Action

When implementing an action on a component, you need to break it down into two steps:

1. In the outside world, decide how you want to react to the action.
   Here, we want to have the action delete the user's account in one place, and
   send a message in another place.
2. In the component, determine when something has happened, and when to tell the
   outside world.
   Here, we want to trigger the outside action (deleting the account or sending
   the message) after the user confirms that they really do want it to happen.

Let's take it step by step.

### Implementing the Action

In the outside world, let's first define what we want to happen when the
user double clicks the button. In this case, we'll find the user's
account and delete it.

In Ember,
each component can have an `actions` hash. Let's look at what the parent component's
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
service and calls the service's `deleteUser()` method.

Next, let's implement the logic to confirm that the user wants to take
the action from the component:

```app/components/button-with-confirmation.js
export default Ember.Component.extend({
  tagName: 'button',
  click() {
    if (confirm(this.get('text'))) {
      this.onConfirm()
    }
  }
});
```

### Passing the Action to the Component

Now we just need to make it so that the `onConfirm()` event in the
`button-with-confirmation()` component triggers the
`userDidDeleteAccount()` action in the `user-profile` component.
One important thing to know about actions is that they're **just
functions** you can call, like any other method on your component.
So they can be passed from one component to another like this:

```app/components/user-profile.hbs
{{button-with-confirmation text="Click here to delete your account." onConfirm=(action 'userDidDeleteAccount')}}
```

This snippet says "take the `userDidDeleteAccount` action from the
parent and make it available on the child component as
`onConfirm`."

We can do a similar thing for our `send-message` component:

```app/templates/components/send-message.hbs
{{button-with-confirmation text="Click to send your message." onConfirm=(action 'sendMessage')}}
```

Actions in components allow you to
decouple an event happening from how it's handled, leading to modular,
more reusable components.
