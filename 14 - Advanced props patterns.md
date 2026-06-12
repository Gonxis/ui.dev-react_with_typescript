### Advanced Props Patterns


#### Children

Earlier in the course, we used `React.ReactElement` as the return type for our function components. That type represents a React component, so naturally we would think that is the type we should use for the `children` prop.

There's only one problem: `children` can be much more than just `ReactElement`s. Observe:

```tsx
const Layout = ({ children }: { children: ReactElement }) => {
  return <div>{children}</div>;
};

const App = () => {
  return <Layout>Hello there!</Layout>;
  // Type Error: 'Layout' components don't accept text as child elements.
  //   Text in JSX has the type 'string', but the expected type of 'children' is 'ReactElement'
};
```

The `children` prop of `Layout` is definitely not a `ReactElement`; it's a `string`. `string` isn't compatible with `ReactElement`, so if we want to use a `string` as children for this component, we'll need to use a union type for our `children` prop.

```ts
type ReactChildrenProp = ReactElement | string;
```

We might as well add `number` to the list too, since it's possible to put a number in as children.

```tsx
const App = () => {
  return <Layout>{100}</Layout>;
};
```

How about `boolean`s, `null`, and `undefined`? Well, TypeScript doesn't try to render those out as text. In fact, if you pass any of those to the `children` prop, it just ignores it and renders nothing. Yes, even `true` doesn't get rendered as text.

Now we've got a type that looks something like this:

```ts
type ReactChildrenProp =
  | ReactElement
  | string
  | number
  | boolean
  | null
  | undefined;
```

As it turns out, there already is a type that matches what we have there: `ReactNode`.

```ts
type ReactNode = ReactChild | boolean | null | undefined;
type ReactChild = ReactElement | ReactText;
type ReactText = string | number;
```

If you want the most flexibility for your `children` prop, use `ReactNode`. In fact, if you annotate your components with `React.FC`, that's the type that it uses for `children` by default.

Of course, there might be times that we want to constrain the type of the `children` prop, such as with these components.

```tsx
const DoubleNumber = ({ children }: { children: number }) => {
  return <>{children * 2}</>;
};
const UppercaseString = ({ children }: { children: string }) => {
  return <>{children.toUpperCase()}</>;
};

<>
  <DoubleNumber>{100}</DoubleNumber>
  <UppercaseString>Hello there!</UppercaseString>
</>;
```

Note that we can't constrain everything about our `children` prop. For example, since every component eventually becomes a `ReactElement` before being passed in as `children`, there's no way to know specifically what React Component created that `ReactElement`.


#### Render Props

A Render Prop is technically any prop that accepts a function that returns a `ReactNode`. It's often used for sharing code between React components. In fact, React itself uses a render prop for the `<Context.Consumer>` component.

React Hooks solve the same problems that render props solve, but you might find some situations where a render prop is the right choice.

[Implementing render props](https://reactjs.org/docs/render-props.html) can be a little tricky, but writing the type annotation for them is very simple. Suppose we had a component that calculated the current mouse position and provided those in render prop form. Consuming this component might look something like this:

```tsx
// Render prop style
<MousePosition render={({x,y}) => <div>{x}, {y}</div>} />
// Children prop style
<MousePosition>{({x,y}) => <div>{x}, {y}</div>}</MousePosition>
```

The type annotation isn't too complicated at all. We'll mark the `render` and `children` props as optional, and have them both be functions that have a mouse position interface as the parameter and return a `ReactNode`.

```tsx
import { ReactNode, FC } from "react";

interface MousePositions {
  x: number;
  y: number;
}

interface MousePositionProps {
  render?: (MousePositions) => ReactNode;
  children?: (MousePositions) => ReactNode;
}

const MousePosition: FC<MousePositionProps> = ({
  render,
  children,
}) => {
  // ...
};
```


#### Style

The `style` prop lets us adjust the CSS of a specific intrinsic element. If you ever need to annotate the `style` prop, you can use the `React.CSSProperties` type to do that.

```tsx
import { CSSProperties } from "react";

const Button: React.FC<{ style: CSSProperties }> = ({
  style,
  children,
}) => {
  return <button style={style}>{children}</button>;
};
```

We can use TypeScript's utility types to make it so only certain style attributes can be assigned through props.

```tsx
type AllowedStyles = "display" | "backgroundColor";

const Button: React.FC<{
  style: Pick<CSSProperties, AllowedStyles>;
}> = ({ style, children }) => {
  return <button style={style}>{children}</button>;
};

<Button style={{ fontSize: 24 }}>Click me!</Button>;
// Type Error: Type '{ fontSize: number; }' is not assignable to type 'Pick<CSSProperties, AllowedStyles>'
```


#### Mirroring HTML Elements

In the above example, we created a `<Button>` component that renders the `<button>` HTML Element. We're only passing two props to our child button, but we likely want to pass all of the props that are available to the `<button>` element.

React gives us a utility type which extracts the props from an HTML element. Often, if we are creating a component that wraps an intrinsic element, we likely want to add more props that control some special behavior. We can use type widening to combine our props with the intrinsic props by either creating an intersection type with `&` or creating an interface that extends the element's props.

```tsx
import { ComponentPropsWithoutRef } from "react";

// Option 1
type ButtonProps = {
  variant?: "primary" | "success" | "warning" | "danger";
} & ComponentPropsWithoutRef<"button">;

// Option 2
interface ButtonProps extends ComponentPropsWithoutRef<"button"> {
  variant?: "primary" | "success" | "warning" | "danger";
}

const Button: React.FC<ButtonProps> = ({
  variant,
  className = "",
  ...props
}) => {
  return (
    <button {...props} className={`${className} ${variant}`} />
  );
};
```

Notice when we pull `ButtonProps` out of the intrinsic `button` element, we pass it a string. We can use a string for any of the intrinsic elements. If we want to pull the props out of a non-intrinsic component, we just have to pass the type of that component in, like so:

```ts
type ButtonComponentProps = ComponentPropsWithoutRef<typeof Button>;
// type ButtonComponentProps = {
//   variant?: "primary" | "success" | "warning" | "danger";
//   etc. etc...
// }
```

If we wanted to restrict which props the consumer of our component can assign to this element, we could also use TypeScripts utility types to Pick or Omit properties from the `ButtonProps` interface.

We'll talk more about `forwardRef` in a later section in the course, but if we wanted to pull off all of the props that a component has _including_ a ref assigned to it, we could use `ComponentPropsWithRef`. This does the same thing, but includes `ref` in the props definition if applicable.
