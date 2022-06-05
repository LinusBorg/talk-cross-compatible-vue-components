---
theme: vuetiful
layout: cover
info: |
  ## Bridging the Gap - Building cross-compatible Vue Components with confidence
  Presentation slides for Vuejs Amsterdam, June 2nd-3rd 2022.
# persist drawings in exports and build
download: '/slides-export.pdf'
drawings:
  persist: false
---

# Bridging the Gap

Building cross-compatible Components for Vue 2&3 with confidence

---
layout: section
---
# What is "the gap"?

---
layout: full-image
title: Canyon in need of a bridge
image: ./assets/canyon1.jpeg

---
<style>
  div.cliff-item {
    position: absolute;
  
  }
</style>
<div class="cliff-item top-[60px] right-[150px] text-vgreen text-4xl transform -rotate-6">Vue 2</div>
<div v-click class="cliff-item top-[300px] left-[90px] text-vgreen text-6xl transform -rotate-6">Vue 3</div>

<div v-click>
  <div class="cliff-item top-[140px] right-[350px] text-vgreen text-3xl transform -rotate-12">
    breaking changes
  </div>
  <div class="cliff-item top-[190px] right-[420px] text-vgreen text-3xl transform rotate-6">
    dependency conflicts
  </div>
  <div class="cliff-item top-[250px] right-[390px] text-vgreen text-3xl transform -rotate-7">
    publishing strategies
  </div>
  <div class="cliff-item top-[320px] right-[420px] text-vgreen text-3xl transform rotate-8">
    Testing issues
  </div>
  <div class="cliff-item top-[380px] right-[370px] text-vgreen text-3xl transform -rotate-6">
    code duplication
  </div>
</div>

---
layout: big-points
titleRow: true
title: We have to...
---

<v-clicks>

1. Write <span class="text-vgreen">compatible code</span> that respects breaking changes
2. Write <span class="text-vgreen">tests</span> that can be run against both versions
3. Manage <span class="text-vgreen">conflicting dependencies</span>
4. Decide how to <span class="text-vgreen">bundle and publish</span> the project

</v-clicks>

---
preload: false
clicks: 6
---

# Introducing Vue-Bridge

<VueBridgeContent/>

<div 
  v-if="$slidev.nav.clicks === 6"
  v-motion-pop
  class="absolute top-[200px] left-[200px] transform -rotate-12 text-3xl p-10 border border-red-500 bg-red-400/80 rounded-lg"
>

This is still under heavy development

</div>

---
layout: big-points
titleRow: true
title: "Vue 2.7 will make this easier"
---

* `Vue 2.7-alpha` was released this week
* Supports Composition API in Vue 2
* `vue-demi` / `@vue/composition-api` not required anymore
* allows optimization of a few things in VueBridge

<teenyicons-arrow-right-circle-outline  class="stroke-vgreen" viewBox="-1 -1 17 17"/> It might be worth to wait until `2.7` drops


---
layout: section
---

# 1. Writing compatible code

---
cols: '1-1'
titleRow: true
title: "Example: v-model on Components"
---

## Vue 2
```js{all|3,7}
export default {
  props: {
    value: String
  },
  methods: {
    handleInput(e) {
      this.$emit('input', e.target.value)
    }
  }
}
```

::right::

## Vue 3
```js{all|3,5,8}
export default {
  props: {
    modelValue: String
  },
  emits: ['update: modelValue'],
  methods: {
    handleInput(e) {
      this.$emit('update:modelValue', e.target.value)
    }
  }
}
```

---
cols: '2-1'
titleRow: true
title: "Example: v-model - Manual Fix"
---

```js{all|6-9}
export default {
  props: {
    modelValue: String
  },
  emits: ['update: modelValue'],
  model: {
    prop: 'modelValue',
    event: 'update:modelValue'
  },
  methods: {
    handleInput(e) {
      this.$emit('update:modelValue', e.target.value)
    }
  }
}
```

::right::

* Manual work every time
* Easy to miss
* Tricky with Typescript

---
cols: '2-1'
titleRow: true
title: "Example: v-model - Vue-Bridge helper"
---

```js{all|1,5,7}
import {defineComponent } from '@vue-bridge/runtime'

export default defineComponent({
  props: {
    modelValue: String
  },
  emits: ['update: modelValue'],
  methods: {
    handleInput(e) {
      this.$emit('update:modelValue', e.target.value)
    }
  }
})
```
::right::

* Write component for Vue 3 (future-proof)
* `@vue-bridge/runtime` adds model config
* works fine with TS

---
layout: section
---

# Unit testing against both Versions

---
layout: big-points
titleRow: true
title: Challenges for unit Testing
---

