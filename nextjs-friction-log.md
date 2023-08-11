# NextJS Friction Log

**Author**: Josh Allmann ([email](mailto:josh_at_transfix_dot_ai)) ([web](https://transfix.ai))

**Date**: 27 July 2023 (Updated 11 Aug 2023)

### Goal

Build a web page that has a search box, and displays search results. Stretch goals: shareable search page via updating query parameters, and search as you type

### About Me

Video / systems dev, but have done my fair share of frontend / JS / React. Have scars from dealing with react state.

#### Why Pick NextJS?
Liked the idea of backend + frontend in a unified framework. That seems like less moving parts operationally? There are big promises (SSR, automatic chunking etc) but I confess I don't really care about those at this point; need something that works and can deliver what I need in a tight package without pulling in too much other stuff, eg standing up a separate backend. Moreover, all the cool kids seem to be using it so there must be something there.

This would be my second nextjs project, and the first with a backend component. The [first](https://metadata.transfix.ai) was entirely statically complied to a single page which mostly worked out fine. Was able to get a WASM module working with that which I was pleased with - see the [Github](https://github.com/j0sh/metadata-reader) repo for details.

### Setup
nextjs in "app" mode with tailwindcss. exact version when I started was 13.4.5

Next.config.js is the default -

```
/** @type {import('next').NextConfig} */
const nextConfig = {}

module.exports = nextConfig
```


### Notes

#### Type Checking

During the [metadata project](https://metadata.transfix.ai), I was not pleased to see that there was no option for type checking (compared to, say, CRA (create-react-app)) found some post somewhere suggesting this in `package.json` which is what I have been using:

```
    "dev:ts": "yarn dev & yarn ts:watch",
    "ts": "tsc --noEmit --incremental",
    "ts:watch": "yarn ts --watch"
```

#### Docs: Pages vs Apps

Most search terms return google results for "pages". most of the information seems useful, but 1) my apps are using the "apps" scheme, and 2) "pages" seems deprecated, and I don't really want to use deprecated approach?

#### Server Actions

Server Actions are alpha. What should I use for a non-alpha alternative? I do not want anything experimental; only things that are stable. managed to figure out what seems to be a more traditional frontend / backend division of responsibility with an API defined via routes.ts . This wasn't really documented anywhere but I have the experience to piece that together.

Another sense about server actions after, reading about it: feels too magical. Not really a good feeling; "what's the catch?"

Sure enough I found myself having to annotate my main page with "use client" as soon as the only interactive component on the page (input box) got an onchange event. It might also have been when trying to introduce hooks to the page; don't really remember.

Perfectly possible I am "holding it wrong" but docs did not really indicate what a *better* way would be. Literally all I have at this point is a `page.tsx` with an input text box.

saw some stackoverflow comment about client components not supporting async very well which was concerning (can't find it anymore)

#### Forms clear after a reload-on-save
The relaod-on-save feature is super nice, but it is quite annoying to have to re-enter form information every time I save the code. (This form is not actually React state; it's just a plain JSX form with some onsubmit functionality.) Create-react-app does not clear forms, and it makes for a much smoother experience during development.

#### General feeling of slowness

Typing into forms, re-renders, etc all feel much slower for some reason, when compared to the equivalent in create-react-app. Don't have objective metrics for this point, but it *feels* sluggish.

#### 'window' object is not defined

Got a message saying `'window' object is not defined`. What? okay, it's trying to render on server even with "use client" mode set. how tf is this supposed to work? I think [this post](https://blog.sethcorker.com/question/how-to-solve-referenceerror-next-js-window-is-not-defined/) helped me work around the issue, but it still seems silly for a *client* component.

#### Weird TS error message about top-level awaits
Followed the instructions (set tsconfig.json target to 'es2017',  module was already 'esnext') - but still got errors about top-level await.


#### Updating NextJS leads to breakage

Noticed a thing on the error page saying that my nextjs was out of date. Did the recommended `yarn add` to upgrade to 13.4.9 and ... things completely broke harder, somehow. Unhelpful error message says someting with webpack, "Unhandled runtime error, TypeError: __webpack_require__.n is not a function"

Had a moment where commenting out everything and replacing with "hello, world" did not even work. Disabled the type checker and finally saw "hello, world"

### Conclusion

At this point I did not have the patience to continue anymore.

Ported things into create-react-app and a deno backend. That is not featureful or well-integrated but these tools mostly get out of my way, so I can bang my head on the product, rather than framework issues.
