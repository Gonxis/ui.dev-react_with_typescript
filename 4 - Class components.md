### Class Components

Before hooks were introduced, Class Components were the only way to maintain state in React Components. Even with hooks, Class Components are still necessary for things like Error Boundaries, and many codebases haven't yet migrated away from Class Components.

That shouldn't stop us from adding type safety to our codebase! Let's start by looking at a classic example of a React component that accepts an initial value for a counter, and has buttons to increase and decrease the number value.

```jsx
class Counter extends React.Component {
  constructor(props) {
    super(props);

    this.state = {
      count: props.defaultCount,
    };
  }
  increment = () => {
    this.setState(({ count }) => ({
      count: count - 1,
    }));
  };
  decrement = () => {
    this.setState(({ count }) => ({
      count: count + 1,
    }));
  };
  render() {
    return (
      <div>
        <h1>Count: {this.state.count}</h1>
        <button onClick={increment}>-</button>
        <button onClick={decrement}>+</button>
      </div>
    );
  }
}
```

Now take a look at where we call this component. Do you see <ins>anything wrong with this</ins>?

```jsx
ReactDOM.render(<Counter defaultCount="Hello" />, rootElement);
```

If we were to run this code, our initial count would be the string value "Hello". Clicking the plus button would change it to "Hello1", but clicking the minus button would change it to `NaN`. 😱 This is not what we want.

With TypeScript, we can avoid these kinds of mistakes altogether. The React.Component class is a [generic class](https://fireship.dev/c/typescript/generics-in-typescript), which means we can pass a TypeScript Interface as a type parameter which represents the props which the component accepts.

```ts
interface CounterProps {
  defaultCount: number;
}
class Counter extends React.Component<CounterProps> {
  constructor(props: CounterProps) {
    super(props);

    this.state = {
      count: props.defaultCount,
    };
  }
  // ...
}
```

Notice also that we also have to annotate the `props` parameter of our class constructor function.

If you have access to the ES2021 Class Field declarations feature, you can simply use the generic parameter, without needing to annotate `props` in the constructor function.

```tsx
interface CounterProps {
  defaultCount: number;
}
class Counter extends React.Component<CounterProps> {
  state = { count: this.props.defaultCount };
  // ...
}
```

Much nicer.

Adding the generic parameter for props allows us to constrain the props which we can pass into this component, which makes TypeScript warn us when we accidentally use our component incorrectly.

```tsx
ReactDOM.render(<Counter defaultCount="Hello" />, rootElement);
// Type Error: Type 'string' is not assignable to type 'number'.
```

We won't go too in-depth in this post, but there's an entire post just about [writing type annotations for props](https://fireship.dev/c/react-with-typescript/prop-types).


## Component State

We aren't done adding types to our component yet. TypeScript is still giving us a warning everywhere that we are accessing `this.state`. For example, on the line where we render our `count` value, TypeScript is warning us that `this.state` _doesn't have_ a `count` property.

```tsx
class Counter extends React.Component<CounterProps> {
  // ...
  render() {
    return (
      <div>
        <h1>Count: {this.state.count}</h1>
        {/* Type Error: Property 'count' does not exist on type 'Readonly<{}>'. */}
        {/* ... */}
      </div>
    );
  }
}
```

By default, `React.Component` uses a Readonly empty object as the type of `this.state`. We can easily override that by assigning the `state` class property an interface as its type.

```tsx
interface CounterState {
  count: number;
}
class Counter extends React.Component<CounterProps> {
  state: CounterState;
  render() {
    return (
      <div>
        <h1>Count: {this.state.count}</h1>
        {/* ... */}
      </div>
    );
  }
}
```

TypeScript will also automatically infer the type of `this.state` from the initial state value if we use ES2021 Class Field declarations.

```tsx
class Counter extends React.Component<CounterProps> {
  state = { count: this.props.defaultCount };
  // (property) count: number
  render() {
    return (
      <div>
        <h1>Count: {this.state.count}</h1>
        {/* ... */}
      </div>
    );
  }
}
```

We still have one more problem, this time with our `this.setState` calls.

```tsx
class Counter extends React.Component<CounterProps> {
  state:CounterState
  render() {
    return (
      <div>
      {/* ... */}
       <button
          onClick={() => {
            this.setState(({count}) => ({ count: count - 1 }))
            // Property 'count' does not
            //   exist on type 'Readonly<{}>'.
          }}
        >
          -
        </button>
    )
  }
}
```

Why is TypeScript warning us that the state parameter of `this.setState` doesn't have a `count` property? Don't we assign a type to `this.state` in our class?

Well, yes, we do. But `this.setState` uses a different mechanism than `this.state` for determining its parameter and return types. The `React.Component` constructor takes a second type parameter to represent our state interface.

```tsx
class Counter extends React.Component<CounterProps, CounterState> {
  // ...
}
```

In fact, adding the `CounterState` as the type for our component's state means we don't have to annotate the `this.state` class property; the generic handles it for us.


#### What about components with no props?

Sometimes you have a component with state, but no props. How do you annotate those classes?

By default, props has a default value of `{}`, so since we never access or use props, that works just fine as the type annotation. That lets us add the necessary type for the component state.

```tsx
class Counter extends React.Component<{}, CounterState> {
  render() {
    return (
      <div>
        <h1>Count: {this.state.count}</h1>
        {/* ... */}
      </div>
    );
  }
}
```

It's important to note that the generic parameters apply types to `this.props`, this.state, and `this.setState`, but not to the parameters of class lifecycle methods, like `shouldComponentUpdate` or `componentDidUpdate`. You need to annotate those parameters with the interfaces you used for your component.

```tsx
class Counter extends React.Component<CounterProps, CounterState> {
  // ...
  componentDidUpdate(
    prevProps: CounterProps,
    prevState: CounterState,
  ) {
    // Do some kind of check against this.props and this.state
  }
  // ...
}
```

Adding custom methods and properties to our component works exactly the same way as they do in regular classes. For class properties, they just need a type annotation; for class methods, so long as you add annotations to the method parameters, TypeScript will be able to type check how you use your class methods.

```tsx
class Counter extends React.Component<CounterProps, CounterState> {
  // ...
  increment = () => {
    this.setState(({ count }) => ({
      count: count - 1,
    }));
  };
  decrement = () => {
    this.setState(({ count }) => ({
      count: count + 1,
    }));
  };
  // ...
}
```
