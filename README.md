# mipd

> TypeScript Utilities for [EIP-6963: Multi Injected Provider Discovery](https://eips.ethereum.org/EIPS/eip-6963)

## Contents

- [Install](#install)
- [Utilities](#utilities)
  - [`createStore`](#createstore)
  - [`requestProviders`](#requestproviders)
  - [`announceProvider`](#announceprovider)
- [`window` Type Polyfill](#window-polyfill)
- [Types](#types)

## Install

```bash
npm i mipd
```

## Utilities

### createStore

```ts
import { createStore } from 'mipd'

const store = createStore()

// Subscribe to the MIPD Store.
store.subscribe(providers => {
  console.log(providers)
  // => [EIP6963ProviderDetail, EIP6963ProviderDetail, ...]
})

// Retrieve emitted Provider Details.
store.getProviders()
// => [EIP6963ProviderDetail, EIP6963ProviderDetail, ...]

// Find a Provider Detail.
store.findProvider({ rdns: 'com.example' })
// => EIP6963ProviderDetail | undefined

// Remove a Provider Detail.
store.removeProvider({ rdns: 'com.example' })

// Clear the store, including all Provider Details.
store.clear()

// Reset the store, and emit an event to request Provider Details.
store.reset()

// Destroy the store, and remove all Provider Details and event listeners.
store.destroy()
```

#### React

```tsx
import { useSyncExternalStore } from 'react'
import { createStore } from 'mipd'

const store = createStore()

function Example() {
  const providers = useSyncExternalStore(store.subscribe, store.getProviders)
  // ...
}
```

#### Svelte

```html
<script lang="ts">
  import { createStore } from 'mipd'
  import { readable } from 'svelte/store'
  const store = createStore()
  const providers = readable(store.getProviders(), store.subscribe)
</script>

<!-- ... -->
```

#### Vue

```html
<script setup lang="ts">
  import { reactive } from 'vue'
  import { createStore } from 'mipd'

  const store = createStore()
  const state = reactive({ providers: store.getProviders() })
  store.subscribe(providers => (state.providers = providers))
</script>

<!-- ... -->
```

### requestProviders

```ts
import { requestProviders } from 'mipd'

let providers = []

const unsubscribe = requestProviders(providerDetail => providers.push(providerDetail))
```

### announceProvider

```ts
import { announceProvider } from 'mipd'

const unsubscribe = announceProvider({
  info: {
    icon: 'https://example.com/icon.png',
    name: 'Example',
    rdns: 'com.example',
    uuid: '00000000-0000-0000-0000-000000000000'
  },
  provider: new EIP1193Provider()
})
```

## `window` Polyfill

By importing the `mipd/window` Polyfill, the types on `window.addEventListener` will be inferred to include the `EIP6963AnnounceProviderEvent` and `EIP6963RequestProviderEvent` types.

```ts
import 'mipd/window'

window.addEventListener(
  'eip6963:announceProvider'
  // ^? 'eip6963:announceProvider' | 'eip6963:requestProvider' | 'click' | ...

  event => {
  // ^? EIP6963AnnounceProviderEvent

    event.type
    //    ^? 'eip6963:announceProvider'
    event.detail
    //    ^? EIP6963ProviderDetail
    event.detail.info
    //           ^? EIP6963ProviderInfo
    event.detail.provider
    //           ^? EIP1193Provider
  }
)

window.addEventListener(
  'eip6963:requestProvider'
  // ^? 'eip6963:announceProvider' | 'eip6963:requestProvider' | 'click' | ...

  event => {
  // ^? EIP6963RequestProviderEvent

    event.type
    //    ^? 'eip6963:requestProvider'
  }
)
```

## Types

### EIP6963ProviderDetail

Event detail from `eip6963:announceProvider`.

#### Import

```ts
import type { EIP6963ProviderDetail } from 'mipd'
```

#### Definition

```ts
export interface EIP6963ProviderDetail<TProvider = Config['EIP1193Provider']> {
  info: EIP6963ProviderInfo
  provider: TProvider
}
```

### EIP6963ProviderInfo

Metadata of the EIP-1193 Provider.

#### Import

```ts
import type { EIP6963ProviderInfo } from 'mipd'
```

#### Definition

```ts
export interface EIP6963ProviderInfo {
  icon: string
  name: string
  rdns?: ... | (string & {})
  uuid: string
}
```

### EIP6963AnnounceProviderEvent

Event type to announce an EIP-1193 Provider.

#### Import

```ts
import type { EIP6963AnnounceProviderEvent } from 'mipd'
```

#### Definition

```ts
export interface EIP6963AnnounceProviderEvent<TProvider = DefaultProvider>
  extends CustomEvent<EIP6963ProviderDetail<TProvider>> {
  type: 'eip6963:announceProvider'
}
```

### EIP6963RequestProviderEvent

Event type to request EIP-1193 Providers.

#### Import

```ts
import type { EIP6963RequestProviderEvent } from 'mipd'
```

#### Definition

```ts
export interface EIP6963RequestProviderEvent extends Event {
  type: 'eip6963:requestProvider'
}
```

## Configuration

In some cases you might want to tune the global configuration (ie. the `EIP1193Provider`). To do this, the following configuration options are available:

| Type                | Default                                         | Description            |
| ------------------- | ----------------------------------------------- | ---------------------- |
| `EIP1193Provider`   | `import('viem').EIP1193Provider`                | The EIP-1193 Provider. |
| `Rdns`              | `'com.enkrypt' \| 'io.metamask' \| (string & {})` | Deterministic identifier for the Provider in the form of an rDNS (Reverse Domain Name Notation) |

Configuration options are customizable using [declaration merging](https://www.typescriptlang.org/docs/handbook/declaration-merging.html). Extend the `Config` interface either directly in your code or in a `d.ts` file (e.g. `eip6963.d.ts`):

```ts
import type { EIP1193Provider } from './eip1193-provider'

declare global {
  interface Config {
    EIP1193Provider: EIP1193Provider
    Rdns: 'com.myapp' | 'com.example' | (string & {})
  }
}
```

## Authors

- [@jxom](https://github.com/jxom) (jxom.eth, [Twitter](https://twitter.com/jakemoxey))

## License

[MIT](/LICENSE) License