* Two versions of `@vue/test-utils` required (`v1` vs. `v2`)
* Subtle API differences between those versions

<p v-click class="text-vgreen">How to run the same tests against two different versions?</p>

---
cols: '1-1'
titleRow: true
title: Example - Unit test
---

### Vue 3 (test-utils `v2`)

```js{all|5,8-12}
import { mount } from '@vue/test-utils'

test('displays message', () => {
  const wrapper = mount(MessageComponent, {
    props: {
      msg: 'Hello world'
    },
    global: {
      provide: {
        store: mockStore
      }
    }
  })

  // Assert the rendered text of the component
  expect(wrapper.text()).toContain('Hello world')
})
```

::right::

### Vue 2 (test-utils `v1`)

```js{all|5,8-10}
import { mount } from '@vue/test-utils'

test('displays message', () => {
  const wrapper = mount(MessageComponent, {
    propsData: {
      msg: 'Hello world'
    },
    provide: {
      store: mockStore
    }
  })

  // Assert the rendered text of the component
  expect(wrapper.text()).toContain('Hello world')
})
```

---
cols: '1-1'
titleRow: true
title: Example - Unit test
---

### Using `@vue-bridge/testing`

<div v-show="$slidev.nav.clicks === 0">

```diff{1-2}
- import { mount } from '@vue/test-utils'
+ import { mount } from '@vue-bridge/testing'

test('displays message', () => {
  const wrapper = mount(MessageComponent, {
    props: {
      msg: 'Hello world'
    },
    global: {
      provide: {
        store: mockStore
      }
    }
  })

  // Assert the rendered text of the component
  expect(wrapper.text()).toContain('Hello world')
})
```

</div>

<div v-click>

```js
import { mount } from '@vue-bridge/testing'


test('displays message', () => {
  const wrapper = mount(MessageComponent, {
    props: {
      msg: 'Hello world'
    },
    global: {
      provide: {
        store: mockStore
      },
    }
  })

  // Assert the rendered text of the component
  expect(wrapper.text()).toContain('Hello world')
})
```

</div>

::right::

<div class="flex h-full justify-center pt-24">
  <ul class="!list-none">
    <li><teenyicons-arrow-up-circle-outline  class="stroke-vgreen" viewBox="-1 -1 17 17"/> Write in <code>v2</code> style for Vue 3</li>
    <li><teenyicons-arrow-up-circle-outline  class="stroke-vgreen" viewBox="-1 -1 17 17"/> Options transformed for Vue 2</li>
    <li><teenyicons-arrow-up-circle-outline  class="stroke-vgreen" viewBox="-1 -1 17 17"/> TS Support</li>
    <li><teenyicons-arrow-up-circle-outline  class="stroke-vgreen" viewBox="-1 -1 17 17"/> More tricks in the docs later</li>
  </ul>
</div>


---
layout: section
---

# Build-Time Optimizations

---
cols: '1-1'
titleRow: true
title: Build-time features
---

```js{all|3,8-9}
import { defineConfig } from 'vite'
import { createVuePlugin } from 'vite-plugin-vue2'
import { vueBridge } from '@vue-bridge/vite-plugin'

export default defineConfig({
  plugins: [
    createVuePlugin(),
    vueBridge({
      vueVersion: '2',
    }),
  ],
})
```

::right::

* Support for version-specific style blocks
* Support for version-specific imports
* alias trickery for easier source-sharing in monorepos

---
cols: '1-1'
titleRow: true
title: Version-specific styles
---

```html{all|2,6-8,11-13}
<template>
  <div class="root-wrapper">
    <slot />
  </div>
</template>
<style scoped v2>
  .root-wrapper /deep/ .class-in-slot {
    color: red;
  }
</style>
<style scoped v3>
  .root-wrapper  :deep(.class-in-slot) {
    color: red;
  }
</style>
```

::right::

* Different deep selectors
* Different transition styles
* escape hatch for various style fixes

---
cols: 1-1
titleRow: true
title: Version-specific imports
---

```js
import Teleport from './custom/teleport.ts?v-bridge'
```

<div class="mt-8" v-click>

```js
// Compiled for Vue 3
import Teleport from './custom/teleport.vue3.ts'

```

</div>
<div class="mt-8" v-click>

```js
// Compiled for Vue 2
import Teleport from './custom/teleport.vue2.ts'
```

</div>

::right::

<ul>
<li>
  put version-specific code into dedicated <code>*.vueX.js</code> files
</li>
<li v-if="$slidev.nav.clicks >=1">
  import with <code>?v-bridge</code> query param
</li>
<li v-if="$slidev.nav.clicks >=1">
  <code>@vue/bridge/vite-plugin</code> will resolve file matching the target
