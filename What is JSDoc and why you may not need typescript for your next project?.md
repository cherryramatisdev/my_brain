---
title: What is JSDoc and why you may not need typescript for your next project?
description: In this quick article we'll discuss why major projects are dropping typescript in favor of javascript without bundling and how to use JSDOC for full type checking + integration with typescript via .d.ts files.
tags:
  - programming
  - beginners
  - javascript
cover_image: 
canonical_url: 
published: false
---
It has been a couple of weeks since I started testing out this technology, JSDoc, for maintaining some JavaScript codebases. This has been especially important after some major events that occurred in the last few months.

- [Sveltejs move from Typescript to Javascript with JSDOC](https://devclass.com/2023/05/11/typescript-is-not-worth-it-for-developing-libraries-says-svelte-author-as-team-switches-to-javascript-and-jsdoc/)
- [Turbo 8 remove typescript **without using JSDOC**](https://github.com/hotwired/turbo/pull/971)

Those events generated a lot of buzz through the tech community primarily with the argument that "removing typescript  is the same as removing types" and that typescript is the only way to write safe javascript. In this article we'll discuss further what is JSDOC, what is the purpose of typescript and why you may not need typescript!

## Table of contents

- [What is JSDOC anyway?](#what-is-jsdoc-anyway?)
- [What is typescript?](#what-is-typescript?)
- [Which problems typescript solves?](which-problems-typescript-solves?)
- [Which problems JSDOC can solve by itself?](#which-problems-jsdoc-can-solve-by-itself?)
- [How to type check code using typescript compiler in a Javascript + JSDOC codebase](#how-to-type-check-code-using-typescript-compiler-in-a-javascript-+-jsdoc-codebase)
- [Conclusion](#conclusion)

## What is JSDOC anyway?

JSDOC is a predefined method of documenting code for javascript ecosystem created in 1999 that works similar to libraries for other languages such as: [Javadoc for java](https://docs.oracle.com/javase/8/docs/technotes/tools/windows/javadoc.html), [YARD for ruby](https://yardoc.org), etc..

Practically speaking jsdoc is a way to write comments that the LSPs or any other tool can parse and provide information for the developer, basically a markup language (you know, like HTML) for coding samples, for example:

```javascript
/**
 * The `foo` function append two strings and return the appended return.
 * @param {string} a
 * @param {string} b
 * @returns {string}
 * @throws {Error} If the first string is empty
*/
function foo(a, b) {
    // Code...
}
```

That produces the following output using the [tsserver LSP](https://github.com/typescript-language-server/typescript-language-server):

![[Pasted image 20240122101811.png]]

With that markup language, we can define text-related information such as function descriptions, error cases (throws), and language-specific details like type signatures for parameters and return values that *populate TypeScript's own signatures*, as you can see in the screenshot.

There is a lot of specific symbols presented on the JSDOC specification that can be found here: https://jsdoc.app

## What is TypeScript?

> Disclaimer: This article will **not** focus on the specific details about type level typescript, if your demand involves that level of type checking you're probably fine using typescript.

TypeScript is a linter and compiler/transpiler that provides a specific syntax for defining types and other specific language constructs such as interfaces, abstract classes, decorators, etc.

At the time it was created, TypeScript surely changed how everyone writes JavaScript. It not only introduced a new language but also competed with Babel in providing a way to support new language features without waiting for official [ECMA](https://en.wikipedia.org/wiki/ECMAScript) support.

Besides the features TypeScript itself proposed, the most important thing it brought to the community was the ability to create cool features around this compiler that enhance the developer experience and productivity. Tools like [tsserver](https://github.com/typescript-language-server/typescript-language-server), [pretty ts errors](https://github.com/yoavbls/pretty-ts-errors), and many others are actively improving the ecosystem for both JavaScript and TypeScript writers.

## Which problems typescript solves?

Most people think that typescript only brought type checking for our codebases, but it's not the only thing it shines:

1. *Stop the run with ECMAScript Support*: With TypeScript, we have the opportunity to use cutting-edge JavaScript features without worrying about browser support. Therefore, features like abstract classes, interfaces, and decorators become accessible to the developer.
2. *Behold! The types are coming*: Together with providing support for modern JavaScript features from the community, TypeScript also provides a [Turing complete](https://en.wikipedia.org/wiki/Turing_completeness) type system that allows both simple checking and complex type logic, mostly used by library authors.

It's important to point out that TypeScript really shines for library developers in the JavaScript ecosystem. If you want to provide a great experience for your users when installing your library, then TypeScript is probably the best solution for the case.

> For those who want to improve their type level skills, I highly recommend this repository for learning by doing: https://github.com/type-challenges/type-challenges.

Another caveat is that TypeScript is not the only way to transpile JavaScript accessible for older versions. If you don't need specific features of TypeScript, it's possible to use alternatives that don't perform type checking.

- [SWC](https://swc.rs)
- [Parcel](https://parceljs.org)
- [Esbuild](https://esbuild.github.io)

## Which problems JSDOC can solve by itself?

Since JSDoc is just a markup language built on top of javascript comments the solvable problems are very narrowed to that aspect, so it's mainly used for:

- Documenting functions or classes.
- Infering simple types for parameters, returns, class variables, etc..
- Interacting with TypeScript complex type system via `.d.ts` files
- Utilizing the potential of already existing tools like the TypeScript compiler and LSP.

Most people default to the TypeScript language as the only way to interact with the compiler and LSP. However, JavaScript using JSDoc is a perfectly fine way to maintain highly secure software without dealing with complex bundle times (if you want to bring a bundle, simply select one from the list presented in the previous topic).

## How to type check code using TypeScript compiler in a Javascript + JSDOC codebase

Since TypeScript is just a linter and it has support for JSDOC types, we can even get the full power of `tsc` on our CI pipelines without worrying about bundling altogether.

First let's define a fairly complex type interacting with `.d.ts` file for extra TypeScript magic **without bundling**.

Setup a simple project with `npm init -y` and write the following `index.js` file:

```javascript
/**
 * foo....
 * @param {import('./types.d.ts').ComplexType<number, { message: string }>} complexType
 * @returns {string}
*/
function foo(complexType) {
    if (complexType.type === 'err') {
        console.log(complexType.data.message)
        return
    }

    return complexType.data
}
```

See how the `import` keyword is being used to import a type file? You can think of this approach as the `.h` and `.c` files from the C language.

> Disclaimer: This JSDOC feature was implemented by the TypeScript team itself together with a lot more possibilities: https://www.typescriptlang.org/docs/handbook/jsdoc-supported-types.html

Now to define the `types.d.ts` file, I tried to implement a fairly complex type simulating `Result` from Rust just to show that we can use full typescript potential with this method:

```typescript
type Ok<T> = { type: 'success', data: T }
type Err<T> = { type: 'err', data: T }

export type ComplexType<V, E> = Ok<V> | Err<E>
```

After defining our sample code, we can use the full power of the typescript linter without using it's compiler, let's define the following `tsconfig.json` config file for that:

```json
{
    "compilerOptions": {
        "checkJs": true
    },
    "exclude": ["node_modules"]
}
```

And we can finally run our command to check the entire code and see a type error (that I hope you already saw before 👀):

![[Pasted image 20240122174717.png]]

## Conclusion

This article was my exploration around the current news of big projects dropping TypeScript from their codebase while still maintaining type checking experience (only svelte) and I hope to have successfully shown my surprise of how much it's possible to achieve without using bundling by default.

I want to point out again that using TypeScript is not a problem at all, and that without this language we wouldn't get amazing projects such as the tsserver LSP.

May the force be with you! 🍒