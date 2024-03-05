---
title: How to manage HTTP requests on page load with elm
description: How to create a simple single page application with elm consuming a HTTP route
tags: 'elm,frontend'
cover_image: ''
canonical_url: null
published: true
---

> DISCLAIMER: I'll be assuming basic knowledge with elm language and elm architecture

I'm currently learning elm as my main frontend language to use alongside ruby on rails (tutorials about using both coming soon) and on this tutorial I'll try to explain how to manage a API call on the page load with json decoding because this was a little hard to find, so I hope to make it easy for you!

## Creating our application

To create this application we'll do the following:

```sh
mkdir project && elm init
```

The packages needed for this project will be the following:

```sh
elm install elm/json
elm install elm-community/json-extra
elm install elm/url
elm install elm/http
elm install elm/core
```

## The initial structure for the application

The first thing we'll be doing is defining the initial architecture for our SPA, we'll be using a `Browser.application` so we can use `init` as a function.

```elm
module Tutorial exposing (..)

import Browser
import Browser.Navigation
import Html exposing (text)
import Url exposing (Url)


type Status
    = Loading
    | Done
    | HttpError


type alias Model =
    { name : String
    , message : Status
    }


type Msg
    = None
    | DataFetched


initialModel : Model
initialModel =
    { name = ""
    , message = Loading
    }


init : flags -> Url -> Browser.Navigation.Key -> ( Model, Cmd Msg )
init _ _ _ =
    ( initialModel, Cmd.none )


view : Model -> Browser.Document Msg
view model =
    { title = "Test"
    , body = [ text model.name ]
    }


update : Msg -> Model -> ( Model, Cmd Msg )
update msg _ =
    case msg of
        None ->
            Debug.todo "TODO"

        DataFetched ->
            Debug.todo "TODO"


subscriptions : Model -> Sub Msg
subscriptions _ =
    Sub.none


main : Program () Model Msg
main =
    Browser.application
        { init = init
        , update = update
        , view = view
        , subscriptions = subscriptions
        , onUrlChange =
            \_ ->
                None
        , onUrlRequest =
            \_ ->
                None
        }
```

With this we define our initial messages, our initial model and a very simple layout just to see the information from our model when we perform the HTTP request.

## Define our fetcher and decoder

To perform the HTTP request we first need a function to register this request and then a decoder to transform this data into typed values so we can populate on our model.

We'll be using the famous "jsonplaceholder" API as an example for this tutorial.

```elm
type alias Todo =
    { title : String }

fetchData : (Result Http.Error Todo -> msg) -> Cmd msg
fetchData msg =
    Http.get
        { url = "https://jsonplaceholder.typicode.com/todos/1"
        , expect = Http.expectJson msg jsonDecoder
        }

jsonDecoder : Decoder Todo
jsonDecoder =
    Json.Decode.succeed Todo
        |> Json.Decode.Extra.andMap (Json.Decode.field "title" string)
```

The decoder works as a way to declarative define the API returning, so first we assume the api returned succesfully with the `Json.Decode.succeed Todo` and then we map through the return with `Json.Decode.Extra.andMap` with a function to define the specific field from the API.

> Tip: You can enter the jsonplaceholder link to check the return body.

## Let's hook all up

Now it's a matter of wrapping all the things on the `init` and `update` functions! let's do it


First we'll be extending one of our variant Msg to contain the http result

```elm
type Msg
    = None
    | DataFetched (Result Http.Error Todo)
```

And also update our `update` function to properly deal with the Result type and extracting the correct information to update our model

```elm
update : Msg -> Model -> ( Model, Cmd Msg )
update msg _ =
    case msg of
        None ->
            ( initialModel, Cmd.none )

        DataFetched result ->
            case result of
                Ok data ->
                    ( { name = data.title, message = Done }, Cmd.none )

                Err _ ->
                    ( { initialModel | message = HttpError }, Cmd.none )
```

And finally let's call our fetcher function on our `init`

```elm
init : flags -> Url -> Browser.Navigation.Key -> ( Model, Cmd Msg )
init _ _ _ =
    ( initialModel, fetchData DataFetched )
```

## Conclusion

I hope this serve as a quick and simple tutorial to perform HTTP requests on page load with elm, it's an amazing language that i'm planning to use it a lot more so I hope to bring more interesting content soon!