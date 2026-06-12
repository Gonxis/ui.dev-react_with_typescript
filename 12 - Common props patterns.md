### Common Props Patterns

Props is always an object. This makes it really easy to annotate props with a type, since we can use any of the usual techniques for objects when writing our props annotation.

I'll use interfaces for my examples, but the same principle applies to type aliases or literal object types.

Here's a component with a bunch of different kinds of props.

```tsx
<FruitBasket
fruitType="apple"
maxFruit={5}
disabled
fruit={[
  'red delicious',
  'granny smith'
]}
>
```

Now imagine we were to take those props and put them into an object. It would look something like this:

```ts
const props = {
  fruitType: "apple",
  maxFruit: 5,
  disabled: true,
  fruit: ["red delicious", "granny smith"],
};
```

This is the object that will be passed in to our FruitBasket component as props. If we were to annotate that object, we would need an interface that looks like this:

```ts
interface FruitBasketProps {
  // This could be a string, but I'm using a union of strings
  // to constrain what the possible values are.
  fruitType: "apple" | "orange" | "banana";
  maxFruit: number;
  disabled?: boolean;
  fruit: string[];
}

const FruitBasket = (props: FruitBasketProps) => {
  // ...
};
```

There's nothing special about it. TypeScript is smart enough to translate the props in the JSX into a props object and check that object against the interface we created. This includes the union type, the array, and the function.

There's one more thing we can do though. By making our props `ReadOnly`, we can ensure we don't mutate our props. This is something that isn't commonly done, since most people know instinctively to not mutate props, but having TypeScript enforce it can help when we make mistakes.

```ts
const FruitBasket = (props: ReadOnly<FruitBasketProps>) => {
  // ...
};
```

Notice that `disabled` is an optional property in our interface. This is important, since we often want to omit certain props from the JSX for brevity. It would be annoying if we always had to write:

```tsx
<FruitBasket disabled={false} />
```

Since the absence of a prop is considered `undefined`, we can treat it as falsy in our component code and safely mark it as optional. We could do the same thing for non-boolean types by assigning a default value in our function parameters.

```ts
interface FruitBasketProps {
  // This could be a string, but I'm using a union of strings
  // to constrain what the possible values are.
  fruitType?: "apple" | "orange" | "banana";
  maxFruit: number;
  disabled?: boolean;
  fruit: string[];
  addFruit: (fruitName: string) => void;
}

const FruitBasket = ({
  fruitType = "apple",
  ...props
}: FruitBasketProps) => {
  // ...
};
```

TypeScript will even recognize indexable types, allowing us to put whatever props we want on a component.

```tsx
interface ListerProps {
  [key: string]: string;
}

const Lister = (props) => {
  return (
    <ul>
      {/* List all of the keys and values in the props object. */}
      {Object.entries(props).map(([key, value]) => (
        <li key={key}>
          <strong>{key}</strong>: {value}
        </li>
      ))}
    </ul>
  );
};

<Lister item1="Hello" item2="World" />;
```

It's possible to use `any`, `unknown`, `object`, or `Function` in our props interfaces as well, and TypeScript will allow it. However, that runs the risk to introducing runtime type errors. As usual with TypeScript, it's best to be as specific as possible as you are writing your type annotations for your components.
