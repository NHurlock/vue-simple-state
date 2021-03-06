# vue-simple-state

Quickly create shared complex reactive state structures. Built using [@vue/reactivity](https://github.com/vuejs/vue-next/tree/master/packages/reactivity) and [rxjs](https://github.com/ReactiveX/rxjs)

<hr />

[![Build Status](https://img.shields.io/github/workflow/status/NHurlock/vue-simple-state/Node%20CI?style=flat-square)](https://github.com/NHurlock/vue-simple-state/actions)
[![Coverage Status](https://img.shields.io/codecov/c/github/NHurlock/vue-simple-state/master?style=flat-square)](https://codecov.io/github/NHurlock/vue-simple-state?branch=master)

[![npm](https://img.shields.io/npm/v/vue-simple-state?style=flat-square)](https://www.npmjs.com/package/vue-simple-state)
[![License](https://img.shields.io/npm/l/vue-simple-state?style=flat-square)](LICENSE)

## Install

```shell
npm install --save vue-simple-state
```
<br>

## Usage - `useState`

`useState` is meant to be used directly inside of a `setup` function in a `Vue` 3+ component, however, it can be used without. Read more about `setup` [in Vue's docs](https://v3.vuejs.org/guide/composition-api-setup.html). For use outside of a `Vue` 3+ component, use the `manualUnsub` option below (if `Vue` is installed, otherwise this is the default), or manually interact with `State` ([see Advanced Usage](#advanced-usage---state))

### `useState(config)` - Options
* `config {}` - (Optional) Configuration to override base configuration per-use
  * [see available config options and defaults in `config`](#usage---config)

### `state` - Direct read-only access to state
The `state` property is read-only direct access to the current state. `state` is used as a [computed](https://v3.vuejs.org/guide/reactivity-computed-watchers.html#computed-values) variable, its value will be accessed using the `value` property.
```javascript
import { useState } from 'vue-simple-state'

export default {
    // ...
    setup() {
        const { state } = useState()

        return {
            state
        }
    }
}
```

### `computed(fn)` - Create a read-only computed state variable using a function
Variables created using the `computed` method take a function to pull out a value from state. `computed` is used as a [computed](https://v3.vuejs.org/guide/reactivity-computed-watchers.html#computed-values) variable, its value will be accessed using the `value` property.
* `fn <(state) => any>` - Function used to extract value from state
```javascript
import { useState } from 'vue-simple-state'

export default {
    // ...
    setup() {
        const { computed } = useState()

        const userName = computed((state) => {
            return state.user.name
        })

        return {
            userName
        }
    }
}
```

### `reactive(path, default?)` - Create a read-only state variable by path
Variables created using the `reactive` method take a path to pull out a value from state and an optional default value if none is found. `reactive` is used as a [computed](https://v3.vuejs.org/guide/reactivity-computed-watchers.html#computed-values) variable, its value will be accessed using the `value` property.
* `path <string[]>` - Path by property name to access state value
* `default <any?>` - (Optional) Default value to return when state value is not set
```javascript
import { useState } from 'vue-simple-state'

export default {
    // ...
    setup() {
        const { reactive } = useState()

        const userName = reactive(['user', 'name'], 'Joe')

        return {
            userName
        }
    }
}
```

### `writable(path, default?)` - Create a read-write computed state variable by path
Variables created using the `writable` method take a path to pull out a value from state and an optional default value if none is found. `writable` is used as a read-write [computed](https://v3.vuejs.org/guide/reactivity-computed-watchers.html#computed-values) variable, its value will be accessed and set using the `value` property.
* `path <string[]>` - Path by property name to access state value
* `default <any?>` - (Optional) Default value to return when state value is not set
```vue
<template>
    <input v-model="userName" />
</template>

<script>
import { useState } from 'vue-simple-state'

export default {
    // ...
    setup() {
        const { writable } = useState()

        const userName = writable(['user', 'name'], 'Joe')

        return {
            userName
        }
    }
}
</script>
```

### `useNamespace(path)` - Create a namespace for state
When using state in components, splitting out state to group related information is essential. Namespaces share all of the same properties/methods as described above but they are scoped to the defined namespace. Namespaces return their own `useNamespace` method, allowing creation of a namespace off of another namespace
* `path <string[]>` - Path by property name to access state value
```javascript
import { useState } from 'vue-simple-state'

export default {
    // ...
    setup() {
        const { useNamespace } = useState()
        const { reactive } = useNamespace(['user'])

        const userName = reactive(['name'], 'Joe')

        return {
            userName
        }
    }
}
```

### `unsubscribe()` - Manually unsubscribe
This method only exists when configured with the `manualUnsub` option set to `true` (or when `Vue` 3+ is not installed). When called it will unsubscribe `useState` from any future state updates
```javascript
import { useState } from 'vue-simple-state'

const { state, unsubscribe } = useState({
    manualUnsub: true
})

// ...use state

// cleanup subscription
unsubscribe()
```
<br>

## Usage - `config`

### `config.set(config)` - Set application-level configuration
Any config set using this method will be used as the default config for the application. These settings can be overwritten per-use of `useState(config)`
* `config {}` - Application-level configuration
  * `manualUnsub <bool = false*>` - Choose to manually unsubscribe from state updates. Defaults to `false`.  When set to `true`, `useState` will return an `unsubscribe` method to call in order to unsubscribe manually. By default, `useState` will unsubscribe using the `onUnmounted` component lifecycle hook. 
    * *When `Vue` 3+ is not installed, this option defaults to `true` and will require `unsubscribe` to be called manually for cleanup
```js
import { config } from 'vue-simple-state'

config.set({
    // ...options
})
```

### `config.get()` - Get the current configuration being used
```js
import { config } from 'vue-simple-state'

const currentConfig = config.get()
```

### `config.reset()` - Reset configuration to default
```js
import { config } from 'vue-simple-state'

// manualUnsub === true

config.reset()

// manualUnsub === false
```
<br>

## Advanced usage - `State`

`State` is the service maintaining the [BehaviorSubject](https://rxjs-dev.firebaseapp.com/api/index/class/BehaviorSubject) that is being used to handle the application state

### `State.observable` - Direct read-only access to the observable
```javascript
import { State } from 'vue-simple-state'

const behaviorSubject = State.observable

behaviorSubject
    .pipe(...)
    .subscribe(fn)
```

### `State.update(fn)` - Update state using a function
* `fn <(state) => state>` - Function used to manipulate state and return an updated state
```javascript
import { State } from 'vue-simple-state'

State.update((state) => {
    state.property = 'value'
    return state
})
```

### `State.subscribe(fn)` - Subscribe a function to state changes
Returns a function that when called will unsubscribe the observer
* `fn <(state) => void>` - Function called each time state changes
```javascript
import { State } from 'vue-simple-state'

function onStateChange(state) {
    console.log('called on state updates', state)
}

const unsubscribe = State.subscribe(onStateChange)

// cleanup subscription
unsubscribe()
```
<br>

## Advanced Examples

### Reading/Writing state to localStorage
```javascript
import { State } from 'vue-simple-state'

function getSavedState() {
    return window.localStorage.getItem(...)
}

function setSavedState(state) {
    window.localStorage.setItem(...)
}

State.update(getSavedState)
State.subscribe(setSavedState)
```

### Creating a custom writable computed state variable
```javascript
import { computed } from 'vue'
import { State, useState } from 'vue-simple-state'

export default {
    // ...
    setup() {
        const { writable } = useState()

        // Given: state.tags === 'one,two'
        // Read tags as an Array: ['one', 'two']
        // Update tags in state as a String: 'one,two,X'
        const stateTags = writable(['tags'])
        const tags = computed({
            get: () => stateTags.value.split(','),
            set: (val) => (stateTags.value = val.join(','))
        })

        return {
            tags
        }
    }
}
```