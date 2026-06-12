### Refs

In class components, refs are only used to store references to the underlying HTML elements which our intrinsic elements render to the page. We create the ref using `React.createRef` and assign it to an instance property on our component. When we assign the ref to an intrinsic element, React puts the DOM node into our ref.

```tsx
class MyComponent extends React.Component {
  private myRef = React.createRef();
  render() {
    return <div ref={this.myRef} />;
    // Type Error: Type 'RefObject<unknown>' is not assignable to type 'RefObject<HTMLDivElement>'
  }
}
```

TypeScript isn't able to infer the type of our ref from how we use it. It has no way of knowing that we are assigning our ref to an `HTMLDivElement`. Because of that, whenever we are using the ref to get a reference to a DOM element, we have to pass an HTML element type in as the generic argument for `createRef`. Then the warning will go away.

```tsx
class MyComponent extends React.Component {
  private myRef = React.createRef<HTMLDivElement>();
  render() {
    return <div ref={this.myRef} />;
  }
}
```

Now, if we access `this.myRef.current`, it will be an `HTMLDivElement` and we'll have type safety as we work with it.

Refs with hooks is a little more nuanced. Function components use refs both for holding references to DOM elements and storing bits of data that don't affect rendering, like class component instance properties. The way we use the `useRef` hook is slightly different in each case.

If we're collecting a reference to a DOM element, we likely want to initialize our ref with `null`; this mimics the behavior of `createRef`. Also like `createRef`, we need to provide a type to the generic that represents the type of the ref's contents.

With `strictNullChecks` turned on, we'll also need to verify that our ref is set before we try to access it.

```tsx
const Form = () => {
  const inputRef = useRef<HTMLInputElement>(null)

  const onClick = () => {
    if (inputRef.current) {
      inputRef.current.focus()
    }
  }
  return <input type="text" ref={inputRef} onClick={onClick}>
}
```

Using refs for any other value, such as arbitrary strings or objects, works exactly the same as `useState` - whatever we pass as the initial value of the ref determines the type of the ref. If we need to override that, we can provide a type to the generic.

```tsx
const App = () => {
  const stringRef = useRef("Hello there!");
  const maybeNumberRef = useRef<number | null>(null);
  // ...
};
```


#### Forwarding Refs

Ref forwarding lets you pass a ref through a component to one of its children. It's not very common, but at very least you can enjoy type safety while you use it!

When we wrap our component in `React.forwardRef`, we have to provide generic types for the ref itself and for the wrapped component's props. The ref type is defined _first_, followed by the props, even though the function parameters put the ref after the props.

We have to provide the props type even if we've already defined those types in the function declaration. Typically, you'll have the easiest time if you wrap your component as you define it, like this:

```tsx
import { forwardRef } from "react";
const Input = forwardRef<HTMLInputElement, { disabled?: boolean }>(
  ({ disabled }, ref) => {
    return <input ref={ref} disabled={disabled} />;
  },
);
```
