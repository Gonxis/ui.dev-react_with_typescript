### (Bonus) Intrinsic Elements

In React, intrinsic elements represent the basic units that React uses to render a component with a particular renderer. In ReactDOM, these are `div`, `span`, and all the other HTML elements.

> Intrinsic elements don't really exist in React Native; instead, you import components directly from React Native that are translated to the native elements. This lesson only focuses on React DOM.

The type declarations for React come with special types which can help us work with these intrinsic elements.

The React type declarations add a namespace called `JSX` to the global. We can access this anywhere that we are using React. This is where the type declarations for intrinsic elements comes from.

For example, if you were to hover over the `<button>` element in this example using VS Code, you would see that it is an instance of `JSX.IntrinsicElements.button` which defines the props for that element. For example, `<button>` can have a `type` and `disabled` prop, but not a `width` prop.

```tsx
const MyButton = () => {
  return <button>Click Me!</button>;
};
```

If we dive into the definition for `JSX.IntrinsicElements.button`, we would seem something that looks like this:

```ts
interface IntrinsicElements {
  // ...
  button: React.DetailedHTMLProps<
    React.ButtonHTMLAttributes<HTMLButtonElement>,
    HTMLButtonElement
  >;
  // ...
}
```

There's a lot going on here, so lets break it down.

`HTMLButtonElement` is a type that represents a button in the DOM. It's part of the standard library that is built into TypeScript. React is using to to say "This intrinsic element renders as this DOM node." If we were to attach a ref to this element, `HTMLButtonElement` is the type that would be assigned to `ref.current`.

Then, we have the `React.ButtonHTMLAttributes` generic. Here's what that looks like:

```ts
interface ButtonHTMLAttributes<T> extends HTMLAttributes<T> {
  autoFocus?: boolean;
  disabled?: boolean;
  form?: string;
  formAction?: string;
  formEncType?: string;
  formMethod?: string;
  formNoValidate?: boolean;
  formTarget?: string;
  name?: string;
  type?: "submit" | "reset" | "button";
  value?: string | string[] | number;
}
```

This interface extends the `HTMLAttributes` interface, and creates prop definitions for all of the attributes that are unique to buttons. `HTMLAttributes` has prop definitions for all of the attributes which are common among HTML elements.

You might be wondering "So what are we doing with that generic `T` there?" Recall that `T` represents the `HTMLButtonElement` type. If we follow the types, so to speak, we eventually end up at the `DOMAttributes` type, which has definitions for all of the events that you can attach to DOM nodes. Here's what `onClick` looks lke.

```ts
interface DOMAttributes<T> {
  // ...
  onClick?: MouseEventHandler<T>;
  // ...
}
```

Aha! Now we are getting somewhere. And what is the definition for `MouseEventHandler`?

```ts
type MouseEventHandler<T = Element> = EventHandler<MouseEvent<T>>;
```

Remember, React uses a synthetic event system which is an abstraction on top of the regular DOM event system. This helps with cross-browser compatibility and makes it so events all behave consistently. These types here model that synthetic event system. The `EventHandler` type is just a function that accepts an event object as a parameter, and returns void, like this:

```ts
type EventHandler<E extends SyntheticEvent> = (e: E) => void;
```

`SyntheticEvent` is another interface with the properties common of all events, like `e.preventDefault()`.

`MouseEvent` extends `SyntheticEvent` and adds a few more properties which are exclusive of `MouseEvents`.

```ts
interface MouseEvent<T = Element, E = NativeMouseEvent>
  extends SyntheticEvent<T, E> {
  // ...
  button: number;
  buttons: number;
  clientX: number;
  clientY: number;
  // ...
}
```

And that's about as deep as we can get in our type spelunking.

To recap, the type definitions for Intrinsic Elements are built into the React type declarations, and include all of the DOM elements that you could possibly use. Some intrinsic elements, like `<button>` have unique prop definitions which match the attributes you can assign to an actual HTMLButtonElement. All intrinsic elements have common properties, including event handlers. And, depending on which event handler you are using, the event parameter might have some extra properties, like how `onClick`'s event parameter has properties for which mouse button did the clicking.

This lesson is just an explainer, designed to give you an in-depth look at these types. In a later section, we'll learn about how we can use these event types as we're working with forms and events.


