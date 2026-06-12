### Function Components

If you know how add type annotations to a regular function, you are 99% of the way towards annotating a React function component. The only thing to remember is that React components all have to return either a React Element or `null`, so it's usually a good idea to add a return type annotation to your function so TypeScript warns us if we return the wrong thing.

We'll use the `React.ReactElement | null` type as our return type. This type represents the output of `React.createElement`, which is what you get when you write any JSX. It also allows us to return `null` if we don't want our component to render anything.

```tsx
import { ReactElement } from "react";

function HelloWorld(): ReactElement | null {
  return <div>Hello world!</div>;
}
```

If you try to return something other than `ReactElement`, like string or number, you might run into some trouble. We'll talk about this when we dive into the built-in types that come with React.

Props can be annotated by giving our function parameters an annotation. Any JSX that calls our function component will be type checked against the type we assign to the props parameter.

```tsx
function Message({ message }: { message: string }): ReactElement {
  return <div>A message: {message}</div>;
}
```

If you want to explicitly annotate a function as a React component, you can use the built-in `React.FunctionComponent` type, or it's alias `React.FC`. This type includes annotations for common static properties that are added to function components, like `displayName`, `propTypes`, and `defaultProps`. It also includes the children prop in your prop definition, and has an explicit return type.

We can use `FC` by annotating a variable that we assign an arrow function component. `FC` and `FunctionComponent` are generic types which accept a type representing the props.

```tsx
import { FC } from "react";

const Message: FC<{ message: string }> = ({
  message,
  children,
}) => {
  return (
    <div>
      A message: {message}
      <br />
      Children: {children}
    </div>
  );
};
```

If you are planning on doing anything unique or special with the type of `children`, you likely want to annotate it directly; in that case, skip using `React.FC`. We discuss all of the ways to annotate the `children` prop later in the course. For now, use whatever style you prefer.

Managing state and lifecycles in function components is done with hooks. Since hooks are a topic unto themselves, we'll cover them in a later section of the course.
