### useEffect, useMemo, useCallback

These next three hooks are relatively similar, and fairly simple, so this lesson should go quick.


#### useEffect

Lets take a look at the type definition for `useEffect`.

```ts
function useEffect(
  effect: EffectCallback,
  deps?: DependencyList,
): void;

type DependencyList = ReadonlyArray<any>;
type EffectCallback = () => void | (() => void | undefined);
```

This type definition tells us three things:

* `useEffect` takes a function, called EffectCallback, as it's first parameter.
* EffectCallback has to return either `void`, or a cleanup function. That cleanup function has to return `void`.
* `useEffect` can optionally take a list of dependencies, which is just an array of `any`.

The only thing to keep in mind when using `useEffect` in TypeScript is that the EffectCallback has to return `void`, which means it returns nothing, or a cleanup function. If we do return a cleanup function, it can't return anything either.

This only really becomes a problem if we are using an implicit return with an arrow function, like this:

```ts
useEffect(() => fetch("https://example.org", { method: "POST" }));
// Type Error: Type 'Promise<Response>' is not assignable to type 'void | (() => void | undefined)'
```

Usually it's best to use the non-implicitly returning form of arrow functions with `useEffect`.

```ts
useEffect(() => {
  fetch("https://example.org", { method: "POST" });
});
// No problem!
```

It's also worth noting that the seldom-used `useLayoutEffect` hook has exactly the same type definition as `useEffect`.


#### useMemo

Since it's short, we'll also look at the type definition for `useMemo`:

```ts
function useMemo<T>(factory: () => T, deps: DependencyList): T;
```

`useMemo` is a generic function. We pass it a function, and whatever that function returns is the return value of `useMemo`. I've never had a situation where TypeScript didn't properly infer the value of `T` based on what I passed in, so you'll likely never have to assign a type to `T`

The only thing to note here is that the dependency list for `useMemo` is not optional - you are required to provide one. Otherwise, what's the point of using `useMemo`? Without a dependency list, the value returned from `useMemo` will be referentially different between renders.


#### useCallback

`useCallback` is just like `useMemo`, except it only memoizes functions for us. Here's the type definition for `useCallback`:

```ts
function useCallback<T extends (...args: any[]) => any>(
  callback: T,
  deps: DependencyList,
): T;
```

This is mostly the same as `useMemo` except the generic type that represents the return value is constrained to be a function. That's because `useCallback` is specifically used for memoizing functions, so naturally we have to pass a function in.

Like `useMemo`, we also must provide a dependency list. Otherwise, we'll get a warning. It's just TypeScripts way of trying to help you out.
