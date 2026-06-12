### Event Handlers

Earlier in the course when we learned about intrinsic elements, we learned that event handlers are built into each intrinsic element. In fact, in React DOM, only intrinsic elements fire events; any other component that accepts an `onClick` or `onKeydown` prop will treat it like any other prop.

If we are passing an event handler directly to an intrinsic element, TypeScript will infer the type of the function's parameters for us.

```tsx
<button onClick={(event) => {
  // (parameter) event: React.MouseEvent<HTMLButtonElement, MouseEvent>
}}>
```

TypeScript automatically knows that `onClick` is a mouse event and that its target is an `HTMLButtonElement`. This is the easiest way to use event handlers.

If you need to write out your event handler function separately, such as if we are passing the event handler as a prop, you have a few options for adding the appropriate types.

First, the only type that really matters is the `event` parameter's type; all React event handlers have a `void` return type.

If we were to take advantage of our IDE, we could easily write out our event handler inline, like we did above, and then grab and use the type annotation which TypeScript infers. Here's what that would look like for an `onChange` event handler.

```tsx
const App = () => {
  function handleOnChange(event: React.FormEvent<HTMLInputElement>):void {
    // ...
  }

  return <input type="text" onChange={handleOnChange}>
}
```

Alternatively, you can use one of the many "EventHandler" types that are built into React's type definition, including `ChangeEventHandler`, `MouseEventHandler`, and `PointerEventHandler`. These are generic types that accept an element type and output a function type matching whatever event handler you need.

Here's that same `onChange` handler, but typed with `ChangeEventHandler` instead.

```tsx
const App = () => {
  const handleOnChange:ChangeEventHandler<HTMLInputElement> = (event) => {
    // ...
  }

  return <input type="text" onChange={handleOnChange}>
}
```

Notice, we don't have to annotate anything inside the function itself; our `ChangeEventHandler` type takes care of that for us.

Both of these are functionally equivalent - use whichever matches your preferences.

Sometimes, the properties on the `event` object aren't complete. One glaring example of this is form submit events. Take this simple form, for example:

```tsx
<form onSubmit={handleSubmit}>
  <div>
    <label>
      Email:
      <input type="email" name="email" />
    </label>
  </div>
  <div>
    <label>
      Message:
      <textarea name="message"></textarea>
    </label>
  </div>
</form>
```

Inside `handleSubmit`, the `event.target` property should represent the `<form>` element, and should have separate properties for each of the named inputs in the form. However, TypeScript isn't quite clever enough to infer that. We need to widen our `target` value to include those properties. We can do that using type widening.

Type widening is a powerful tool which can help you add properties to objects type definitions. It works by creating an intersection type between the original type of the object and an interface representing the properties you want to add.

```tsx
interface FormFields {
  email: HTMLInputElement;
  message: HTMLTextAreaElement;
}
function handleSubmit(event: React.FormEvent<HTMLFormElement>) {
  event.preventDefault();
  const target = event.target as typeof event.target & FormFields;

  const formValues = {
    email: target.email.value,
    message: target.message.value,
  };

  // Do whatever with the form values.
}
```

Notice that I'm able to maintain the original type of `event.target` by starting my intersection with `typeof event.target`. This lets me add additional fields to `event.target` without changing the other fields.

In some cases, we might want to use the same event handler with two different kinds of elements, such as buttons and anchor tags. In this case, it might make more sense to assert that our target is a union of the two element types.

```ts
function handleClick(
  event: React.MouseEvent<HTMLButtonElement | HTMLAnchorElement>,
) {
  const target = event.target as
    | HTMLButtonElement
    | HTMLAnchorElement;

  // If target has an `href` property, we know it's an anchor
  if ("href" in target) {
    target.href;
  }
}
```

All of the properties common to both buttons and anchors would be available without needing to type narrow; if we need to access a property unique to one of the element types, we can use the `in` operator to check for the existence of the property to narrow our type.
