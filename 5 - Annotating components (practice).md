### (Practice) Annotating Components

Below is a CodeSandbox link which has some TypeScript code. Your job is to add types to the class components so all of the red squiggles indicating errors go away.

Good luck!

[Class Component Annotation CodeSandbox](https://codesandbox.io/s/practice-typescript-component-annotation-5q4pc)


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
 * This app has two class components. Your job is
 * to add the necessary props and state interfaces and
 * annotate the two components. You'll know you are done
 * when the red squiggles are gone.
 *
 * There is also an `unknown` type used in the `getPokemon`
 * method - you should replace that with an interface.
 *
 * You might be tempted to create an interface that perfectly
 * models the results of the API call - don't. It's best
 * to only model the properties which are actually used in
 * the component.
 *
 * Don't change the implementation of the components;
 * only add type annotations.
 */

class Pokemon extends React.Component {
  state = { pokemon: null };

  getPokemon = async () => {
    const response = await fetch(this.props.url);
    const data: unknown = await response.json();
    this.setState({ pokemon: data });
  };
  componentDidMount() {
    this.getPokemon();
  }
  render() {
    if (!this.state.pokemon) return null;
    return (
      <li className="pokemon">
        <img
          alt={this.state.pokemon.name}
          src={this.state.pokemon.sprites.other.dream_world.front_default}
        />
        <strong>{this.state.pokemon.name}</strong>
      </li>
    );
  }
}

const PAGE_SIZE = 50;
export default class PokeList extends React.Component {
  state = { page_num: 0, pokemon_list: null };
  getPokemonList = async () => {
    const response = await fetch(
      `https://pokeapi.co/api/v2/pokemon?limit=${PAGE_SIZE}&offset=${
        this.state.page_num * PAGE_SIZE
      }`
    );
    const data: unknown = await response.json();
    this.setState({ pokemon_list: data.results });
  };
  componentDidMount() {
    this.getPokemonList();
  }
  componentDidUpdate(prevProps, prevState) {
    if (prevState.page_num !== this.state.page_num) {
      this.setState({ pokemon_list: null });
      this.getPokemonList();
    }
  }
  render() {
    return (
      <div>
        {this.state.page_num >= 1 && (
          <button
            onClick={() => {
              this.setState((state) => ({ page_num: state.page_num - 1 }));
            }}
          >
            Prev
          </button>
        )}
        <button
          onClick={() => {
            this.setState((state) => ({ page_num: state.page_num + 1 }));
          }}
        >
          Next
        </button>
        <ul>
          {this.state.pokemon_list?.map((pokemon) => (
            <Pokemon key={pokemon.url} url={pokemon.url} />
          ))}
        </ul>
      </div>
    );
  }
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


src/styles.css
```css
/* Your CSS goes here */
.pokemon {
  display: flex;
  align-items: center;
  border-bottom: 1px solid rgba(255, 255, 255, 0.5);
  padding: 1rem 0.5rem;
}
.pokemon img {
  max-height: 50px;
  margin-right: 0.5rem;
}
```


package.json
```json
{
  "name": "practice-typescript-component-annotation",
  "version": "1.0.0",
  "description": "",
  "keywords": [],
  "main": "src/index.tsx",
  "dependencies": {
    "@types/react": "17.0.0",
    "@types/react-dom": "17.0.0",
    "react": "17.0.1",
    "react-dom": "17.0.1",
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
    "strict": true,
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