</li>
</ul> 

---
layout: section
---

# Managing conflicting dependencies
---
cols: 1-1
titleRow: true
title: Dependency issues in flat repositories
---

```json{all|8-9|5|10,11|11-12|8-9,12|14-16}
{
  "name": "my-vue-package",
  "dependencies": {
    "lodash": "^4.17.21",
    "@vueuse/core": "^8.5.0"
  },
  "devDependencies": {
    "vue": "^3.2.32",
    "vue2": "npm:vue@^2.6.14",
    "@vitejs/plugin-vue": "^2",
    "vite-plugin-vue2": "^2",
    "vue-template-compiler": "^2.6.14",
  },
  "peerDependencies": {
    "vue": "^2.6 || ^3.2"
  }
}
```

::right::

Flat repositories are bad because:

* they require package aliases
* ...which can lead to peer Dependency issues on install
* Packages relying on vue-demi make process cumbersome

<div v-click class="text-3xl mt-10 text-center">
  A better solution: workspaces
</div>

---
cols: 1-1
titleRow: true
title: Clean Dependencies with Workspaces (1)
---

```bash{all|15-17|8-13|3-6}
# Folder Structure

lib-vue2/
├─ package.json
├─ vite.config.js
├─ src/                 # --> Symlink to /lib-vue3/src

lib-vue3/
├─ package.json
├─ vite.config.js
├─ src/
  ├─ index.js
  ├─ MyComponent.vue

package.json
pnpm-workspaces.yaml
vite.config.shared.js   # shared build config
```

---
cols: 1-1
titleRow: true
title: Clean Dependencies with Workspaces (2)
---

```bash{15|3,4}
# Folder Structure

lib-vue2/
├─ package.json
├─ vite.config.js
├─ src/                 # --> Symlink to /lib-vue3/src

lib-vue3/
├─ package.json
├─ vite.config.js
├─ src/
  ├─ index.js
  ├─ MyComponent.vue

package.json
pnpm-workspaces.yaml
vite.config.shared.js   # shared build config
```

::right::

<div v-show="$slidev.nav.clicks >= 1">

```json{all|2|8-11,14|all}
{
  "name": "vue2-my-package",
  "dependencies": {
    "lodash": "^4.17.21",
    "@vueuse/core": "^8.5.0"
  },
  "devDependencies": {
    "@vue/test-utils": "^1.0.0",
    "vite-plugin-vue2": "^2",
    "vue": "^2.6.14",
    "vue-template-compiler": "^2.6.14",
  },
  "peerDependencies": {
    "vue": "^2.6"
  }
}
```
</div>

---
cols: 1-1
titleRow: true
title: "Clean Dependencies with Workspaces (3)"
---

```bash{8,9}
# Folder Structure

lib-vue2/
├─ package.json
├─ vite.config.js
├─ src/                 # --> Symlink to /lib-vue3/src

lib-vue3/
├─ package.json
├─ vite.config.js
├─ src/
  ├─ index.js
  ├─ MyComponent.vue

package.json
pnpm-workspaces.yaml
vite.config.shared.js   # shared build config
```

::right::

<div v-show="$slidev.nav.clicks >= 1">

```json{all|2|8-10,13|all}
{
  "name": "vue3-my-package",
  "dependencies": {
    "lodash": "^4.17.21",
    "@vueuse/core": "^8.5.0"
  },
  "devDependencies": {
    "@vitejs/plugin-vue": "^2",
    "@vue/test-utils": "^2.0.0-0",
    "vue": "^3.2.34",
  },
  "peerDependencies": {
    "vue": "^3.2.34"
  }
}
```

</div>

---
layout: section
---

# Bundling & Publishing 

---
cols: '1-1'
titleRow: true
title: Bundling and Publishing as two Packages
---

```js
// Vue 3
import { MyComponent } from 'vue3-my-package'
// Vue 2
import { MyComponent } from 'vue2-my-package'
```
<div v-click>

```bash
lib-vue3
├─ dist/
  ├─ index.js # Vue 3 Bundle
├─ package.json
lib-vue2
  dist/
  ├─ index.js # Vue 2 Bundle
├─ package.json
```

</div>
<div v-click>

```json
{
  "name": "vue{2,3}-my-package"
  "exports": {
    ".": {
      "import": "./dist/index.js"
    },
  }
}
```

</div>

::right::

<div class="flex h-full justify-center pt-24">

