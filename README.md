# CSS-in-TS research analysis - ⚠️ Draft - Work in progress

> _Last update: **Jan 2021**_

This document contains a thorough analysis of all the current **CSS-in-JS** solutions.  
The baseline reference we'll use for comparison is a **CSS Modules** approach.  
We're using **Next.js** as a SSR framework for building resources.  
Last important aspect is type-safety with full **TypeScript** support.

<br />

> ✋ Please checkout our [goals](#goals) & [disclaimer](#disclaimer) before jumping to conclusions.

<br />

## Table of contents

- [Motivation](#motivation)
- [Goals](#goals)
- [Disclaimer](#disclaimer)
- [Overview](#overview)
  - [CSS Modules](#css-modules)
  - [Styled JSX](#styled-jsx)
  - [Styled Components](#styled-components)
  - [Emotion](#emotion)
  - [Treat](#treat)
  - [TypeStyle](#typestyle)
  - [Fela](#fela)
  - [Stitches](#stitches)
  - [JSS](#jss)
  - [Goober](#goober)
- [Libraries not included](#libraries-not-included)
- [Running the examples](#running-the-examples)
- [Feedback and Suggestions](#feedback-and-suggestions)

<br />

## Motivation

The CSS language and CSS Modules approach have some limitations especially if you want to have solid and type-safe code. Some of these limitations have alterative solutions, others are just being "annoying" or "less than ideal":

1. **Styles cannot be co-located with components**  
  This can be frustrating when authoring many small components, but it's not a deal breaker. However, the experience of moving back-and-forth between the component and the .css file, searching for a given class name, and not being able to easily _"go to style definition"_ is a huge productivity bottleneck.

2. **Styling pseudos and media queries requires selector duplication**  
  Another frustrating fact at some point is the need to duplicate your class name when defining __pseudo classes and elements__, or __media queries__. You can overcome these limitations using a CSS preprocessor like __SASS, LESS or Stylus__, which all support the `&` parent selector, enabling __contextual styling__.
  
    ```css
    .button {}

    /* duplicated selector declaration for pseudo classes/elements */
    .button:hover {}
    .button::after {}

    @media (min-width: 640px) {
      /* duplicated selector declaration inside media queries */
      .button {}
    }
    ```

3. **Styles usage is disconnected from their definition**  
  You get no IntelliSense with CSS Modules, of what styles/classes are defined in the `.module.css` files, making **copy-paste** a required tool, lowering the DX. It also makes __refactoring very cumbersome__, because of the lack of safety.

4. **Using type-safe design tokens is a nightmare**  
  Any design tokens, defined in JS/TS cannot be directly used in CSS. There are 2 workarounds for this issue, neither of them being elegant:
   - We could inject them as [CSS Variables](https://developer.mozilla.org/en-US/docs/Web/CSS/Using_CSS_custom_properties), but we still don't get any IntelliSense or type-safety
   - We could use **inline styles**, which is less performant and also introduces another way to write styles (camelCase vs. kebab-case), while also splitting the styling in 2 different places.

<br />

## Goals

There are specific goals we're looking for:

- 🥇 SSR support and easy integration with Next.js
- 🥇 full TypeScript support
- 🥇 great DX with code completion & syntax highlight
- 🥈 light-weight
- 🥈 comprehensive documentation
- 🥉 intuitive API and low learning curve

<br />

Getting even more specific, we wanted to experience the usage of various CSS-in-JS solutions regarding:

- defining __global styles__
- using __media queries__ & __pseudo classes__
- __dynamic styles__ based on component `props` (aka. component variants), or from user input
- __bundle size__ impact

<br />

## Disclaimer

This analysis is intended to be **objective** and **unopinionated**:
- I don't work on any of these solutions, and have no intention or motivation of _promoting_ or _trashing_ either of them.  
- I have no prior experience with any CSS-in-JS solution, so I'm __not biased__ towards any of them. I've equally used all the solutions analyzed here.

<br />

👎 **What you WON'T FIND here?**  
- which solution is _"the best"_, or _"the fastest"_, as I'll not add any subjective grading, or performance metrics
- what solution should you pick for your next project, because I have no idea what your project is and what your goals are

<br />

👍 **What you WILL FIND here?**  
- an overview of (almost) all CSS-in-JS solutions available at this date (see _last update_ on top) that we've tried to integrate into a **Next.js v10 + TypeScript** empty project, with __minimal effort__;
- a limited set of **quantitative metrics** that allowed me to evaluate these solutions, which might help you as well;
- an additional list of **qualitative personal observations**, which might be either minor details or deal-breakers when choosing a particular solution.

The libraries are not presented in any particular order. If you're interested in a brief __history of CSS-in-JS__, you should checkout the [Past, Present, and Future of CSS-in-JS](https://www.youtube.com/watch?v=75kmPj_iUOA) talk by Max Stoiber.

---

<br/>
<br/>

## Overview

|      | [1. Co&#8209;location](#1-co-location) | [2. DX](#2-dx) | [3. `` tag` ` ``](#3-tag-tagged-templates) | [4. `{ }`](#4--object-styles) | [5. TS](#5-ts) | [6. `&` ctx](#6--ctx-contextual-styles) | [7. Nesting](#7-nesting) | 8. Theme | [9. `.css`](#9-css-static-css-extraction) | [10. `<style>`](#10-style-tag) | [11. Atomic](#11-atomic-css) | [12. `className`](#12-classname) | [13. `styled`](#13-styled) | [14. `css` prop](#14-css-prop) | [15. Learn](#15-learning-curve) | [16. Page size delta](#16-page-size-delta) |
| :--- | :------------------: | :---: | :-------------: | :------: | :---: | :--------: | :--------: | :------: | :-------: | :-----------: | :--------: | :-------------: | :----------: | :------------: | :-------: |     ---: |
| [CSS Modules](#css-modules)             | ❌ | ✅ | ✅ | ❌ | ❌ | ❌ | ✅ | ❌ | ✅ | ❌ | ❌ | ✅ | ❌ | ❌ | -  | -                     |
| [Styled JSX](#styled-jsx)               | ✅ | 🟠 | ✅ | ❌ | 🟠 | ❌ | ✅ | ❌ | ❌ | ✅ | ❌ | ✅ | ❌ | ❌ | 📉 |  `+3.6 kB / +13.0 kB` |
| [Styled Components](#styled-components) | ✅ | 🟠 | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ | ❌ | ✅ | 🟠 | 📈 | `+13.9 kB / +39.0 kB` |
| [Emotion](#emotion)                     | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ | ❌ | ✅ | ✅ | 📉 |  `+6.9 kB / +20.0 kB` |
| [Treat](#treat)                         | ❌ | ✅ | ❌ | ✅ | ✅ | ✅ | ❌ | ✅ | ✅ | ❌ | ❌ | ✅ | ❌ | ❌ | 📉 |  `+0.3 kB /  -0.1 kB` |
| [TypeStyle](#typestyle)                 | ✅ | ✅ | ❌ | ✅ | ✅ | ✅ | ✅ | 🟠 | ❌ | ✅ | ❌ | ✅ | ❌ | ❌ | 📈 |  `+2.8 kB / +19.0 kB` |
| [Fela](#fela)                           | ✅ | 🟠 | 🟠 | ✅ | 🟠 | ✅ | ✅ | ✅ | ❌ | ✅ | ✅ | ✅ | ❌ | ❌ | 📉 | `+12.6 kB / +45.0 kB` |
| [Stitches](#stitches)                   | ✅ | ✅ | ❌ | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ | ✅ | ✅ | ✅ | ✅ | ✅ | 📉 |  `+8.6 kB / +32.0 kB` |
| [JSS](#jss)                             | ✅ | 🟠 | 🟠 | ✅ | 🟠 | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ | ✅ | 🟠 | ❌ | 📉 | `+20.2 kB / +65.0 kB` |
| [Goober](#goober)                       | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ❌ | ✅ | ❌ | ✅ | ✅ | 🟠 | 📉 |  `+2.2 kB /  +7.0 kB` |

<br />

### LEGEND:

- ✅ - full & out-of-the-box support
- 🟠 - partial or limited support, less than ideal, or requiring some additional manual work for full support
- ❌ - lack of support

<br />

#### 1. Co-location

The ability to define styles within the same file as the component. You can also extract the styles into a separate file and import them, but the other way around does not apply.

[⬆️ to overview](#overview)

<br />

#### 2. DX

Refers to the **Developer eXperience** which includes 2 main aspects:

- **syntax highlighting** for styles definition;
- **code-completion/suggestions** for supported CSS properties, and available values (we're evaluating only the suggestion feature, not type-safety);

[⬆️ to overview](#overview)

<br />

#### 3. `` tag` ` `` (Tagged Templates)

Support for defining __styles as strings__, using ES Tagged Templates:

- uses `kebab-case` for property names, just like plain CSS syntax;
- enables easier migration from plain CSS to CSS-in-JS, because we don't have to completely re-write your styles;
- requires installing additional code editor plugin(s) for [syntax highlight and code completion](#2-dx), otherwise your code would look like a plain `string`;
- requires an additional step to parse the string and convert it to JS, which can be done either at built time (slower builds), or at runtime (slightly larger payload);

[⬆️ to overview](#overview)

<br />

#### 4. `{ }` (Object Styles)

Support for defining __styles as objects__, using plain JavaScript objects:

- uses `camelCase` for property names, like we do in [React Native](https://reactnative.dev/docs/next/style);
- migrating existing CSS requires a complete rewrite (don't know how we would automate this);
- we don't need additional tooling for syntax highlighting, as we get it out-of-the-box because we have to deal with JS code;
- without proper TS definitions shipped with the library, we won't get code completion (☝️ we're only interested in TS, not Flow);

[⬆️ to overview](#overview)

<br />

#### 5. TS

TypeScript support, either built-in, or via `@types` package, which should include:

- typings for the library API;
- Style Object typings (in case the library supports the object syntax);
- `Props` generics (if needed);

[⬆️ to overview](#overview)

<br />

#### 6. `&` ctx (Contextual Styles)

Support for __contextual styles__ allowing us to easily define __pseudo classes & elements__ and __media queries__ without the need to repeat the selector, as required in plain CSS:

- can either support the SASS/LESS/Stylus `&` parent selector;
- or provide any specific API or syntax to achieve this;

[⬆️ to overview](#overview)

<br />

#### 7. Nesting

Support for __arbitrary nested selectors__:

- this feature allows for great flexibility, which might be useful, or required in some specific use-cases;
- to keep in mind that it also introduces too many ways of defining styles, which might cause chaos if we want to enforce good-practices, consistency, scalability and maintainability;

[⬆️ to overview](#overview)

<br />

#### 8. Theme

Built-in support for Theming or managing design tokens/system. Note that **we haven't tested this feature**.

[⬆️ to overview](#overview)

<br />

#### 9. `.css` (Static CSS extraction)

Support for extracting and serving the styles as static `.css` files:
 
- it reduces the total bundle/page size, because we don't need additional runtime library, to inject and evaluate the styles;
- this approach affects **FCP/FMP** metrics negatively when users have an empty cache, and positively when having full cached styles;
- dynamic styling could potentially increase the generated file, because all style combinations must be pre-generated at built time;
- more suitable for less interactive solutions, where you serve a lot of different pages and you want to take advantage of cached styles (ie: e-commerce, blogs);

[⬆️ to overview](#overview)

<br />

#### 10. `<style>` tag

Support for serving the styles injected inside `<style>` tags in the document's `<head>`:

- makes dynamic styling super easy;
- incurs larger payload, because we're also shipping a runtime library to handle dynamic styles;
- when using SSR, styles required for the initial render are shipped twice to the client: once during SSR, and again during hydration;
- more suited for highly dynamic and interactive (single page) applications;

[⬆️ to overview](#overview)

<br />

#### 11. Atomic CSS

The ability to generate **atomic css classes**, thus increasing style reusability, and reducing duplication:

- this generates a separate CSS class for each CSS property;
- you'll get larger HTML files, because each element will contain a large number of CSS classes applied;
- theoretically [atomic CSS-in-JS](https://sebastienlorber.com/atomic-css-in-js) reduces the scaling factor of your styles, [Facebook is doing it](https://www.youtube.com/watch?v=9JZHodNR184) as well;
- it's debatable if the CSS total size reduction, is greater than the HTML size increase (what is the final delta)
- theoretically, if the class names are shorter than the CSS property definition, the delta is positive so we're shipping less bytes;
- however, we're basically moving part of bytes from CSS to HTML, which might be harder to cache if we have dynamic SSRed pages;
- also, depends a lot on what changes more frequently: the styles? or the markup?

[⬆️ to overview](#overview)

<br />

#### 12. `className`

The library API returns a `string` which we have to add to our component or element;

- this is similar how we would normally style React components, so it's easy to adopt because we don't have to learn a new way of dealing with styles;
- to combine styles we'll probably have to use string concatenation;

[⬆️ to overview](#overview)

<br />

#### 13. `styled`

The API creates a wrapper (styled) component which includes the generated `className`(s):

- we'll have to learn a new way to define styles, because we're not applying styles to elements, instead we're creating new components that include the styles elements;
- this also introduces a bit of indiretion when figuring out what native element gets rendered inside a larger component;
- this technique was first introduced and popularized by [Styled Components](#styled-components);

[⬆️ to overview](#overview)

<br />

#### 14. `css` prop

Allows passing styles using a special `css` prop, similar how you would define inline styles, but the library generates a unique CSS class name behind the scenes:

- it's a convenient and ergonomic API;
- this technique was first introduced and popularized by [Emotion](#emotion) v10;

[⬆️ to overview](#overview)

<br />

#### 15. Learning curve

A **subjective** opinion regarding the learning curve, considering that I have experience with CSS Modules, React, Hooks, TS.  
Note ☝️ - this is very _superficial_, and meant to be only a _note to myself_. You should really evaluate this on your own.

[⬆️ to overview](#overview)

<br />

#### 16. Page size delta

The total page size difference in kB (transferred gzipped & minified / uncompressed & minified) compared to __CSS Modules__, for the entire index page production build using Next.js:

- keep in mind that this includes an almost __empty page__, with only a couple of components;
- this is great for evaluating the minimal overhead, but does NOT offer any insight on the scaling factor: logarithmic, linear, or exponential;
- the values for the __runtime library__ are taken from Chrome Devtools Network tab, [Transferred over network vs Resource size](https://developers.google.com/web/tools/chrome-devtools/network/reference#uncompressed);

[⬆️ to overview](#overview)

<br/>

---

<br/>

### Overall observations

The following observations apply for all solutions (with minor pointed exceptions).

<br />

#### ✅ Code splitting

Components used only in a specific route will only be bundled for that route. This is something that Next.js performs out-of-the-box.

<br />

#### ✅ Global styles

All solutions offer a way to define global styles, some with a separate API.  

<br />

#### ✅ SSR

All solutions offer Server-Side Render support, and are easy to integrate with by Next.js.

<br />

#### ✅ Vendor prefixes

All solutions automatically add vendor specific prefixes out-of-the-box.

<br />

#### ✅ Unique class names

All solutions generate unique class names, like CSS Modules do. The algorithms used to generate these names vary a lot between libraries:

- some libraries use a **hashing** algorithm, requiring more computing, but resulting in idempotent names (for example: `.heading` style from `Card` component will always have the `.Card_heading_h7Ys5` hash);
- other libraries use **counting**, basically incrementing either a number (`.heading-0-2-1`, `.input-0-2-2`), or the alphabet letters (`a, b, c, ... aa, ab, ac`, etc), making this approach more performant, but resulting in non-idempotent class names (can't figure out if this has any potential drawbacks, or not);

<br />

#### ✅ No inline styles

None of the solutions generate inline styles, which is an older approach, used by Radium & Glamor. The approach is less performant than CSS classes, so it's [not recommended](https://reactjs.org/docs/dom-elements.html#style). It also implies using JS event handlers to trigger pseudo classes, as inline styles do not support them. Apparently, all modern solutions nowadays moved away from this approach.

<br />

#### ✅ Full CSS support

All solutions support most CSS properties that you would need: **pseudo classes & elements**, **media queries** and **keyframes** are the ones that we've tested.

<br />

#### 🟠 Performance Metrics

Understanding how these features affect [Core Web Vitals](https://web.dev/vitals/#core-web-vitals) and [Performance Metrics](https://web.dev/lighthouse-performance/#metrics) in general is an extremely important factor to consider, and the way styles are delivered to the client has probably the biggest impact, so let's analyse this in detail.

Also, there are 2 different scenarios we need to consider:

- 📭 **Empty cache**: the user visits our page for the first time, or a returning user visits our page after the cache was invalidated (a new version was released);
- 📬 **Full cache**: a returning user visits our page, and has all static resources cached (`.js`, `.css`, media, etc);

<br />

#### 1. `.css` file extraction  

Solutions that generate `.css` static files, which you normally would include as `<link>` tag(s) in the `<head>` of your page, are basically [rendering-blocking resources](https://developers.google.com/web/fundamentals/performance/critical-rendering-path/render-blocking-css). This highly affects **FCP**, **LCP** and any other metric that follows.

📭 **Empty cache**  
If the user has an empty cache, the following needs to happen, **negatively** impacting **FCP** and **LCP**:

- the browser needs to make an additional request, which implies a full **RTT** (Round Trip Time) to our server;
- transfer all CSS file content;
- parse it and build the CSSOM;
- these will delay any rendering of the `<body>`, even if the entire HTML is already loaded, and it may even be eagerly parsed, and some resources already fetched in advance;

It's true that you can fetch in **parallel** other `<head>` resources (additional `.css` or `.js` files), but this is generally a bad practice;

📬 **Full cache**  
However, on subsequent visits, the entire `.css` resource would be cached, so **FCP** and **LCP** would be positively impacted.

<br />

💡 **Key points**  
This solution appears to be better suited when:
- we have many Server Side Rendered pages that our users visit, maybe even containing a common `.css` file that can be cached when visiting other pages;
- we don't update the styles frequently, so they can be cached for longer periods of time;
- we want to optimize for returning visitors, affecting first-time visits instead;

<br />

#### 2. `<style>` tag injected styles

During **SSR**, styles will be added as `<style>` tag(s) in the `<head>` of the page. Keep in mind that these usually do NOT include all styles needed for the page, because most libraries perform [Critical CSS extraction](#-critical-css-extraction), so these `styles` should be usually smaller than the entire `.css` static file discussed previously.

📭 **Empty cache**  
Because we're shipping less CSS bytes, and they are inlined inside the `.html` file, this would result in faster **FCP** and **LCP**:

- we don't need additional requests for `.css` files, so the browser is not blocked;
- if we move all other `.js` files requests to the end of the document, `<head>` won't do any requests, so rendering will occur super fast;
- however, eventually we would ship additional bytes, that were not needed with static `.css` extraction:
   - the runtime library (between 1.6kB - 20kB);
   - the styles required for the page, bundled in `.js` files along with the components, during [hydration](https://nextjs.org/docs/basic-features/pages#pre-rendering) (this includes all the critical CSS already shipped inside the `<style>` tag + others);
- all these files are required to be fetched, parsed and executed to get a **fully interactive** page;

📬 **Full cache**  
When the user's cache is full, the additional `.js` files won't require fetching, as they are already cached.  
However, if the page is **SSRed**, the inlined critical CSS rendered in the `<style>` tag of the document will be downloaded again, unless we deal with static HTML that can be cached as well, or we deal with HTML caching on our infrastructure.

But, by default, we will ship extra bytes on every page HTTP request, regardless if it's cached or not.

<br />

💡 **Key points**  
This solution appears to be better suited when:
- we deal with SPA (Single Page Applications), where we have one (or few) SSR pages;
- we update the styles frequently, so even if they could be cached, it won't have a positive impact;
- we want to optimize for first-time visitors, affecting returning visitors instead;

<br />

#### 🟠 Critical CSS extraction

⚠️ Work in progress...

<br />

#### 🟠 Dead code removal

Most solutions say they remove unused code/styles. This is only **half-true**. Unused code is indeed more difficult to accumulate, especially of you compare it to plain `.css` files as we used to write _a decade ago_. But when compared to CSS Modules, the differencies are not that big. Any solution that offers the option to define **arbitrary selectors** or **nested styles** will bundle them, regardless if they are used or not inside the component. We've managed to ship unused styles with all the tested solutions.

Basically, what we get is code removal when you delete the component, or you don't import it anymore.

<br />

#### 🟠 Debugging / Inspecting

There are 2 methods to inject & update styles into the DOM from JavaScript:

<br />

##### 1. Using `<style>` tag(s)

This approach implies adding one or more `<style>` tag(s) in the DOM (either in the `<head>` or somewhere in the `<body>`), using [.appendChild()](https://developer.mozilla.org/en-US/docs/Web/API/Node/appendChild) to add the `<style>` Node(s), in addition with either [.textContent](https://developer.mozilla.org/en-US/docs/Web/API/Node/textContent), [.innerHTML](https://developer.mozilla.org/en-US/docs/Web/API/Element/innerHTML) to update the `<style>` tag(s).

- using this approach, we can easily _see_ what styles get added to the DOM, because we can inspect the DOM from our DevTools, like any other DOM Node;
- using only one `<style>` tag and updating its whole content, could be slow to update the entire DOM when we actually changed only a tiny set of CSS rule(s);
- most libraries use this solution in `DEVELOPMENT` mode, because it provides a better debugging experience;
   - **TypeStyle** uses this in `PRODUCTION` also;

<br />

##### 2. Using `CSSStyleSheet` API

First used by **JSS**, this method uses [`CSSStyleSheet.insertRule()`](https://developer.mozilla.org/en-US/docs/Web/API/CSSStyleSheet/insertRule) to inject the styles directly into the **CSSOM**.

- using this approach it's a bit more difficult to _see_ what styles get injected into the CSSOM, because even if you see the CSS applied on the elements it will point to an empty `<style>` tag;
   - to see all the injected styles, you'll have to select the `<style>` tag;
   - get access to it via `$0` in Chrome DevTools (or get a reference to it in any other way, using the DOM API);
   - access `.sheet.cssRules` on the `<style>` tag to see the Array of CSS rules that it contains;
- this method is apparently more performant than the previous one, so most libraries use this method in `PRODUCTION`;
   - **JSS** and **Stitches** use it in `DEVELOPMENT` mode as well;

<br />

#### ❌ No component deduping
If the same component is imported by 2 different routes, it will be send twice to the client. This is surely a limitation of the bundler/build system, in our case Next.js, and __not related to the CSS-in-JS solution__.

In Next.js, code-splitting works at the route level, bundling all components required for a specific route, but according to their [official blog](https://nextjs.org/blog/next-9-2#improved-code-splitting-strategy) and [web.dev](https://web.dev/granular-chunking-nextjs/) if a component is used in __more than 50%__ of the pages, it should be included in the `commons` bundle. However, in our example, we have 2 pages, each of them importing the `Button` component, and it's included in each page bundle, not in the `commons` bundle. Since the code required for styling is bundled with the component, this limitation will impact the styles as well, so it's worth keeping this in mind.

<br />

---

<br />

### CSS Modules

This is a well established, mature and solid approach. Without a doubt, it's a great improvement over BEM, SMACCS, or any other methodology to structure and organize your CSS, especially in component-based applications.

Launched in __2015__ | [Back to Overview](#overview)

<br />

- ✅ __Context-aware code completion__
- ❌ __No Styles/Component co-location__
- ❌ __No TypeScript support__
- ❌ __No Atomic CSS__
- ❌ __No Theming support__

- __Styles definition method(s)__
  - ✅ plain CSS
  - ❌ Style Objects

- __Styles nesting__
  - ❌ Contextual styles: _(requires SASS, LESS or Stylus)_
  - ✅ Abitrary nesting

- __Styles apply method(s)__
  - ✅ `className`
  - ❌ `styled` component
  - ❌ `css` prop

- __Styles output__
  - ✅ `.css` file extraction
  - ❌ `<style>` tag injection

- 📉📈 __Learning curve__: easy to learn, but difficult to master

<br />

This is the baseline we'll consider when comparing all the following __CSS-in-JS__ solutions. Checkout the [motivation](#motivation) to better understand the limitations of this approach that we're trying to fill.

<br />

|                 | Transferred / gzipped | Uncompressed |
| :-------------- | --------------------: | -----------: |
| Runtime library |                     - |            - |
| Index page size |               71.5 kB |       201 kB |

<br />

```
Page                                Size     First Load JS
┌ ○ /                               2.15 kB        64.9 kB
├   └ css/7a5b6d23ea12e90bddea.css  407 B
├   /_app                           0 B            62.7 kB
├ ○ /404                            3.03 kB        65.7 kB
└ ○ /other                          706 B          63.4 kB
    └ css/57bb8cd5308b249275fa.css  443 B
+ First Load JS shared by all       62.7 kB
  ├ chunks/commons.7af247.js        13.1 kB
  ├ chunks/framework.9d5241.js      41.8 kB
  ├ chunks/main.03531f.js           6.62 kB
  ├ chunks/pages/_app.6e472f.js     526 B
  ├ chunks/webpack.50bee0.js        751 B
  └ css/d9aac052842a915b5cc7.css    325 B
```

<br/>

---

<br/>

### Styled JSX

Very simple solution, doesn't have a dedicated website for documentation, everything is on Github. It's not popular, but it is the built-in solution in Next.js.

Version: __`3.4`__ | Maintained by [Vercel](https://github.com/vercel) | Launched in __2017__ | [View Docs](https://github.com/vercel/styled-jsx) | ... [back to Overview](#overview)

<br />

- ✅ __Styles/Component co-location__
- 🟠 __Context-aware code completion__:  to get syntax highlighting & code completion, an editor extension is required
- 🟠 __TypeScript support__:  `@types` can be additionaly installed, but the API is too minimal to require TS
- ❌ __No Atomic CSS__
- ❌ __No Theming support__

- __Styles definition method(s)__
  - ✅ Tagged Templates
  - ❌ Style Objects

- __Styles nesting__
  - ❌ Contextual styles
  - ✅ Abitrary nesting

- __Styles apply method(s)__
  - ✅ `className`
  - ❌ `styled` component
  - ❌ `css` prop

- __Styles output__
  - ❌ `.css` file extraction
  - ✅ `<style>` tag injection

- 📉 __Low Learning curve__: because the API is minimal and very simple

<br />

#### Other benefits

- 😌 out-of-the-box support with Next.js
- 👍 for user input styles, it generates a new class name for each change, but it removes the old one
- 👌 it generates slightly smaller styles than the default CSS modules settings, probably due to shorter unique class names (you can see this in the difference between the runtime library size __3.9 kB__ and the page size increase, which I'd expect to be larger, but instead it's __3.6 kB__, this being tiny code base, so this delta might scale)

<br />

#### Worth mentioning observations

- 😏 unlike CSS modules, we can target HTML `elements` also, and it generates unique class names for them (not sure if it's a good practice, though)
- 🤓 we'll need to optimize our styles by [splitting static & dynamic styles](https://github.com/vercel/styled-jsx#dynamic-styles), to avoid rendering duplicated styles
- 🤨 unique class names are added to elements, even if we don't target them in our style definition, resulting in un-needed slight html pollution (optimizing this is cumbersome, and it's _a lot of work for little benefit_)
- 😕 it will bundle any defined styles, regardless if they are used or not, just like plain CSS
- 😢 there's no support for __contextual styles__, so defining __pseudo classes__ or __media queries__ has the same downsides as plain CSS, requiring selectors/class names duplication (a [SASS plugin](https://github.com/vercel/styled-jsx#css-preprocessing-via-plugins) is required to get this feature)

<br />

#### Conclusions

Overall, we felt like writting plain CSS, with the added benefit of being able to define the styles along with the component, so we __don't need an additional `.css` file__. Indeed, this is the philosophy of the library: supporting CSS syntax inside the component file. We can __use any JS/TS constants of functions__ with string interpolation. Working with __dynamic styles is pretty easy__ because it's plain JavaScript in the end. We get all these benefits at a very low price, with a pretty __small bundle overhead__.

The downsides are the overall experience of writting plain CSS. __Without nesting support__ pseudo classes/elements and media queries getting pretty cumbersome to manage.

<br />

|                 | Transferred / gzipped | Uncompressed |
| :-------------- | --------------------: | -----------: |
| Runtime library |                3.9 kB |      10.7 kB |
| Index page size |               75.1 kB |       214 kB |
| vs. CSS Modules |           __+3.6 kB__ |   __+13 kB__ |

<br />

```
Page                                                           Size     First Load JS
┌ ○ /                                                          2.64 kB        69.3 kB
├   /_app                                                      0 B            66.6 kB
├ ○ /404                                                       3.03 kB        69.6 kB
└ ○ /other                                                     1.17 kB        67.8 kB
+ First Load JS shared by all                                  66.6 kB
  ├ chunks/1dfa07d0b4ad7868e7760ca51684adf89ad5b4e3.3baab1.js  3.53 kB
  ├ chunks/commons.7af247.js                                   13.1 kB
  ├ chunks/framework.9d5241.js                                 41.8 kB
  ├ chunks/main.99ad68.js                                      6.62 kB
  ├ chunks/pages/_app.949398.js                                907 B
  └ chunks/webpack.50bee0.js                                   751 B
```

<br/>

---

<br/>

### Styled Components

For sure one of the most popular and mature solutions, with good documentation. It uses Tagged Templates to defines styles by default, but can use objects as well. It also popularized the `styled` components approach, which creates a new component along with the defined styles.

Version: __`5.2`__ | Maintained by [Max Stoiber](https://twitter.com/mxstbr) & [others](https://opencollective.com/styled-components#category-ABOUT) | Launched in __2016__ | [View Docs](https://styled-components.com/docs) | ... [back to Overview](#overview)

<br />

- ✅ __Styles/Component co-location__
- ✅ __TypeScript support__:  `@types` must be additionaly installed, via DefinitelyTyped
- ✅ __Built-in Theming support__
- 🟠 __Context-aware code completion__: requires an editor extension/plugin
- ❌ __No Atomic CSS__

- __Styles definition method(s)__
  - ✅ Tagged Templates
  - ✅ Style Objects

- __Styles nesting__
  - ✅ Contextual styles
  - ✅ Abitrary nesting

- __Styles apply method(s)__
  - ❌ `className`
  - ✅ `styled` component
  - 🟠 `css` prop

- __Styles output__
  - ❌ `.css` file extraction
  - ✅ `<style>` tag injection

- 📈 __Higher Learning curve__: we have to learn the API, get used to using the `styled` wrapper components, and basically get used to a new way to manage our styles

<br />

#### Worth mentioning observations

- 🧐 the `css` prop is mentioned in the API docs, but there are no usage examples
- 🤓 we need to split static & dynamic styles, otherwise it will render duplicate output
- 😕 bundles nested styles even if they are not used in component
- 😵 we can mix Tagged Templates with Styled Objects, which could lead to convoluted and different syntax for each approach (kebab vs camel, EOL character, quotes, interpolation, etc)
- 🥴 some more complex syntax appears to be a bit cumbersome to get right (mixing animations with Styled Objects, dynamic styles based on `Props` variations, etc)
- 🤫 for user input styles, it generates a new class name for each update, but it does NOT remove the old ones, appending indefinitely to the DOM

<br />

#### Conclusions

Styled components offers a novel approach to styling components using the `styled` method which creates a new component including the defined styles. You don't feel like writting CSS, so coming from CSS Modules we'll have to learn a new, more programatic way, to define styles. Because it allows both `string` and `object` syntax, it's a pretty flexibile solution both for migrating our existing styles, and for starting a project from scratch. Also, the maintainers did a pretty good job keeping up with most of the innovations in this field.

However before adopting it, we must be aware that it comes with a certain cost for our bundle size.

<br />

|                 | Transferred / gzipped | Uncompressed |
| :-------------- | --------------------: | -----------: |
| Runtime library |               14.2 kB |      36.7 kB |
| Index page size |               85.4 kB |       240 kB |
| vs. CSS Modules |          __+13.9 kB__ |   __+39 kB__ |

<br />

```
Page                                                           Size     First Load JS
┌ ○ /                                                          2.5 kB         79.4 kB
├   /_app                                                      0 B            76.9 kB
├ ○ /404                                                       3.03 kB        79.9 kB
└ ○ /other                                                     1.04 kB        77.9 kB
+ First Load JS shared by all                                  76.9 kB
  ├ chunks/1dfa07d0b4ad7868e7760ca51684adf89ad5b4e3.3f0ffd.js  13.8 kB
  ├ chunks/commons.7af247.js                                   13.1 kB
  ├ chunks/framework.9d5241.js                                 41.8 kB
  ├ chunks/main.99ad68.js                                      6.62 kB
  ├ chunks/pages/_app.7093f3.js                                921 B
  └ chunks/webpack.50bee0.js                                   751 B
```

<br/>

---

<br/>

### Emotion

Probably the most comprehensive, complete and sofisticated solution. Detailed documentation, fully built with TypeScript, looks very mature, rich in features and well maintained.

Version: __`11.1`__ | Maintained by [Mitchell Hamilton](https://twitter.com/mitchellhamiltn) & [others](https://opencollective.com/emotion#category-ABOUT) | Launched in __2017__ | [View Docs](https://emotion.sh/docs/introduction) | ... [back to Overview](#overview)

<br />

- ✅ __Styles/Component co-location__
- ✅ __TypeScript support__
- ✅ __Built-in Theming support__
- ✅ __Context-aware code completion__: for using the `styled` components approach, an additional editor plugin is required
- ❌ __No Atomic CSS__

- __Styles definition method(s)__
  - ✅ Tagged Templates
  - ✅ Style Objects

- __Styles nesting__
  - ✅ Contextual styles
  - ✅ Abitrary nesting

- __Styles apply method(s)__
  - ❌ `className`
  - ✅ `styled` component
  - ✅ `css` prop

- __Styles output__
  - ❌ `.css` file extraction
  - ✅ `<style>` tag injection  

- 📉 __Low Learning curve__: when using the `css` prop, which is the primary approach, the API is pretty straightforward (the `styled` approach however incurs the same learning curve for [Styled Components](#styled-components))

<br />

#### Other benefits

- 😎 the `css` prop offers great ergonomics during development, however it seems to be a newer approach, based on [React 17 new `jsx` transform](https://reactjs.org/blog/2020/09/22/introducing-the-new-jsx-transform.html), and [configuring](https://emotion.sh/docs/css-prop) it is not trivial, differs on your setup, and implies some boilerplate (which should change soon and become easier)

<br />

#### Worth mentioning observations

- 😕 bundles nested styles even if they are not used in component
- 🤫 for user input styles, it generates a new class name for each update, but it does NOT remove the old ones, appending indefinitely to the DOM
- 😑 using `styled` approach will add `3 kB` to our bundle, because it's imported from a separate package
- 🤔 don't know how to split static and dynamic styles, resulting in highly polluted duplicated styles in head for component variants (same applies to `css` prop & `styled` components)

<br />

#### Conclusions

Overall Emotion looks to be a very solid and flexible approach. The novel `css` prop approach offers great ergonomics for developers. Working with dynamic styles and TypeScript is pretty easy and intuitive. Supporting both `strings` and `objects` when defining styles, it can be easily used both when migrating from plain CSS, or starting from scratch. The bundle overhead is not negligible, but definitely much smaller than other solutions, especially if you consider the rich set of features that it offers.

It seems it doesn't have a dedicated focus on performance, but more on Developer eXperience. It looks like a perfect "well-rounded" solution.

<br />

|                 | Transferred / gzipped | Uncompressed |
| :-------------- | --------------------: | -----------: |
| Runtime library |                7.5 kB |      19.0 kB |
| Index page size |               78.4 kB |       221 kB |
| vs. CSS Modules |           __+6.9 kB__ |   __+20 kB__ |

<br />

```
Page                                                           Size     First Load JS
┌ ○ /                                                          2.47 kB        72.6 kB
├   /_app                                                      0 B            70.1 kB
├ ○ /404                                                       3.03 kB        73.1 kB
└ ○ /other                                                     1.04 kB        71.1 kB
+ First Load JS shared by all                                  70.1 kB
  ├ chunks/1dfa07d0b4ad7868e7760ca51684adf89ad5b4e3.19c2e4.js  7.1 kB
  ├ chunks/commons.800e6d.js                                   13.1 kB
  ├ chunks/framework.9d5241.js                                 41.8 kB
  ├ chunks/main.45755e.js                                      6.55 kB
  ├ chunks/pages/_app.398ef5.js                                832 B
  └ chunks/webpack.50bee0.js                                   751 B
```

<br/>

---

<br/>

### Treat

Modern solution with great TypeScript integration and no runtime overhead. It's pretty minimal in its features, straightforward and opinionated. Everything is processed at compile time, and it generates static CSS files, similar to CSS Modules, Linaria, or Astroturf.

Version: __`1.6`__ | Maintained by [Seek OSS](https://github.com/seek-oss/) | Launched in __2019__ | [View Docs](https://seek-oss.github.io/treat/) | ... [back to Overview](#overview)

<br />

- ✅ __TypeScript support__
- ✅ __Built-in Theming support__
- ✅ __Context-aware code completion__
- ❌ __No Styles/Component co-location__: styles must be placed in an external `.treat.ts` file
- ❌ __No Atomic CSS__

- __Styles definition method(s)__
  - ❌ Tagged Templates
  - ✅ Style Objects

- __Styles nesting__
  - ✅ Contextual styles
  - ❌ Abitrary nesting

- __Styles apply method(s)__
  - ✅ `className`
  - ❌ `styled` component
  - ❌ `css` prop

- __Styles output__
  - ✅ `.css` file extraction
  - ❌ `<style>` tag injection

- 📉 __Low Learning curve__: coming from CSS Modules it feels like home, the additional API required for variants is pretty straightforward and easy to learn

<br />

#### Other benefits

- 👮 forbids __nested arbitrary selectors__ (ie: `& > span`), which might be seen as a downside, when it's actually discourages bad-practices like __specificity wars__

<br />

#### Worth mentioning observations

- 😕 bundles styles even if they are not used in component
- 😥 it doesn't handle dynamic styles: you can use built-in `variants` based on predefined types, or __inline styles__ for user defined styles

<br />

#### Conclusions

When using Treat, we felt a lot like using CSS Modules: we need an external file for styles, we place the styles on the elements using `className`, we handle dynamic styles with __inline styles__, etc. However, we don't write CSS, and the overall experience with TypeScript support is magnificent, because everything is typed, so we don't do any __copy-paste__. Error messages are very helpful in guiding us when we do something we're not supposed to do. It's also the only analyzed solution the __extracts styles as `.css` files__ at built time, which should greatly improve the page load metrics.

The only thing to look out for is the limitation regarding dynamic styling. In highly interactive UIs that require user input styling, we'll have to use inline styles.

Treat is built with restrictions in mind, with a strong user-centric focus, balacing the developer experience with solid TypeScript support. It's also worth mentioning that [Mark Dalgleish](https://twitter.com/markdalgleish), co-author of CSS Modules, works at Seek and he's also a contributor.

<br />

|                 | Transferred / gzipped | Uncompressed |
| :-------------- | --------------------: | -----------: |
| Runtime library |                     - |            - |
| Index page size |               71.8 kB |       200 kB |
| vs. CSS Modules |           __+0.3 kB__ |    __-1 kB__ |

<br />

```
Page                                Size     First Load JS
┌ ○ /                               2.11 kB        64.8 kB
├   └ css/4ca0d586ad5efcd1970b.css  422 B
├   /_app                           0 B            62.7 kB
├ ○ /404                            3.03 kB        65.8 kB
└ ○ /other                          632 B          63.4 kB
    └ css/adb81858cf67eabcd313.css  435 B
+ First Load JS shared by all       62.7 kB
  ├ chunks/commons.7af247.js        13.1 kB
  ├ chunks/framework.9d5241.js      41.8 kB
  ├ chunks/main.03531f.js           6.62 kB
  ├ chunks/pages/_app.2baddf.js     546 B
  ├ chunks/webpack.50bee0.js        751 B
  └ css/08916f1dfb6533efc4a4.css    286 B
```

<br/>

---

<br/>

### TypeStyle

Minimal library, focused only on type-checking. It is framework agnostic, that's why it doesn't have a special API for handling dynamic styles. There are React wrappers available, but the typings feels a bit convoluted.

Version: __`2.1`__ | Maintained by [Basarat](https://twitter.com/basarat) | Launched in __2017__ | [View Docs](https://typestyle.github.io/) | ... [back to Overview](#overview)

<br />

- ✅ __Styles/Component co-location__
- ✅ __TypeScript support__
- ✅ __Context-aware code completion__
- 🟠 __Built-in Theming support__: uses TS `namespaces` to define theming, which is [not recommended](https://basarat.gitbook.io/typescript/project/namespaces) even by the author himself, or by TS core team member [Orta Therox](https://youtu.be/8qm49TyMUPI?t=1277).
- ❌ __No Atomic CSS__

- __Styles definition method(s)__
  - ❌ Tagged Templates
  - ✅ Style Objects

- __Styles nesting__
  - ✅ Contextual styles
  - ✅ Abitrary nesting

- __Styles apply method(s)__
  - ✅ `className`
  - ❌ `styled` component
  - ❌ `css` prop

- __Styles output__
  - ❌ `.css` file extraction
  - ✅ `<style>` tag injection

- 📈 __High Learning curve__: the API is simple, but it doesn't provide a lot of features, so we'll still need to do manual work and to re-adjust the way we'll author styles

<br />

#### Worth mentioning observations

- 😕 bundles nested styles even if they are not used in component
- 😕 it doesn't handle dynamic styles, so we have to use regular JS functions to compute styles
- 🤨 when composing styles, we'll have to manually add some internal typings
- 🤔 don't know how to split dynamic and static styles, so it's very easy to create duplicated generated code
- 😱 it creates a single `<style>` tag with all the styles, and replaces it on update, and apparently it doesn't use `insertRule()`, not even in production builds, which might be an important performance drawback in large & highly dynamic UIs

<br />

#### Conclusions

Overall TypeStyle seems a minimal library, relatively easy to adopt because we don't have to rewrite our components, thanks to the classic `className` approach. However we do have to rewrite our styles, because of the Style Object syntax. We didn't feel like writting CSS, so there is a learning curve we need to climb.

With Next.js or React in general we don't get much value out-of-the-box, so we still need to perform a lot of manual work. The external [react-typestyle](https://github.com/Malpaux/react-typestyle) binding doesn't support hooks, it seems to be an abandoned project and the typings are too convoluted to be considered an elegant solution.

<br />

|                 | Transferred / gzipped | Uncompressed |
| :-------------- | --------------------: | -----------: |
| Runtime library |                4.5 kB |       8.6 kB |
| Index page size |               74.3 kB |       210 kB |
| vs. CSS Modules |           __+2.8 kB__ |   __+19 kB__ |

<br />

```
Page                                                           Size     First Load JS
┌ ○ /                                                          2.41 kB        68.6 kB
├   /_app                                                      0 B            66.2 kB
├ ○ /404                                                       3.03 kB        69.2 kB
└ ○ /other                                                     953 B          67.1 kB
+ First Load JS shared by all                                  66.2 kB
  ├ chunks/1dfa07d0b4ad7868e7760ca51684adf89ad5b4e3.250ad4.js  3.09 kB
  ├ chunks/commons.7af247.js                                   13.1 kB
  ├ chunks/framework.9d5241.js                                 41.8 kB
  ├ chunks/main.99ad68.js                                      6.62 kB
  ├ chunks/pages/_app.d59d73.js                                893 B
  └ chunks/webpack.50bee0.js                                   751 B
```

<br/>

---

<br/>

### Fela

It appears to be a mature solution, with quite a number of users. The API is intuitive and very easy to use, great integration for React using hooks.

Version: __`11.5`__ | Maintained by [Robin Weser](https://twitter.com/robinweser) | Launched in __2016__ | [View Docs](https://fela.js.org/docs/) | ... [back to Overview](#overview)

<br />

- ✅ __Styles/Component co-location__
- ✅ __Built-in Theming support__
- ✅ __Atomic CSS__
- 🟠 __TypeScript support__: it exposes Flow types, which work ok, from our (limited) experience
- 🟠 __Context-aware code completion__: styles defined outside the component require explicit typing to get code completion

- __Styles definition method(s)__
  - 🟠 Tagged Templates
  - ✅ Style Objects

- __Styles nesting__
  - ✅ Contextual styles
  - ✅ Abitrary nesting

- __Styles apply method(s)__
  - ✅ `className`
  - ❌ `styled` component
  - ❌ `css` prop

- __Styles output__
  - ❌ `.css` file extraction
  - ✅ `<style>` tag injection

- 📉 __Low Learning curve__: the API is simple, considering that we're comfortable with React hooks

<br />

#### Other benefits

- 😌 easy and simple to use API, very intuitive
- 🥳 creates very short and atomic class names (like `a`, `b`, ...)
- 😎 it has a lot of plugins that can add many additional features (but will also increase bundle size)

<br />

#### Worth mentioning observations

- 😕 bundles nested styles even if they are not used in component
- 🤨 when defining styles outside the component, we have to explicitly add some internal typings to get code completion
- 🥺 there's no actual TS support and the maintainer considers it a [low priority](https://github.com/robinweser/fela/issues/590#issuecomment-409373362)
- 🤕 without TS support, we cannot get fully type-safe integration into Next.js + TS (there are [missing types from the definition file](https://twitter.com/pfeiffer_andrei/status/1349106486740475904))
- 🤔 the docs say it supports string based styles, but they are a second-class citizen and they seem to work only for global styles
- 😵 some information in the docs is spread on various pages, sometimes hard to find without a search feature, and the examples and use cases are not comprehensive

<br />

#### Conclusions

Fela looks to be a mature solution, with active development. It introduces 2 great features which we enjoyed a lot. The first one is the basic principle that _"Style as a Function of State"_ which makes working with dynamic styles feel super natural and integrates perfectly with React's mindset. The second is atomic CSS class names, which should potentially scale great when used in large applications.

The lack of TS support however is a bummer, considering we're looking for a fully type-safe solution. Also, the scaling benefits of atomic CSS should be measured against the library bundle size.

<br />

|                 | Transferred / gzipped | Uncompressed |
| :-------------- | --------------------: | -----------: |
| Runtime library |               13.1 kB |      42.6 kB |
| Index page size |               84.1 kB |       246 kB |
| vs. CSS Modules |          __+12.6 kB__ |   __+45 kB__ |

<br />

```
Page                             Size     First Load JS
┌ ○ /                            3.46 kB        78.6 kB
├   /_app                        0 B            75.2 kB
├ ○ /404                         3.03 kB        78.2 kB
└ ○ /other                       2.06 kB        77.2 kB
+ First Load JS shared by all    75.2 kB
  ├ chunks/commons.7af247.js     13.1 kB
  ├ chunks/framework.37f4a7.js   42.1 kB
  ├ chunks/main.03531f.js        6.62 kB
  ├ chunks/pages/_app.f7ff86.js  12.6 kB
  └ chunks/webpack.50bee0.js     751 B
```

<br/>

---

<br/>

### Stitches

Very young library, probably the most solid, modern and well-thought-out solution. The overall experience is just great, full TS support, a lot of other useful features baked in the lib.

Version: __`0.0.2`__ | Maintained by [Modulz](https://github.com/modulz) | Launched in __2020__ | [View Docs](https://stitches.dev/docs) | ... [back to Overview](#overview)

<br />

- ✅ __Styles/Component co-location__
- ✅ __TypeScript support__
- ✅ __Context-aware code completion__
- ✅ __Built-in Theming support__
- ✅ __Atomic CSS__

- __Styles definition method(s)__
  - ❌ Tagged Templates
  - ✅ Style Objects

- __Styles nesting__
  - ✅ Contextual styles
  - ✅ Abitrary nesting

- __Styles apply method(s)__
  - ✅ `className`
  - ✅ `styled` component
  - ✅ `css` prop _(used only to override `styled` components)_

- __Styles output__
  - ❌ `.css` file extraction
  - ✅ `<style>` tag injection

- 📉 __Low Learning curve__: the API is simple and intuitive, documentation is top-notch

<br />

#### Other benefits

- 😌 easy and simple to use API, a pleasure to work with
- 😎 great design tokens management and usage
- 🥰 documentation is exactly what we'd expect: no more, no less

<br />

#### Worth mentioning observations

- 😕 bundles nested styles even if they are not used in component
- 😵 uses `insertRule()` in development also, so we cannot see what gets bundled
- 🤨 it expands short-hand properties, from `padding: 1em;` will become `padding-top: 1em; padding-right: 1em; padding-bottom: 1em; padding-left: 1em;`
- 🤔 dynamic styles can be defined either using built-in `variants` (for predefined styles), or styles created inside the component to get access to the `props`
- 🧐 would help a lot to get the search feature inside the docs

<br />

#### Conclusions

Stitches is probably the most modern solution to this date, with full out-of-the-box support for TS. Without a doubt, they took some of the best features from other solutions and put them together for an awesome development experience. The first thing that impressed us was definitely the documentation. The second, is the API they expose which is close to top-notch. The features they provide are not huge in quantity, but are very well-thought-out.

However, we cannot ignore the fact that it's still in beta. Also, the authors identify it as "light-weight", but at __8 kB__ it's worth debating. Nevertheless, we will keep our eyes open and follow its growth.

<br />

|                 | Transferred / gzipped | Uncompressed |
| :-------------- | --------------------: | -----------: |
| Runtime library |                8.9 kB |      29.5 kB |
| Index page size |               80.1 kB |       233 kB |
| vs. CSS Modules |           __+8.6 kB__ |   __+32 kB__ |

<br />

```
Page                                                           Size     First Load JS
┌ ○ /                                                          2.42 kB        73.9 kB
├   /_app                                                      0 B            71.5 kB
├ ○ /404                                                       3.03 kB        74.5 kB
└ ○ /other                                                     959 B          72.4 kB
+ First Load JS shared by all                                  71.5 kB
  ├ chunks/1dfa07d0b4ad7868e7760ca51684adf89ad5b4e3.f723af.js  8.46 kB
  ├ chunks/commons.7af247.js                                   13.1 kB
  ├ chunks/framework.9d5241.js                                 41.8 kB
  ├ chunks/main.99ad68.js                                      6.62 kB
  ├ chunks/pages/_app.51b7a9.js                                832 B
  └ chunks/webpack.50bee0.js                                   751 B
```

<br/>

---

<br/>

### JSS

Probably the grandaddy around here, JSS is a very mature solution being the first of them, and still being maintained. The API is intuitive and very easy to use, great integration for React using hooks.

Version: __`10.5`__ | Maintained by [Oleg Isonen](https://twitter.com/oleg008) and [others](https://opencollective.com/jss#category-ABOUT) | Launched in __2014__ | [View Docs](https://cssinjs.org/) | ... [back to Overview](#overview)

<br />

- ✅ __Styles/Component co-location__
- ✅ __Built-in Theming support__
- ❌ __Atomic CSS__
- 🟠 __TypeScript support__ _([definition files](https://github.com/cssinjs/jss/blob/master/packages/react-jss/src/index.d.ts) exist, but for some reason, they [don't work](https://github.com/andreipfeiffer/css-in-js/issues/9#issuecomment-774125968))_
- 🟠 __Context-aware code completion__ _(Object Styles didn't work for us, due to lack of TS support)_

- __Styles definition method(s)__
  - 🟠 Tagged Templates: _(available with additional [plugin](https://cssinjs.org/jss-plugin-template?v=v10.5.1), with limited features)_
  - ✅ Style Objects

- __Styles nesting__
  - ✅ Contextual styles
  - ✅ Abitrary nesting

- __Styles apply method(s)__
  - ✅ `className`
  - 🟠 `styled` component _(available with additional [plugin](https://cssinjs.org/styled-jss?v=v2.2.3))_
  - ❌ `css` prop

- __Styles output__
  - ❌ `.css` file extraction
  - ✅ `<style>` tag injection

- 📉 __Low Learning curve__: the API is simple, considering that we're comfortable with React hooks

<br />

#### Other benefits

- 😌 easy and simple to use API, very intuitive
- 😎 it has a lot of plugins that can add many additional features (but will also increase bundle size)

<br />

#### Worth mentioning observations

- 😕 bundles nested styles even if they are not used in component
- 😳 keep in mind that [`react-jss` package](https://github.com/cssinjs/jss/blob/master/packages/react-jss/package.json#L47), which is used with React/Next.js, depends on [jss-preset-default](https://cssinjs.org/jss-preset-default), which includes many [plugins](https://github.com/cssinjs/jss/blob/master/packages/jss-preset-default/package.json#L35-L47) by default, so you don't need to manually add some of the plugins;
- 🤔 `react-jss` uses className by default. There's also `styled-jss` that uses __Styled Components__ approach, but it has no types, and couldn't make it work on top of `react-jss`;
- 😤 global styles were frustrating to setup, we've finally managed to used them thanks to [StackOverFlow](https://stackoverflow.com/questions/54201412/how-can-i-add-style-to-the-body-element-with-jss), because the docs have no mention of `injectSheet` API (or we couldn't find it anywhere);
- 😖 the docs are generally difficult to follow, and finding the information you need is a cumbersome process:
   - there is no search;
   - there are a lot of plugins, so you don't know where to look for a particular feature;
   - some plugins influence other plugins, or other docs pages, and they sometimes don't contain all the combinations of features, so the docs are not comprehensive (ie: we had to figure out on our own how to use **contextual styles** with **media queries**).

<br />

#### Conclusions

The API is similar in many ways to React Native StyleSheets, while the hooks helper allows for easy dynamic styles definition. There are many plugins that can add a lot of features to the core functionality, but attention must be payed to the total bundle size, which is significant even with the bare minimum only.

Also, being the first CSS-in-JS solution built, it lacks many of the modern features that focuses on developer experience.

<br />

|                 | Transferred / gzipped | Uncompressed |
| :-------------- | --------------------: | -----------: |
| Runtime library |               19.3 kB |      58.7 kB |
| Index page size |               91.7 kB |       266 kB |
| vs. CSS Modules |          __+20.2 kB__ |   __+65 kB__ |

<br />

```
Page                                                           Size     First Load JS
┌ ○ /                                                          2.42 kB        85.9 kB
├   /_app                                                      0 B            83.4 kB
├ ○ /404                                                       3.03 kB        86.5 kB
└ ○ /other                                                     969 B          84.4 kB
+ First Load JS shared by all                                  83.4 kB
  ├ chunks/1dfa07d0b4ad7868e7760ca51684adf89ad5b4e3.c41897.js  18.9 kB
  ├ chunks/commons.f6669c.js                                   13.1 kB
  ├ chunks/framework.37f4a7.js                                 42.1 kB
  ├ chunks/main.c73430.js                                      6.62 kB
  ├ chunks/pages/_app.fb643d.js                                2 kB
  └ chunks/webpack.245f04.js                                   751 B
```

<br/>

---

<br/>

### Goober

A really light-weight solution, not very popular, but with a load of features.

Version: __`2.0`__ | Maintained by [Cristian Bote](https://twitter.com/cristianbote_) | Launched in __2019__ | [View Docs](https://goober.js.org/) | ... [back to Overview](#overview)

<br />

- ✅ __Styles/Component co-location__
- ✅ __Built-in Theming support__
- ✅ __TypeScript support__
- ✅ __Context-aware code completion__
- ❌ __Atomic CSS__

- __Styles definition method(s)__
  - ✅ Tagged Templates
  - ✅ Style Objects

- __Styles nesting__
  - ✅ Contextual styles
  - ✅ Abitrary nesting

- __Styles apply method(s)__
  - ✅ `className`
  - ✅ `styled` component (_see details below_)
  - 🟠 `css` prop (_is supported, but requires a separate babel plugin_)

- __Styles output__
  - ❌ `.css` file extraction
  - ✅ `<style>` tag injection

- 📉 __Low Learning curve__: if you're using only the `css` API with `className`, it's a low learning curve

<br />

#### Other benefits

- 🤏 really tiny
- 😎 it supports a very wide range of defining styles, so it's pretty versatile and full featured in this regard (however, I fear that having all these options, a large team could mix various ways of defining styles, so it's more difficult to _enforce consistency_)

<br />

#### Worth mentioning observations

- 😕 bundles nested styles even if they are not used in component
- 🤫 for user input styles, it generates a new class name for each update, but it does NOT remove the old ones, appending indefinitely to the DOM
- 🤔 don't know how to split static and dynamic styles, resulting in highly polluted duplicated styles in head for component variants
- 😱 it creates a single `<style>` tag with all the styles, and appends to it on update, and apparently it doesn't use `insertRule()`, not even in production builds, which might be an important performance drawback in large & highly dynamic UIs

<br />

#### Conclusions

Looking at Goober you cannot ask yourself what kind of magic did Cristian Bote do to fit all the features inside this tiny library. It is really mind blowing. It is marketed as being _"less than 1KB"_, which is not entirely accurate, but still... it's the smallest library we've tested.

<br />

|                 | Transferred / gzipped | Uncompressed |
| :-------------- | --------------------: | -----------: |
| Runtime library |                1.6 kB |       2.8 kB |
| Index page size |               73.7 kB |       208 kB |
| vs. CSS Modules |           __+2.2 kB__ |    __+7 kB__ |

<br />

```
Page                             Size     First Load JS
┌ ○ /                            3.89 kB        68.4 kB
├   /_app                        0 B            64.6 kB
├ ○ /404                         3.03 kB        67.6 kB
└ ○ /other                       2.49 kB          67 kB
+ First Load JS shared by all    64.6 kB
  ├ chunks/commons.7af247.js     13.1 kB
  ├ chunks/framework.9d5241.js   41.8 kB
  ├ chunks/main.03531f.js        6.62 kB
  ├ chunks/pages/_app.8a4776.js  2.37 kB
  └ chunks/webpack.50bee0.js     751 B
```

<br />

## Libraries not included

We know there are a lot of other libraries out there, besides the ones covered above. We're only covered the ones that have support for **React**, support for **SSR**, an easy integration with **Next.js**, good **documentation** and a sense of ongoing **support and maintenance**. Please checkout our [goals](#goals).

<br />

### style9

[Style9](https://github.com/johanholmerin/style9) is a new library, inspired by Facebook's own CSS-in-JS solution called stylex. Style9 is unique because it's the only open source library that supports both `.css` static extraction + atomic CSS. It has TS support and easy to integrate with Next.js.

However, it has quite a few limitations (at least as of Feb 2021) that makes it practically unusable in a real production application that we would want to scale, both in code & team size:

- CSS properties are a bit weirdly typed and you also have to learn a few proprietary and opinionated rules:
   - `fontSize`, or `borderRadius` must be `string`, so you cannot use `fontSize: "2em";`, for instance;
   - `padding`/`margin` supports `string`, but expands it automatically, so you cannot use `margin: "0 auto";`, for instance;
- TS types are custom, and a bit limited:
   - `boxShadow` is not supported, for instance;
- no Media Queries support (at least with TypeScript), which is a **total deal breaker** for us;
- cannot use design tokens defined as `Enum` or `POJO`, only constants are supported, which is a **smaller deal breaker** for us;
- dynamic styles are cumbersome to use:
   - it supports styles toggling, similar to `classNames` lib, but not dynamically/computed/expression based;
   - not sure if this is a limitation regarding static extraction (although Treat is doing it), or a TS poor typing limitation, or regarding atomic CSS class name generation;
   - there is an experimental addon [style9-components](https://github.com/johanholmerin/style9-components.macro) that tries to solve this;
- no support user styles, so we have to use inline styles;
- no global styles support (not a deal breaker for us);
- no theming support, as it doesn't handle dynamic styles very well (not a deal breaker for us):
   - there is some exploration in this regard, with [style9-theme](https://github.com/johanholmerin/style9-theme.macro);
- documentation is not comprehensive, it contains a lot of code comments, without code examples, making it even more difficult to follow & understand

Some pluses:
- it's the first lib we've tested that actually doesn't bundle unused styles;
- it doesn't allow arbitrary seletors / nesting, which is a good thing, because it enforces good practices and consistency;
- it is framework anostic;

As a conclusion, it wants to be a powerful solution with very interesting features, but it's not mature yet. As far as we see, it mostly designed towards more static solutions. Dynamic styling seems to be difficult to handle, and it's probably not the selling point.

<br />

### Aphrodite

It's not a popular solution, the approach is similar to **React Native StyleSheets**  way of styling components. Has built-in TypeScript support and a simple API.

- global styles are a bit cumbersome to define
- able to nest media queries & pseudo selectors, but cannot nest arbitrary rules/selectors
- no dynamic out-of-the-box support, so you have to get around that, like inline styles I guess, or like in React Native
- doesn't add any real value, except the ergonomics to colocate styles with the component.

### Glamor

I got it started with Next.js, but it feels fragile. The [Glamor official example](https://github.com/vercel/next.js/tree/canary/examples/with-glamor) throws an error regarding `rehydrate`. When commenting it out, it works, but not sure what the consequences are.

- it looks like an unmaintained or abandoned package
- documentation is pretty minimal
- lacks any TS support
- has a lot of documented experimental features, marked as "buggy"
- it feels like a side/internal project at FB, that is not used anymore.

### Linaria

Didn't manage to start it with Next.js + TypeScript.

It was an interesting solution, as it promises zero-runtime overhead, generating `.css` files at build time, while the style are collocated within the components.

### Cxs

Didn't manage to start it with Next.js + TypeScript. The [official example](https://github.com/vercel/next.js/tree/canary/examples/with-cxs) uses version 3, while today we have version 6. The example doesn't work, because the API has changed.

The solution looked interesting, because it is supposed to be very light-weight.

### Astroturf

Didn't manage to start it with Next.js + TypeScript. The [official example](https://github.com/vercel/next.js/tree/canary/examples/with-astroturf) uses an older version of Next.js.

The solution is not that popular, but it was the first to use `.css` extraction with collocated styles.

### Otion

Looks promising, atomic css and light-weight. It has a working [Next.js example](https://github.com/kripod/otion/tree/main/packages/example-nextjs), but we didn't consider it because it lacks any documentation.

### Styletron

It looks like a not so popular solution, which also lacks support for TypeScript. It looks like the maintainers work at Uber and they use it internally. It focused on generating unique atomic CSS classes, which could potentially deduplicate a lot of code.

### Radium

The project was put in [Maintenance Mode](https://formidable.com/blog/2019/radium-maintenance/). They recommend other solutions.

### Glamorous

The project was [discontinued](https://github.com/paypal/glamorous/issues/419) in favor of Emotion.

<br />

## Running the examples

Each implementation sits on their own branch, so we can have a clear separation at built time.

```bash
# install dependencies
yarn

# for development
yarn dev

# for production
yarn build
yarn start
```

<br />

## Feedback and Suggestions

To get in touch, my DMs are open [@pfeiffer_andrei](https://twitter.com/pfeiffer_andrei).

<br />

**Special thanks and appreciations** go to everyone that helped putting this document together, and making it more accurate:

- Martin Hochel ([@martin_hotell](https://twitter.com/martin_hotell))
- Oleg Isonen ([@oleg008](https://twitter.com/oleg008))
