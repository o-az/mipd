<script setup lang="ts">
import { createStore } from 'mipd'
import { reactive } from 'vue'

const store = createStore()
const state = reactive({ providers: store.getProviders() })
store.subscribe((providers) => (state.providers = providers))
</script>

<template>
  <div>
    <pre>
      {{ state.providers }}
    </pre>
  </div>
</template>
