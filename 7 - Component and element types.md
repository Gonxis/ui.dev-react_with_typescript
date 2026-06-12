### Component and Element Types

Before we go much further, we need to be clear about the different kinds of things that we work with as we write JSX.


#### [Elements](https://reactjs.org/docs/glossary.html#elements)

The most often used function in React is `React.createElement`, but you don't normally see it. That's because our JSX code is transformed into `React.createElement` calls. This function turns a component into an element, which can then be rendered to the DOM (or wherever else) using `ReactDOM.render`. When you call `React.createElement`, you pass in whatever props you want your element to have. It can then begin maintaining its own state.

Here are a few elements, and the results of calling `console.log` on one of them.

```tsx
const helloElement = <Hello />;
const worldElement = React.createElement(World);
// {
//   '$$typeof': Symbol(react.element),
//   type: [Function: World],
//   key: null,
//   ref: null,
//   props: {},
//   _owner: null,
//   _store: {}
// }
```

These React elements represent a single, unique immutable object that is returned from a component. If we were to look at the return type of `React.createElement`, we would see that it is `React.Element`, which makes perfect sense. This is the case whether we are using `React.createElement` directly, or using JSX.

We can use `ReactElement` as a prop.

```tsx
function RenderElement({ element }: { element: ReactElement }) {
  return <div>{element}</div>;
}
```

Since our element already is an element, we don't need to wrap it in JSX brackets - we can just plop it right into our component.

`ReactElement` is an object shaped like what we saw up above, with `key`, `props`, `ref`, etc. What about components that return something other than `ReactElement`, like a string or array of numbers? If we were to try putting one of these components inside a JSX block, TypeScript would give us a warning.

```tsx
function RenderString() {
  return "Hello world!";
}

function App() {
  return (
    <div>
      <RenderString />
      {/* Type Error: 'RenderString' cannot be used as a JSX component.
      Its return type 'string' is not a valid JSX element */}
    </div>
  );
}
```

To get this to work, we need to convince TypeScript that our string is actually a `ReactElement`. To do that, we could assert that our string is `unknown`, and then assert that it is a `ReactElement`.

If you remember doing this from the TypeScript course, you know that double type assertions are _dangerous_. We are lying to the type checker, and if we're wrong about the types, we could have a very hard time tracking down where our runtime type errors are coming from. We can give ourselves just a bit more type safety by asserting our string is a type that is common between `ReactElement` and strings: `ReactNode`. This is a type built into React's type definition that represents anything that can be used as a child of a React component, including strings and arrays.

```tsx
import { ReactNode, ReactElement } from "react";

function RenderString() {
  return ("Hello world!" as ReactNode) as ReactElement;
}
```

Now, if we use our component in JSX, the type errors have gone away.

There is a slightly easier way to solve this. Just wrap our string in a React fragment to turn it into a `ReactElement`.

```tsx
import { Fragment } from "react";

function RenderString() {
  return <Fragment>Hello world!</Fragment>;
}
```

How you handle this is up to you. Just know that you have a few options.


#### [Components](https://reactjs.org/docs/glossary.html#components)

A React component is a class or function which returns a React Node. Components define what props you can pass and what state can be held by the component, but the component itself doesn't hold state. It's just a template. Since you can call a function or instantiate a class multiple times, React components are intended to be used multiple times.

These are components:

```tsx
class Hello extends React.Component {
  render() {
    return <div>Hello</div>;
  }
}
const World = () => {
  return <div>World</div>;
};
```

These two component definitions, a class component and a function component, can be represented with a single type built into React's type definitions: `React.ComponentType<P>`, with `P` being the props of the component. Unsurprisingly, the type definition for `React.ComponentType<P>` is just a union of a class component and function component definition.

```ts
type ComponentType<P = {}> =
  | ComponentClass<P>
  | FunctionComponent<P>;
```

This next example is a little contrived, but it does show one way you might use `ComponentType`.

We can create a component that accepts another _component_ as a prop, and renders that component as an element. We can use `ComponentType` to annotate our `Comp` prop.

```tsx
function RenderComponent({ Comp }: { Comp: ComponentType }) {
  return (
    <div>
      <Comp />
    </div>
  );
}
```

We'll talk more about using components as props later in the course.