<ul class="!list-none">
    <li><teenyicons-arrow-up-circle-outline  class="stroke-vgreen" viewBox="-1 -1 17 17"/> Dedicated bundles for Vue 2 and Vue 3</li>
    <li><teenyicons-arrow-up-circle-outline  class="stroke-vgreen" viewBox="-1 -1 17 17"/> Separate packages for Vue 2 and Vue 3</li>
    <li><teenyicons-arrow-up-circle-outline  class="stroke-vgreen" viewBox="-1 -1 17 17"/> Clean separation of dependencies</li>
    <li><teenyicons-arrow-up-circle-outline  class="stroke-vgreen" viewBox="-1 -1 17 17"/> straightforward generation of type declarations</li>
  </ul>

</div>

---
cols: '1-1'
titleRow: true
title: Bundling and Publishing as one Package
---

```js
// Vue 3
import { MyComponent } from 'vue-my-package'
// Vue 2
import { MyComponent } from 'vue-my-package/vue2'
```
```bash
lib-vue3
  ├─ dist/
    ├─ index.js # Vue 3 Bundle
  dist-vue2/
    ├─ index.js # Vue 2 Bundle
  package.json
```
```json
{
  "name": "vue-my-package",
  "exports": {
    ".": {
      "import": "./dist/index.js"
    },
    "./vue2": "./dist-vue2/index.js"
  }
}
```

::right::

<div class="flex h-full justify-center pt-24">
  <ul class="!list-none">
    <li><teenyicons-arrow-up-circle-outline  class="stroke-vgreen" viewBox="-1 -1 17 17"/> Dedicated bundles for Vue 2 and Vue 3</li>
    <li><teenyicons-arrow-up-circle-outline  class="stroke-vgreen" viewBox="-1 -1 17 17"/> One package to install regardless of target version</li>
    <li><teenyicons-arrow-down-circle-outline  class="stroke-red-500" viewBox="-1 -1 17 17"/> Possibly more conflicts with <code>peerDependencies</code></li>
    <li><teenyicons-arrow-down-circle-outline  class="stroke-red-500" viewBox="-1 -1 17 17"/> Trickier to get right with type declarations</li>
  </ul>
</div>

---
layout: big-points
title: Template Demo
---

# Demo Time!

https://github.com/vue-bridge/template-monorepo

---
title: Docs
---
<VueBridgeDocs />

---
layout: big-points
titleRow: true
title: Vue-Bridge needs you!
---

* Add missing Documentation chapters
* Review & proof-read existing Documentation
* Provide additional Templates
* Contribute eslint rules
* Write proper end-to-end tests
* ... and so much more!

<a href="https://github.com/vue-bridge/vue-bridge/discussions">Become a contributor!</a>

---
layout: big-points
title: Links Collection
---

<div class="grid grid-rows-3 grid-cols-[150px,1fr] gap-y-8 gap-x-2">
  <span>Docs</span>
  <span>
    <a href="https://vue-bridge.docs.netlify.app" target="blank" rel="noopener">
      https://vue-bridge.docs.netlify.app
    </a>
  </span>
  <span>Repo</span>
  <span>
    <a href="https://github.com/vue-bridge/vue-bridge" target="blank" rel="noopener">
      https://github.com/vue-bridge/vue-bridge
    </a>
  </span>
  <span>Template</span>
  <span>
    <a href="https://github.com/vue-bridge/template-monorepo" target="blank" rel="noopener">
      https://github.com/vue-bridge/template-monorepo
    </a>
  </span>
  <span>Twitter</span>
  <span>
    <a href="https://twitter.com/VueBridge" target="blank" rel="noopener">
      https://twitter.com/VueBridge
    </a>
  </span>
</div>

---
layout: outro
preload: false
title: Outro
twitter: '@Linus_Borg'
repository: 'github.com/linusborg'
hostedSlides: 'https://github.com/LinusBorg/talk-cross-compatible-vue-components'
---

<div class="absolute left-12 top-[200px] right-12 text-center text-light-600">
  <p class="text-4xl !leading-[1.5em]">You made it! We're done!</p>
  <p class="text-4xl !leading-[1.5em]">Questions?</p>
</div>
<div 
  class="absolute bottom-6 right-6 w-[200px] p-3 bg-white light:bg-vblue rounded-lg bg-opacity-50 mr-0 light:bg-opacity-40 mr-0 text-vblue light:text-white"
  v-motion
  :initial="{ x: 250 }"
  :enter="{ x: 0, transition: { delay: 500 } }"
  >

<a href="https://www.sli.dev" target="blank" rel="noopener">
  <img src="/slidev-logo.png" alt="Slidev Logo" class="w-36">
</a>

<p class="text-sm !mt-0">This talk was built with Slidev</p>

<a class="text-sm" href="https://sli.dev" target="_blank" rel="noopener">https://sli.dev</a>

</div>