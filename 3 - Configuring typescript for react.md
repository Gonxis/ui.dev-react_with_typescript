### Configuring TypeScript for React

There are a few settings in our tsconfig.json we need to configure to use React TypeScript.


#### React Type Declarations

React doesn't ship with TypeScript declarations; we have to install them from DefinitelyTyped. All it takes is one little npm install.

```bash
npm install --save-dev @types/react @types/react-dom
```


#### JSX Transform

If we are using React 16.14 or React 17, we'll want to make sure we have TypeScript 4.1 or later installed. These versions of React and TypeScript introduced a [new JSX transform](https://reactjs.org/blog/2020/09/22/introducing-the-new-jsx-transform.html) which adds a few benefits:

* It lets you use JSX in your code without importing React
* It can decrease the bundle size in certain circumstances
* It future-proofs your app for new features that are coming in future React versions.

We can configure TypeScript to use these new JSX transforms by setting the `jsx` option in tsconfig.json. There are two options we can use:

* `react-jsx`, for compiling to production code
* `react-jsxdev`, for development mode

Since there are two different options depending on our environment, it might make sense to create two tsconfig.json files. We can have our development tsconfig.json file extend our production configuration, so we don't need to duplicate our settings.

```json
// tsconfig.dev.json
{
  "extends": "./tsconfig.json",
  "compilerOptions": {
    "jsx": "react-jsxdev"
  }
}
```

Then we put all of our other configuration in tsconfig.json, including using the production JSX transform.


#### Legacy JSX Transform

If you are using an older version of React, you can configure the legacy JSX transform by setting the `jsx` setting to `react`. This will convert JSX syntax into the appropriate `React.createElement` calls, but it requires that React is imported inside of every module.

```json
// tsconfig.json
{
  "compilerOptions": {
    "jsx": "react"
    // ...
  }
}
```

If we are using the legacy JSX transform, we can change the JSX factory function from `React.createElement` to something else. For example, if we are using Preact, we can set our JSX factory to the `h` function.

```json
// tsconfig.json
{
  "compilerOptions": {
    "jsx": "react",
    "jsxFactory": "h"
    // ...
  }
}
```

Just like with `React.createElement`, this factory function expects the `h` function to be imported in every module that uses JSX.

```ts
import { h } from "preact";

const HelloWorld = () => <div>Hello</div>;
```


#### Fragments

You might remember that `React.Fragment` is used to wrap adjacent elements. There is also a shorthand syntax which looks something like this:

```tsx
const Hello = () => {
  return (
    <>
      <h1>Hello</h1>
      <h2>World</h2>
    </>
  );
};
```

This shorthand needs to be transformed to use the correct Fragment component, which can also be configured using `jsxFragmentFactory`. Regardless of whether you are using the new or legacy transform, you'll likely want this setting to be `Fragment`.

```json
// tsconfig.json
{
  "compilerOptions": {
    "jsx": "react-jsx",
    "jsxFragmentFactory": "Fragment"
  }
}
```


#### .tsx Extension

The most important thing to remember when using React with TypeScript is that file extensions matter. TypeScript has some syntax which is incompatible with JSX syntax. To get around these conflicts, TypeScript only recognizes JSX inside of files that end with `.tsx`; it also disables the conflicting TypeScript syntax in those files.

If you ever find strange, nonsensical errors in your JSX code, double check your extension.
