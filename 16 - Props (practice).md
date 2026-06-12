### (Practice) Props

Below is a CodeSandbox link which has some TypeScript code. Your job is to annotate the props of the various React components so the red squiggles indicating errors go away.

[Props Annotation CodeSandbox](https://codesandbox.io/s/practice-typescript-props-ysq4w)


public/index.html
```html
<!DOCTYPE html>
<html lang="en">

<head>
	<meta charset="utf-8">
	<meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
	<meta name="theme-color" content="#000000">
	<!--
      manifest.json provides metadata used when your web app is added to the
      homescreen on Android. See https://developers.google.com/web/fundamentals/engage-and-retain/web-app-manifest/
    -->
	<link rel="manifest" href="%PUBLIC_URL%/manifest.json">
	<link rel="shortcut icon" href="%PUBLIC_URL%/favicon.ico">
	<!--
      Notice the use of %PUBLIC_URL% in the tags above.
      It will be replaced with the URL of the `public` folder during the build.
      Only files inside the `public` folder can be referenced from the HTML.

      Unlike "/favicon.ico" or "favicon.ico", "%PUBLIC_URL%/favicon.ico" will
      work correctly both with client-side routing and a non-root public URL.
      Learn how to configure a non-root public URL by running `npm run build`.
    -->
	<title>React App</title>
</head>

<body>
	<noscript>
		You need to enable JavaScript to run this app.
	</noscript>
	<div id="root"></div>
	<!--
      This HTML file is a template.
      If you open it directly in the browser, you will see an empty page.

      You can add webfonts, meta tags, or analytics to this file.
      The build step will place the bundled scripts into the <body> tag.

      To begin the development, run `npm start` or `yarn start`.
      To create a production bundle, use `npm run build` or `yarn build`.
    -->
</body>

</html>
```


src/App.tsx
```tsx
import * as React from "react";

/**
 * Props Cardio
 *
 * This page has several components that accept and consume props.
 * Your job is to add the neccessary type annotations that represent
 * the props of the components.
 *
 * Comments above each section of components give hints as to how to
 * properly add types to the components.
 *
 * You'll know that you are done when all of the red squiggles are gone.
 */

// HTML Element Mirroring
type VariantColors = "primary" | "secondary";
const Button = ({
  variantColor,
  style,
  ...props
}: {
  variantColor: VariantColors;
}) => {
  const innerStyle = {
    ...style,
    color: "white",
    backgroundColor:
      variantColor === "primary" ? "rgb(26,100,255)" : "rgb(170,170,170)"
  };
  return <button style={innerStyle} {...props} />;
};

// Basic Props
const Counter = ({ count }) => {
  return <h1>Count: {count}</h1>;
};
const CounterButtons = ({ setCount }) => {
  return (
    <div>
      <Button
        variantColor="primary"
        onClick={() => setCount((count) => count - 1)}
      >
        -
      </Button>
      <Button onClick={() => setCount((count) => count + 1)}>+</Button>
    </div>
  );
};

// Children Prop & Style Prop
const Tooltip = ({ children, contents, style }) => {
  const [hovered, setHovered] = React.useState(false);
  return (
    <div
      style={{ position: "relative" }}
      onMouseEnter={() => setHovered(true)}
      onMouseLeave={() => setHovered(false)}
    >
      <div
        style={{
          ...style,
          display: hovered ? "block" : "none",
          position: "absolute",
          top: "100%"
        }}
      >
        {contents}
      </div>
      {children}
    </div>
  );
};

// Render Props
const MousePosition = ({ children }) => {
  const [mousePosition, setMousePosition] = React.useState({ x: 0, y: 0 });

  React.useEffect(() => {
    function onMove(event: MouseEvent) {
      setMousePosition({ x: event.clientX, y: event.clientY });
    }
    document.addEventListener("mousemove", onMove);
    return () => {
      document.removeEventListener("mousemove", onMove);
    };
  }, []);

  return children(mousePosition);
};

// Don't change anything below this point
// Any of the red squiggles down here will
// go away once you fix the issues above.
export default function App() {
  const [count, setCount] = React.useState(0);
  return (
    <div>
      <div>See the instructions in the code editor.</div>
      <Tooltip contents="Counter Tooltip">
        <Counter count={count} />
      </Tooltip>
      <CounterButtons setCount={setCount} />
      <MousePosition>
        {({ x, y }) => (
          <Tooltip
            style={{
              border: "solid 1px rgba(255,255,255,0.2)",
              padding: "6px",
              borderRadius: "3px"
            }}
            contents={
              <span style={{ color: "red" }}>Mouse Position Tooltip</span>
            }
          >
            <h2>
              Mouse Position: {x}, {y}
            </h2>
          </Tooltip>
        )}
      </MousePosition>
    </div>
  );
}
```


src/index.tsx
```tsx
import React from "react"
import ReactDOM from "react-dom"
import './theme.css'
import './styles.css'
import App from "./App"

function ColorfulBorder() {
  return (
    <ul className='border-container'>
      <li className='border-item' style={{ background: 'var(--red)' }} />
      <li className='border-item' style={{ background: 'var(--blue)' }} />
      <li className='border-item' style={{ background: 'var(--pink)' }} />
      <li className='border-item' style={{ background: 'var(--yellow)' }} />
      <li className='border-item' style={{ background: 'var(--aqua)' }} />
    </ul>
  )
}

const rootElement = document.getElementById("root");
ReactDOM.render(
  <React.StrictMode>
    <ColorfulBorder />
    <div className='container'>
      <App />
    </div>
  </React.StrictMode>,
  rootElement
)
```


src/theme.css
```css
/* Ignore this file. You can put your CSS in the styles.css file */

@import url("https://ui.dev/font");

:root {
  --black: #000;
  --white: #fff;
  --red: #f32827;
  --purple: #a42ce9;
  --blue: #2d7fea;
  --yellow: #f4f73e;
  --pink: #eb30c1;
  --gold: #ffd500;
  --aqua: #2febd2;
  --gray: #282c35;
}

*,
*:before,
*:after {
  box-sizing: inherit;
}

html {
  font-family: proxima-nova, -apple-system, BlinkMacSystemFont, Segoe UI, Roboto, Oxygen-Sans, Ubuntu, Cantarell, Helvetica Neue, sans-serif;
  text-rendering: optimizeLegibility;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  box-sizing: border-box;
  font-size: 18px;
}

body {
  margin: 0;
  padding: 0;
  min-height: 100vh;
  background: var(--black);
  color: var(--white);
}

.container {
  margin: 0 auto;
  max-width: 1100px;
  padding: 50px;
}

.border-container {
  padding: 0;
  margin: 0;
  display: flex;
}

.border-item {
  width: 20vw;
  height: 12px;
  list-style-type: none;
}

a {
  color: var(--gold);
  font-weight: 600;
}
```


package.json
```json
{
  "name": "practice-typescript-props",
  "version": "1.0.0",
  "description": "",
  "keywords": [],
  "main": "src/index.tsx",
  "dependencies": {
    "@types/react": "17.0.0",
    "@types/react-dom": "17.0.0",
    "react": "16.13.1",
    "react-dom": "16.13.1",
    "react-scripts": "3.4.1"
  },
  "devDependencies": {
    "typescript": "3.3.3"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  },
  "browserslist": [
    ">0.2%",
    "not dead",
    "not ie <= 11",
    "not op_mini all"
  ]
}
```

tsconfig.json
```json
{
  "compilerOptions": {
    "outDir": "build/dist",
    "module": "esnext",
    "target": "es2019",
    "lib": [
      "esnext",
      "dom"
    ],
    "sourceMap": true,
    "allowJs": true,
    "jsx": "react",
    "moduleResolution": "node",
    "rootDir": "src",
    "forceConsistentCasingInFileNames": true,
    "noImplicitReturns": true,
    "noImplicitThis": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "suppressImplicitAnyIndexErrors": true,
    "noUnusedLocals": true,
    "esModuleInterop": true,
    "isolatedModules": true
  }
}
```
