### (Bonus) PropTypes

The [PropTypes](https://reactjs.org/docs/typechecking-with-proptypes.html) package lets you assign types to the props that are passed into your components _without_ using TypeScript. In fact, you might have spent this entire section wondering why we need to add type definitions to our component's props when PropTypes can already give us type checking.

TypeScript is a static type checker. It looks at all of your code to make sure the types of your properties, variables, and parameters all match up. Since it checks your code without running it, it can give you hints and warnings in your IDE which can help you as you write your code.

PropTypes, on the other hand, does _runtime_ checking of the types of your props. In development mode, PropTypes prints a warning to the console if you use the wrong type for one of your props. It can't help you as you write your code, though; only when you run it.

Using both of these tools together can be helpful. For example, if you are writing a library, it's possible that users of your library aren't using TypeScript. In that case, assigning PropTypes to your components can help them catch bugs.

PropTypes are assigned as a static property to class components, or as a property directly on function components. We'll use the `FruitBasketProps` to demonstrate using TypeScript and PropTypes together.

```tsx
import { Component, FC } from "react";
import PropTypes from "prop-types";

interface FruitBasketProps {
  fruitType: "apple" | "orange" | "banana";
  maxFruit: number;
  disabled?: boolean;
  fruit: string[];
  addFruit: (fruitName: string) => void;
}

const FruitBasketPropTypes = {
  fruitType: PropTypes.oneOf(["apple", "orange", "banana"])
    .isRequired,
  maxFruit: PropTypes.number.isRequired,
  disabled: PropTypes.bool,
  fruit: PropTypes.arrayOf(PropTypes.string),
  // Notice it can't check for the presence of parameters like TypeScript can.
  addFruit: PropTypes.func,
};
const FruitBasket: React.FC<FruitBasketProps> = (props) => {
  // ...
};

FruitBasket.propTypes = FruitBasketPropTypes;

class FruitBasket extends Component<FruitBasketProps> {
  static propTypes = FruitBasketPropTypes;
  // ...
}
```

Using PropTypes isn't as important when we're working in TypeScript, since TypeScript can validate our props as we write our code using more sophisticated type checking. Still, you might find it helpful to include PropTypes, especially if JavaScript users are consuming your components.
