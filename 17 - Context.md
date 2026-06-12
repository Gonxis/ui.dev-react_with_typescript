### Context

React Context lets you pass values from a parent component to any of the children components. The way context is implemented makes it possible for the type definition of your context to be available wherever you consume it.

We create context using `React.createContext`. We have to pass in a default value. This is the value the context provides if it is accessed outside of a `<Context.Provider>` tree. The type of the entire context is inferred from the type of the default value.

Often, the default value is `null` or `undefined`. If we need to set an explicit type for our context, we can pass it in as a generic type.

```ts
import { createContext, Dispatch, SetStateAction } from "react";

interface ThemeModeInterface {
  mode: "dark" | "light";
  setMode: Dispatch<SetStateAction<"dark" | "light">>;
}

const ThemeModeContext = createContext<ThemeModeInterface | null>(
  null,
);
```

This has one substantial downside - we have to check if the value of our context is `null` every time we access it, regardless of whether we are in a Provider tree or not. However, we can get around these limitations with a bit of TypeScript slight of hand.

One option is to indicate that our default value is non-null with the non-null assertion operator.

```ts
const ThemeModeContext = createContext<ThemeModeInterface>(
  undefined!,
);
```

Another option is to assert that an empty object is actually our interface.

```ts
const ThemeModeContext = createContext({} as ThemeModeInterface);
```

In both of these cases, we are sacrificing type safety. If we try to access our context outside of the Provider tree, we'll likely run into runtime type errors.

One way to approach this is by creating a prescribed method of accessing that particular context that checks if the context value is actually set. Here's an example that uses the `useContext` hook.

```ts
export const useThemeMode = () => {
  const themeMode = useContext(ThemeModeContext);
  if (!themeMode?.mode)
    throw new Error(
      "The theme mode context was accessed outside of the provider tree",
    );
  return themeMode;
};
```

This is typically paired with a component that manages the context value and returns the context provider.

```tsx
export const ThemeModeProvider: FC = ({ children }) => {
  const [mode, setMode] = useState<"dark" | "light">("light");

  // Memoize the context value to avoid unnecessary re-renders
  const contextValue = useMemo(() => ({ mode, setMode }), [
    mode,
    setMode,
  ]);

  return (
    <ThemeModeContext.Provider value={contextValue}>
      {children}
    </ThemeModeContext.Provider>
  );
};
```

By only exporting the provider and the `useThemeMode` hook, we can ensure that an error will be thrown if the context is accessed outside of the provider tree.

If we access our context using the `useContext` hook or the `ThemeModeContext.Consumer` render prop, TypeScript will give us the type which we initialized our context with; this isn't the case with the static `contextType` property on class components, though - `this.context` will always need to be annotated directly on the class.
