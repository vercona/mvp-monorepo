Just brainstorming, I need to think about this more. 

## Some potential Style Libs
- [solid styled](https://github.com/lxsmnsyc/solid-styled)
- [sfc also sort of exist](https://github.com/lxsmnsyc/solid-sfc/blob/main/examples/counter/src/App.solid.tsx)
- [vite css modules](https://github.com/Bluskript/vite-plugin-inline-css-modules)


## On Color Space:

[origin](https://bottosson.github.io/)

[why use OK](https://evilmartians.com/chronicles/oklch-in-css-why-quit-rgb-hsl)
[oklab() css](https://developer.mozilla.org/en-US/docs/Web/CSS/color_value/oklab)
[oklch() css](https://developer.mozilla.org/en-US/docs/Web/CSS/color_value/oklch)

palettes and gradients:
- [acerola palette generator](https://acerola.gg/colors.html)
- [gradient palette (be sure to select oklab)](https://colordesigner.io/gradient-generator)
- [advanced gradient generator](https://gradient.style/)

Oklab color pickers
- [advanced picker](https://oklch.com/)
- [simple](https://ok-color-picker.netlify.app/)
- [color wheel](https://observablehq.com/@shan/oklab-color-wheel)



## On Using Utility Classes

I am not a tailwind evangelist, do not like to sacrifice the power of css selectors and prefer css classes to be my method of reuse.

unocss example:
```jsx
function HelloWorld() {
  css`
    .wrapper {
      --uno: "ml-auto text-gray-800 dark:text-gray-100 group transition-all ease-in-out duration-500";
    }

    .child {
      --uno: "padding-y-when-small padding-x-when-small text-base gap-3 md:gap-6 m-auto p-4 md:py-3 flex transition-all ease-in-out duration-500"
    }

    /* Because its scoped we can do this without concern */
    [hidden=true] {
      &.wrapper { --uno: "no-sidebar-width" }
      &> .child { --uno: 'md:max-w-5xl lg:max-w-5xl xl:max-w-6xl' }
    }

    [hidden=false] {
      &.wrapper { --uno: "minus-width-280px" }
      &> .child { --uno: 'md:max-w-3xl lg:max-w-3xl xl:max-w-4xl' }
    }
  `;

  return (
    <div
      class="wrapper"
      :hidden="hideSideMenu"
    >
      <div class="child" />
    </div>
  )
}
```


## Depth Theming

This will likely be a collection of `SCSS` files to represent themes.

Each theme will be a class comprized of `.tier-{n}` sub-classes, as well as base variables.
Each `.tier-{n}` class is composed of a standard set of variables that are use throughout the frontend.

The benefit of this system is three fold:
1. Zero Thought Design - You shouldn't have to think much about color specifics while coding components.
2. Easy theming - update top level theme and all tier classes will update, then all their variables (great for user theming).
3. Easy to update/refactor - swapping the tier class on an element to update colors or edit theme variables
4. Easy to override - Should it be necessary, you can inline css variables, create custom tier, override with other classes etc.

Limitation:
1. at the component boundary, its hard to set an arbitrary depth and have the components children update their relative depths.
2. may be difficult to use 3rd part component libs and incorporate depth based styling

atm the best way to overcome the component boundary issue is to pass a depth prop and have children increment on it...
there is a benefit to handling depth manually in that you can choose when and where the depth is incremented.
For example, you may not want `<p></p>` to automatically increment.
may be possible to create a global [directive](https://docs.solidjs.com/references/api-reference/special-jsx-attributes/use_) or compile-time helper.

```scss
.theme-dark {
  /***   COLORS   ***/
  --color-red:   oklab(56.66% 0.24 0.11); // oklch(57.83% 0.2642 25.37)
  --color-white: oklab(100% 0 0); // oklch(100% 0 0);
  --color-base:  oklab(0% 0 0);  // oklch(0% 0 0);

  .success { //alert/error, warn, info, etc…
    … // still debating this
  }

  /***   TIERS   ***/
  .tier-0 {
    --default-bg: var(—color-base);
    --hover-bg: …
    --active-bg: …
    --disabled-bg: …
    //border, color, stroke, fill, etc.
  }

  .tier-2 {
    …
  }
  // to tier-n, such that tier-0 has contrast over tier-n in case you want to cycle depths via mod n


  // can maybe auto apply styles to children? only works to depth-1 tho as it is implicit...
  .tier-0 > * { @apply .tier-1 }
  .tier-1 > * { @apply .tier-2 }

  // might be necessary to do (this can prob be automated with scss functions):
  .tier-1 > * > * { @apply .tier-3 }
  .tier-1 > * > * > * { @apply .tier-4 }
  // otherwise you cannot trivially style components deeply. only other way is @apply scss
  .deep-component-class {@apply .tier-3} // which would be a lot of redundant code
  // could use js to dynamically update comp children based in parents given depth
  //data-depth={depth+1}


  // the benefit of this approach is you can target when to increment depth
  .tier-2 {
    …
    &> .d\+1 { @apply .tier-3 }
    &> .d\+2 { @apply .tier-4 }
    &> .d\+3 { @apply .tier-5 }
  }
}
```

```jsx
function HelloWorld() {
  css`
    .wrapper {
      background-color: var(--default-bg)
    }

    .text {
      color: var(--default-text)
    }
  `;

  return (
    <div class="wrapper" :style="{"--default-bg": 'override color'}">
      <p class="text"> Text </p>
    </div>
  )
}

function PageLayout() {
  return (
    <div class="tier-0" style={{"--default-bg": 'override color'}}>
      <HelloWorld class="tier-1"/>
      <span class="tier-1"> <p class="tier-2"> Text </p> </span>
      <error class="alert"/>
    </div>
  )
}
```


## Component Theme Overrides

Here's some vue code on how I created component specific overrides. 
Not necessarily advocating for this, but was necessary for my use-case at the time
as we could not expose css selectors to users 
(
  I was able to scrape and compile `.theme-default` at build time
  so users would know what component variables exist
).
```html
<template>
  <div :class="[theme, 'theme-default']" :style="{"--default-bg": 'override color'}">
    <thing class="tier-1" />
    <error class="alert"/>
  </div>
</template>

<!-- Component Theme -->
<style lang="scss">
  /*NOTE: not scoped
    map variables to component specific version so they 
    can be overwritten by a user created theme for example.
    .custom-theme {
      --comp-error-bg: red
    }
  */
  .theme-default {
    /* comp stands for component, you'd pick something more descriptive */
    --comp-bg: var(—default-bg);
    --comp-thing-bg: var(—default-bg);
    --comp-error-bg: var(—default-bg)
  }
</style>

<!-- Component Styles -->
<style lang="scss" scoped>
  div {
    background: var(--comp-bg)
  }

  thing {
    background: var(--comp-thing-bg)
  }

  error {
    background: var(--comp-error-bg)
  }
</style>
```