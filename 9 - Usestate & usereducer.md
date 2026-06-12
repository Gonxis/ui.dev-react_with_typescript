### useState & useReducer

There are two hooks for storing state: `useState` and `useReducer`.


#### useState

`useState` is simple. The state value's type is the type of whatever you pass in to the `useState` function.

```ts
const [stringState, setStringState] = useState("Hello!");
const [numberState, setNumberState] = useState(5);
```

Sometimes you need to tell `useState` exactly what type your state should be, such as when you don't pass in an initial value, or you want to use a union type. `useState` is a generic function, so we can pass the type in before we call the function.

```ts
// This can only be undefined
const [uninitializedState, setUninitializedState] = useState();

// This can be a string or undefined
const [stringState, setStringState] = useState<
  string | undefined
>();

// Its more common to initialize with null
const [maybeNumberState, setMaybeNumberState] = useState<
  string | null
>(null);
```

Of course, you can put whatever kind of thing you want in there - arrays, functions, classes - you name it. Just make sure you pass a type into the generic function.

When working with component composition, it's common to pass the state and state setter function as props. If we only ever intend to pass a literal value to our state setter function, then annotating that function is easy - pass in value as a parameter; return `void`;

```ts
interface UpstreamComponentProps {
  stringState: string;
  setStringState: (newValue: string) => void;
}
```

This gets a bit tricker if we need to use an update function with our state setter. In this case, it's probably easier to use the built-in type which React uses.

In VS Code, we can hover over `setStringState` to see that its type is `Dispatch<SetStateAction<string>>` (replacing `string` with whatever the actual type is). Both `Dispatch` and `SetStateAction` are built into React's type declarations, so we can use them in our props interface.

```ts
interface UpstreamComponentProps {
  stringState: string;
  setStringState: Dispatch<SetStateAction<string>>;
  maybeNumberState: number | null;
  setMaybeNumberState: Dispatch<SetStateAction<number | null>>;
}
```


#### useReducer

If you need a bit more control over how your state is updated, or if you have several pieces of state that update in concert, `useReducer` can really come in handy.

Quick recap on reducers. We have state, which is usually an object (although it can be any type). `useReducer` gives us our state and a `dispatch` function. We call `dispatch` with an action, which is also usually an object. Dispatch then calls our reducer function with our current state and our action. The results of that reducer function become our new state.

That means we'll need to create three things to use `useReducer`:

* A reducer function
* A state type
* An action type, which is a union of several types representing each of the possible actions.

For this example, we'll create a shopping list that separates our grocery items by category. Our state will be an array of objects that conform to the ShoppingListItem interface, so lets create that.

```ts
type Category = "Bread" | "Fruit" | "Vegetable" | "Meat" | "Milk";

interface ShoppingListItem {
  id: string;
  title: string;
  completed: boolean;
  category: Category;
}

type ShoppingListState = ShoppingListItem[];
```

Next, we'll create the interfaces for our actions. Each action should include a property which identifies the action, such as 'add', 'edit', 'delete', or 'complete'. Actions should also have any additional data that is needed to perform the action, such as the ID of the ShoppingListItem that we are completing.

Once we've created all of our action interfaces, we can create a discriminating union of them, which makes it easy to know what type of action we are working with just by checking the type property.

```ts
interface AddAction {
  type: "add";
  category: Category;
}
interface EditAction {
  type: "edit";
  id: string;
  title: string;
}
interface DeleteAction {
  type: "delete";
  id: string;
}
interface CompleteAction {
  type: "complete";
  id: string;
}
export type ShoppingListAction =
  | AddAction
  | EditAction
  | DeleteAction
  | CompleteAction;
```

Finally, we can create our shopping list reducer function. Remember, this reducer takes our state interface and a union of actions as its parameters. We'll use a `switch` statement to determine what action was passed in, and modify our state array accordingly.

```ts
function shoppingReducer(
  state: ShoppingListState,
  action: ShoppingListAction,
): ShoppingListState {
  switch (action.type) {
    case "add":
      return state.concat({
        // Good enough random ID generator.
        id: `${Math.random()}-${Date.now()}`,
        title: "",
        category: action.category,
        completed: false,
      });
    case "complete":
      return state.map((item) => {
        if (item.id === action.id) {
          return { ...item, completed: true };
        }
        return item;
      });
    case "delete":
      return state.filter((item) => item.id !== action.id);
    case "edit":
      return state.map((item) => {
        if (item.id === action.id) {
          return { ...item, title: action.title };
        }
        return item;
      });
    default:
      return state;
  }
}
```

Now here comes the cool part. The `useReducer` type definition has a few function overloads, but the most common takes two parameters: our reducer function, and the initial state.

```ts
const [state, dispatch] = useReducer(shoppingReducer, []);
```

Wait, don't we need to annotate useReducer at all? In this case, we don't. TypeScript has pulled the type of our state and actions out of the parameters of `shoppingReducer` and used those as the types of `state` and `dispatch`.

Actually, in the case of `dispatch`, the type is `Dispatch<ShoppingListAction>` which should look familiar. This is the same shape as the state setter function for `useState`, except it uses our action instead of `SetStateAction`. That makes it really easy to pass our `dispatch` function to other components using props or context.

Now, if we use our `dispatch` function incorrectly, TypeScript can give us a warning.

```ts
dispatch({ type: "explode" });
// Type Error: Type '"explode"' is not assignable to
//   type '"add" | "edit" | "delete" | "complete"'
```
