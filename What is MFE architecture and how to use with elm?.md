---
tags:
  - personal
  - article
---

...wip

## Table of contents

- [What is this microfrontend thingy?](#what-is-this-microfrontend-thingy)
- [How the communication work?](#how-the-communication-work)
- [What is module federation and why it matters?](#what-is-module-federation-and-why-it-matters)
- [How to use it with elm as a remote](#how-to-use-it-with-elm-as-a-remote)
- [How to use it with elm as a host](#how-to-use-it-with-elm-as-a-host)
- [How about my styles?](#how-about-my-styles)
- [Conclusion](#conclusion)

> TODO: i want to put more detail here, maybe granularize more the topics? :think:

## What is this microfrontend thingy?

Microfrontends are an architecture for managing independent and composable frontends,
giving advantages like independent deploys, independent choice of language or framework,
full control of a specific domain within the application,
etc...

![what is microfrontend](2556286.png)

> TODO: talvez pros e contras numa tabela ou lista aqui?

## How the communication work?

The communication between microfrontends happen through an compiled file
conventionally called `remoteEntry.js`, this file contains all the details about
how to run this specific component including details about the runtime and shared
libraries

Normally we use bundlers like vite or webpack to bundle all this project and reference the
correct components under the `remoteEntry.js` file.

Complementing the architecture we have host projects that consume these remote entry files
and render it inside the current UI like using a plain regular component.

> TODO: organizar melhor isso aqui, as ideias ta ai mas ficou baguncado

## What is module federation and why it matters?

## How to use it with elm as a remote

To use elm as a remmote application we'll opt in for using `vite` as our bundler
(mostly because it's simpler to configure and often faster), we'll use a vite plugin for elm
alongside the standard plugin for module federation.

Firsts things first let's create a elm project:

```sh
$ mkdir remote_elm
$ cd remote_elm
$ elm init
```

```sh
$ tree
.
├── elm.json
└── src

2 directories, 1 file
```

After that let's create a simple counter application within a `src/Main.elm` file:

```elm
module Main exposing (..)

import Browser
import Html exposing (Html, button, div, text)
import Html.Events exposing (onClick)

main : Program () Model Msg
main =
    Browser.sandbox { init = init, update = update, view = view }

type alias Model = Int

init : Model
init =
    0

type Msg
    = Increment
    | Decrement

update : Msg -> Model -> Model
update msg model =
    case msg of
        Increment ->
            model + 1

        Decrement ->
            model - 1


view : Model -> Html Msg
view model =
    div []
        [ button [ onClick Decrement ] [ text "-" ]
        , div [] [ text (String.fromInt model) ]
        , button [ onClick Increment ] [ text "+" ]
        ]
```

With a basic project application at hand, let's setup vite to help build our project shall we?

```sh
$ npm init -y
$ npm install -DE elm vite vite-plugin-elm
```

> PS: This flag for the install command means "install into devDependencies and use exact version"

With this we should have a sample `package.json` looking like the following:

```json
{
  "name": "project-name",
  ....
  "devDependencies": {
    "elm": "^0.19.1-6",
    "vite": "6.0.11",
    "vite-plugin-elm": "3.0.1"
  }
}
```

After that we can generate a sample configuration for vite to run our project by editing a file called `vite.config.js`:

```javascript
import { defineConfig } from 'vite'
import elmPlugin from 'vite-plugin-elm'

export default defineConfig({
  plugins: [
    elmPlugin(),
  ],
  build: {
    modulePreload: false,
    target: 'esnext',
    minify: false,
    cssCodeSplit: false,
  },
})
```

After that we should be able to run `npx vite` and have our app running
through the dev server.

> TODO: print eh ok aqui?


Now that we have a basic webserver running with vite,
let's configure the module federation library to sync things up.

First we need to install the library:

```sh
$ npm install -DE @originjs/vite-plugin-federation
```

And after that we can modify our `vite.config.js`:

```diff
import { defineConfig } from 'vite'
import elmPlugin from 'vite-plugin-elm'
+import federation from '@originjs/vite-plugin-federation'

export default defineConfig({
  plugins: [
   elmPlugin(),
+  federation({
+     name: 'remoteElmApp',
+     filename: 'remoteEntry.js',
+     exposes: {
+       "./Main": "./src/Main.elm"
+     },
+   }),
  ],
+ base: '/',
  build: {
    modulePreload: false,
    target: 'esnext',
    minify: false,
    cssCodeSplit: false,
  },
+ server: {
+   port: 5001,
+ },
+ preview: {
+   port: 5001,
+ },
})
```

Here we're defining our elm app as a remote and exposing a particular component
(in our case the only one we have), we're also being explicit about the port usage
for server and preview (mainly because our host app will need to match the full URL)

> TODO: aqui eh legal ter um app host react pra consumir o remote elm ou partimos para o host elm e dps fechamos tudo?

## How to use it with elm as a host

??? (nao sei ainda kkk)

## How about my styles?

..

## Conclusion
