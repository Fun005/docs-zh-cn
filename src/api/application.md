# 应用实例 API  {#application-api}

## createApp()  {#createapp}

Creates an application instance. Expects the root component as the first argument, and optional props to be passed to the root component as the second argument.

- **Type**

  ```ts
  function createApp(rootComponent: Component, rootProps?: object): App
  ```

- **Example**

  With inline root component:

  ```js
  import { createApp } from 'vue'

  const app = createApp({
    /* root component options */
  })
  ```

  With imported component:

  ```js
  import { createApp } from 'vue'
  import App from './App.vue'

  const app = createApp(App)
  ```

- **See also:** [Guide - Creating a Vue Application](/guide/essentials/application.html)

## createSSRApp()

Creates an application instance in [SSR Hydration](/guide/scaling-up/ssr.html#client-hydration) mode. Usage is exactly the same as `createApp()`.

## app.mount()

Mounts the application instance in a container element. The first argument can either be a CSS selector (the first matched element will be used) or an actual DOM element. Returns the root component instance.

If the component has a template or a render function defined, it will replace any existing DOM nodes inside the container. Otherwise, if the runtime compiler is available, the `innerHTML` of the container will be used as the template.

In SSR hydration mode, it will hydrate the existing DOM nodes inside the container. If there are [mismatches](/guide/scaling-up/ssr.html#hydration-mismatch), the existing DOM nodes will be morphed to match the expected output.

For each app instance, `mount()` can only be called once.

- **Type**

  ```ts
  interface App {
    mount(rootContainer: Element | string): ComponentPublicInstance
  }
  ```

- **Example**

  ```js
  import { createApp } from 'vue'
  const app = createApp(/* ... */)

  app.mount('#app')
  ```

  Can also mount to an actual DOM element:

  ```js
  app.mount(document.body.firstChild)
  ```

## app.unmount()

Unmounts a mounted application instance. This will in turn trigger the unmount lifecycle hooks for all components in the application's component tree.

- **Type**

  ```ts
  interface App {
    unmount(): void
  }
  ```

## app.provide()

Provide a value that can be injected in all descendent components within the application. Expects the injection key as the first argument, and the provided value as the second. Returns the application instance itself.

- **Type**

  ```ts
  interface App {
    provide<T>(key: InjectionKey<T> | symbol | string, value: T): this
  }
  ```

- **Example**

  ```js
  import { createApp } from 'vue'

  const app = createApp(/* ... */)

  app.provide('message', 'hello')
  ```

  Inside a component in the application:

  <div class="composition-api">

  ```js
  import { inject } from 'vue'

  export default {
    setup() {
      console.log(inject('message')) // 'hello'
    }
  }
  ```

  </div>
  <div class="options-api">

  ```js
  export default {
    inject: ['message'],
    created() {
      console.log(this.message) // 'hello'
    }
  }
  ```

  </div>

- **See also:**
  - [Provide / Inject](/guide/components/provide-inject.html)
  - [App-level Provide](/guide/components/provide-inject.html#app-level-provide)

## app.component()  {#appcomponent}

Registers a global component if passing both a name string and a component definition, or retrieves an already registered one if only the name is passed.

- **Type**

  ```ts
  interface App {
    component(name: string): Component | undefined
    component(name: string, component: Component): this
  }
  ```

- **Example**

  ```js
  import { createApp } from 'vue'

  const app = createApp({})

  // register an options object
  app.component('my-component', {
    /* ... */
  })

  // retrieve a registered component
  const MyComponent = app.component('my-component')
  ```

- **See also:** [Component Registration](/guide/components/registration.html)

## app.directive()  {#appdirective}

Registers a global custom directive if passing both a name string and a directive definition, or retrieves an already registered one if only the name is passed.

- **Type**

  ```ts
  interface App {
    directive(name: string): Directive | undefined
    directive(name: string, directive: Directive): this
  }
  ```

- **Example**

  ```js
  import { createApp } from 'vue'

  const app = createApp({
    /* ... */
  })

  // register (object directive)
  app.directive('my-directive', {
    /* custom directive hooks */
  })

  // register (function directive shorthand)
  app.directive('my-directive', () => {
    /* ... */
  })

  // retrieve a registered directive
  const myDirective = app.directive('my-directive')
  ```

- **See also:** [Custom Directives](/guide/reusability/custom-directives.html)

## app.use()  {#appuse}

Installs a [plugin](/guide/reusability/plugins.html). Expects the plugin as the first argument, and optional plugin options as the second argument.

The plugin can either be an object with an `install()` method, or a directly a function (which itself will used as the install method). The options (second argument of `app.use()`) will be passed along to the plugin's install method.

When `app.use()` is called on the same plugin multiple times, the plugin will be installed only once.

- **Type**

  ```ts
  interface App {
    use(plugin: Plugin, ...options: any[]): this
  }
  ```

- **Example**

  ```js
  import { createApp } from 'vue'
  import MyPlugin from './plugins/MyPlugin'

  const app = createApp({
    /* ... */
  })

  app.use(MyPlugin)
  ```

- **See also:** [Plugins](/guide/reusability/plugins.html)

## app.mixin()  {#appmixin}

Applies a global mixin (scoped to the application). A global mixin applies its included options to every component instance in the application.

- **Type**

  ```ts
  interface App {
    mixin(mixin: ComponentOptions): this
  }
  ```

:::warning Not Recommended
Mixins are supported in Vue 3 mainly for backwards compatibility due to its wide-spread use in ecosystem libraries. Use of mixins, especially global mixins, should be avoided in application code.

For logic reuse, prefer [Composables](/guide/reusability/composables.html) instead.
:::

## app.version

Provides the version of Vue that the application was created with. This is useful inside [plugins](/guide/reusability/plugins.html), where you might need conditional logic based on different Vue versions.

- **Type**

  ```ts
  interface App {
    version: string
  }
  ```

- **Example**

  Performing a version check inside a plugin:

  ```js
  export default {
    install(app) {
      const version = Number(app.version.split('.')[0])
      if (version < 3) {
        console.warn('This plugin requires Vue 3')
      }
    }
  }
  ```

- **See also:** [Global API - version](/api/general.html#version)

## app.config  {#appconfig}

Every application instance exposes a `config` object that contains the configuration settings for that application. You can modify its properties (documented below) before mounting your application.

```js
import { createApp } from 'vue'

const app = createApp(/* ... */)

console.log(app.config)
```

## app.config.errorHandler

Assign a global handler for uncaught errors from the following sources:

- Component renders
- Event handlers
- Lifecycle hooks
- `setup()` function
- Watchers
- Custom directive hooks
- Transition hooks

The error handler receives three arguments: the error, the component instance that triggered the error, and an information string specifying the error source type.

- **Type**

  ```ts
  interface AppConfig {
    errorHandler?: (
      err: unknown,
      instance: ComponentPublicInstance | null,
      // `info` is a Vue-specific error info,
      // e.g. which lifecycle hook the error was thrown in
      info: string
    ) => void
  }
  ```

- **Example**

  ```js
  app.config.errorHandler = (err, instance, info) => {
    // handle error, e.g. report to a service
  }
  ```

## app.config.warnHandler  {#appconfigwarnhandler}

Assign a custom handler for runtime warnings from Vue. It receives the warning message as the first argument, the source component instance as the second argument, and a component trace string as the third.

This can be used to filter out specific warnings to reduce console verbosity. All Vue warnings should be addressed during development, so this is only recommended during debug sessions to focus on specific warnings among many, and should be removed once the debugging is done.

:::tip
Warnings only work during development, so this config is ignored in production mode.
:::

- **Type**

  ```ts
  interface AppConfig {
    warnHandler?: (
      msg: string,
      instance: ComponentPublicInstance | null,
      trace: string
    ) => void
  }
  ```

- **Example**

  ```js
  app.config.warnHandler = (msg, instance, trace) => {
    // `trace` is the component hierarchy trace
  }
  ```

## app.config.performance

Set this to `true` to enable component init, compile, render and patch performance tracing in the browser devtool performance/timeline panel. Only works in development mode and in browsers that support the [performance.mark](/) API.

- **Type**: `boolean`

- **See also:** [Guide - Performance](/guide/best-practices/performance.html)

## app.config.compilerOptions

Configure runtime compiler options. Values set on this object will be passed to the in-browser template compiler and affect every component in the configured app. Note you can also override these options on a per-component basis using the [`compilerOptions` option](/).

::: warning Important
This config option is only respected when using the full build (i.e. the standalone `vue.js` that can compile templates in the browser). If you are using the runtime-only build with a build setup, compiler options must be passed to `@vue/compiler-dom` via build tool configurations instead.

- For `vue-loader`: [pass via the `compilerOptions` loader option](/). Also see [how to configure it in `vue-cli`](/).

- For `vite`: [pass via `@vitejs/plugin-vue` options](/).
  :::

### app.compilerOptions.isCustomElement

Specifies a check method to recognize native custom elements. If a tag name matches this condition, Vue will render it as a native element instead of attempting to resolve it as a Vue component.

Native HTML and SVG tags don't need to be matched in this function - Vue's parser recognizes them automatically.

- **Type:** `(tag: string) => boolean`

- **Example**

  ```js
  // treat all tags starting with 'ion-' as custom elements
  app.config.compilerOptions.isCustomElement = (tag) => {
    return tag.startsWith('ion-')
  }
  ```

- **See also:** [Vue and Web Components](/guide/extras/web-components.html)

### app.compilerOptions.whitespace

Adjusts template whitespace handling behavior.

Vue removes / condenses whitespaces in templates to produce more efficient compiled output. The default strategy is "condense", with the following behavior:

1. Leading / ending whitespaces inside an element are condensed into a single space.
2. Whitespaces between elements that contain newlines are removed.
3. Consecutive whitespaces in text nodes are condensed into a single space.

Setting this option to `'preserve'` will disable (2) and (3).

- **Type:** `'condense' | 'preserve'`

- **Default:** `'condense'`

- **Example**

  ```js
  app.config.compilerOptions.whitespace = 'preserve'
  ```

### app.compilerOptions.delimiters

Adjusts the delimiters used for text interpolation within the template. This is typically used to avoid conflicting with server-side frameworks that also use mustache syntax.

- **Type:** `[string, string]`

- **Default:** `{{ "['\u007b\u007b', '\u007d\u007d']" }}`

- **Example**

  ```js
  // Delimiters changed to ES6 template string style
  app.config.compilerOptions.delimiters = ['${', '}']
  ```

### app.compilerOptions.comments

Adjusts treatment of HTML comments in templates.

By default, Vue will remove the comments in production. Setting this option to `true` will force Vue to preserve comments even in production. Comments are always preserved during development. This option is typically used when Vue is used with other libraries that rely on HTML comments.

- **Type:** `boolean`

- **Default:** `false`

- **Example**

  ```js
  app.config.compilerOptions.comments = true
  ```

## app.config.globalProperties

An object that can be used to register global properties that can be accessed on any component instance inside the application. This is a replacement of Vue 2's `Vue.prototype` which is no longer present in Vue 3. As with anything global, this should be used sparingly.

If a global property conflicts with a component’s own property, the component's own property will have higher priority.

- **Type**

  ```ts
  interface AppConfig {
    globalProperties: Record<string, any>
  }
  ```

- **Usage**

  ```js
  app.config.globalProperties.msg = 'hello'
  ```

  This makes `msg` available inside any component template in the application, and also on `this` of any component instance:

  ```js
  export default {
    mounted() {
      console.log(this.msg) // 'hello'
    }
  }
  ```

## app.config.optionMergeStrategies

An object for defining merging strategies for custom component options.

The merge strategy receives the value of that option defined on the parent and child instances as the first and second arguments, respectively.

- **Type**

  ```ts
  interface AppConfig {
    optionMergeStrategies: Record<string, OptionMergeFunction>
  }

  type OptionMergeFunction = (to: unknown, from: unknown) => any
  ```

- **Example**

  ```js
  const app = createApp({
    // option from self
    msg: 'Vue',
    // option from a mixin
    mixins: [
      {
        msg: 'Hello '
      }
    ],
    mounted() {
      // merged options exposed on this.$options
      console.log(this.$options.msg)
    }
  })

  // define a custom merge strategy for `msg`
  app.config.optionMergeStrategies.msg = (parent, child) => {
    return (parent || '') + (child || '')
  }

  app.mount('#app')
  // logs 'Hello Vue'
  ```
