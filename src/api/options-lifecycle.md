# Options: Lifecycle

:::tip Note
All lifecycle hooks automatically have their `this` context bound to the instance, so that you can access data, computed properties, and methods. This means **you should not use an arrow function to define a lifecycle method** (e.g. `created: () => this.fetchTodos()`). The reason is arrow functions bind the parent context, so `this` will not be the component instance as you expect and `this.fetchTodos` will be undefined.
:::

## beforeCreate

- **类型：** `Function`

- **详细介绍：**

  Called synchronously immediately after the instance has been initialized, before data observation and event/watcher setup.

- **其他相关：** [Lifecycle Diagram](/guide/essentials/lifecycle.html#lifecycle-diagram)

## created

- **类型：** `Function`

- **详细介绍：**

  Called synchronously after the instance is created. At this stage, the instance has finished processing the options which means the following have been set up: data observation, computed properties, methods, watch/event callbacks. However, the mounting phase has not been started, and the `$el` property will not be available yet.

- **其他相关：** [Lifecycle Diagram](/guide/essentials/lifecycle.html#lifecycle-diagram)

## beforeMount

- **类型：** `Function`

- **详细介绍：**

  Called right before the mounting begins: the `render` function is about to be called for the first time.

  **This hook is not called during server-side rendering.**

- **其他相关：** [Lifecycle Diagram](/guide/essentials/lifecycle.html#lifecycle-diagram)

## mounted

- **类型：** `Function`

- **详细介绍：**

  Called after the instance has been mounted, where element, passed to [`app.mount`](/api/application.html#app-mount) is replaced by the newly created `vm.$el`. If the root instance is mounted to an in-document element, `vm.$el` will also be in-document when `mounted` is called.

  **This hook is not called during server-side rendering.**

- **其他相关：** [Lifecycle Diagram](/guide/essentials/lifecycle.html#lifecycle-diagram)

## beforeUpdate

- **类型：** `Function`

- **详细介绍：**

  Called when data changes, before the DOM is patched. This is a good place to access the existing DOM before an update, e.g. to remove manually added event listeners.

  **This hook is not called during server-side rendering, because only the initial render is performed server-side.**

- **其他相关：** [Lifecycle Diagram](/guide/essentials/lifecycle.html#lifecycle-diagram)

## updated

- **类型：** `Function`

- **详细介绍：**

  Called after a data change causes the virtual DOM to be re-rendered and patched.

  The component's DOM will have been updated when this hook is called, so you can perform DOM-dependent operations here. However, in most cases you should avoid changing state inside the hook. To react to state changes, it's usually better to use a [computed property](./options-state.html#computed) or [watcher](./options-state.html#watch) instead.

  **This hook is not called during server-side rendering.**

- **其他相关：** [Lifecycle Diagram](/guide/essentials/lifecycle.html#lifecycle-diagram)

## activated

- **类型：** `Function`

- **详细介绍：**

  Called when a kept-alive component is activated.

  **This hook is not called during server-side rendering.**

- **其他相关：**
  - [`<KeepAlive/>`](/guide/built-ins/keep-alive)

## deactivated

- **类型：** `Function`

- **详细介绍：**

  Called when a kept-alive component is deactivated.

  **This hook is not called during server-side rendering.**

- **其他相关：**
  - [`<KeepAlive/>`](/guide/built-ins/keep-alive)

## beforeUnmount

- **类型：** `Function`

- **详细介绍：**

  Called right before a component instance is unmounted. At this stage the instance is still fully functional.

  **This hook is not called during server-side rendering.**

- **其他相关：** [Lifecycle Diagram](/guide/essentials/lifecycle.html#lifecycle-diagram)

## unmounted

- **类型：** `Function`

- **详细介绍：**

  Called after a component instance has been unmounted. When this hook is called, all directives of the component instance have been unbound, all event listeners have been removed, and all child component instance have also been unmounted.

  **This hook is not called during server-side rendering.**

- **其他相关：** [Lifecycle Diagram](/guide/essentials/lifecycle.html#lifecycle-diagram)

## errorCaptured

- **类型：** `(err: Error, instance: Component, info: string) => ?boolean`

- **详细介绍：**

  Called when an error from any descendent component is captured. The hook receives three arguments: the error, the component instance that triggered the error, and a string containing information on where the error was captured. The hook can return `false` to stop the error from propagating further.

  :::tip
  You can modify component state in this hook. However, it is important to have conditionals in your template or render function that short circuits other content when an error has been captured; otherwise the component will be thrown into an infinite render loop.
  :::

  **Error Propagation Rules**

  - By default, all errors are still sent to the global `config.errorHandler` if it is defined, so that these errors can still be reported to an analytics service in a single place.

  - If multiple `errorCaptured` hooks exist on a component's inheritance chain or parent chain, all of them will be invoked on the same error.

  - If the `errorCaptured` hook itself throws an error, both this error and the original captured error are sent to the global `config.errorHandler`.

  - An `errorCaptured` hook can return `false` to prevent the error from propagating further. This is essentially saying "this error has been handled and should be ignored." It will prevent any additional `errorCaptured` hooks or the global `config.errorHandler` from being invoked for this error.

## renderTracked

- **类型：** `(e: DebuggerEvent) => void`

- **详细介绍：**

  Called when virtual DOM re-render is tracked. The hook receives a `debugger event` as an argument. This event tells you what operation tracked the component and the target object and key of that operation.

- **用途：**

  ```vue-html
  <div id="app">
    <button v-on:click="addToCart">Add to cart</button>
    <p>Cart({{ cart }})</p>
  </div>
  ```

  ```js
  const app = createApp({
    data() {
      return {
        cart: 0
      }
    },
    renderTracked({ key, target, type }) {
      console.log({ key, target, type })
      /* This will be logged when component is rendered for the first time:
      {
        key: "cart",
        target: {
          cart: 0
        },
        type: "get"
      }
      */
    },
    methods: {
      addToCart() {
        this.cart += 1
      }
    }
  })

  app.mount('#app')
  ```

## renderTriggered

- **类型：** `(e: DebuggerEvent) => void`

- **详细介绍：**

  Called when virtual DOM re-render is triggered. Similarly to [`renderTracked`](#rendertracked), receives a `debugger event` as an argument. This event tells you what operation triggered the re-rendering and the target object and key of that operation.

- **用途：**

  ```vue-html
  <div id="app">
    <button v-on:click="addToCart">Add to cart</button>
    <p>Cart({{ cart }})</p>
  </div>
  ```

  ```js
  const app = createApp({
    data() {
      return {
        cart: 0
      }
    },
    renderTriggered({ key, target, type }) {
      console.log({ key, target, type })
    },
    methods: {
      addToCart() {
        this.cart += 1
        /* This will cause renderTriggered call
          {
            key: "cart",
            target: {
              cart: 1
            },
            type: "set"
          }
        */
      }
    }
  })

  app.mount('#app')
  ```
