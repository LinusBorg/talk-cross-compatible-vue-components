---
theme: vuetiful
layout: cover
info: |
  ## Bridging the Gap - Building cross-compatible Vue Components with confidence
  Presentation slides for Vuejs Amsterdam, June 2nd-3rd 2022.
# persist drawings in exports and build
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
3. Decide how to <span class="text-vgreen">bundle and publish</span> the project
4. Manage potentially <span class="text-vgreen">conflicting dependencies</span>

</v-clicks>

---
preload:false
---

# Introducing Vue-Bridge

<VueBridgeContent/>

<!-- 
 - Still incomplete / alpha stage
 - We Focus on runtime & testing libraries
 - More infos in the docs
-->

---
layout: section
---

# 1. Writing compatible code

---
cols: '1-1'
titleRow: true
title: "Example: Lifecycle names"
---

## Vue 2
```js
export default {
  created() {
    window.addEventListener('resize', this.handler)
  },
  beforeDestroy() {
    window.removeEventListener('resize', this.handler)
  }
}
```

::right::

## Vue 3
```js{all|5}
export default {
  created() {
    window.addEventListener('resize', this.handler)
  },
  beforeUnmount() {
    window.removeEventListener('resize', this.handler)
  }
}
```

---
cols: '2-1'
titleRow: true
title: "Example: Lifecycle Names - Manual Fix"
---

```js
export default {
  created() {
    window.addEventListener('resize', this.handler)
  },
  beforeDestroy() {
    window.removeEventListener('resize', this.handler)
  }
  beforeUnmount() {
    window.removeEventListener('resize', this.handler)
  }
}
```

::right::

* Manual work every time
* Duplication
* Error prone
* Tricky with Typescript

---
cols: '2-1'
titleRow: true
title: "Example: Lifecycle Names - Vue-Bridge helper"
---

```js
import {defineComponent } from '@vue-bridge/runtime'

export default defineComponent({
  created() {
    window.addEventListener('resize', this.handler)
  },
  beforeUnmount() {
    window.removeEventListener('resize', this.handler)
  }
})
```
::right::

* Converts hook for Vue 2 at runtime
* No duplication
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

```js{all|5}
import { mount } from '@vue/test-utils'

test('displays message', () => {
  const wrapper = mount(MessageComponent, {
    props: {
      msg: 'Hello world'
    }
  })

  // Assert the rendered text of the component
  expect(wrapper.text()).toContain('Hello world')
})
```

::right::

### Vue 2 (test-utils `v1`)

```js{all|5}
import { mount } from '@vue/test-utils'

test('displays message', () => {
  const wrapper = mount(MessageComponent, {
    propsData: {
      msg: 'Hello world'
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

# Managing conflicting dependencies
---
cols: 1-1
titleRow: true
title: Dependency issues in flat repositories
---

```json{all|8-9|5|10,11|11-12|9,12|14-16}
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
title: Clean Dependencies with Workspaces
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
title: Clean Dependencies with Workspaces
---

```bash{15}
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

```json{2|8-10,13|all}
{
  "name": "vu2-my-package",
  "dependencies": {
    "lodash": "^4.17.21",
    "@vueuse/core": "^8.5.0"
  },
  "devDependencies": {
    "vue": "^2.6.14",
    "vite-plugin-vue2": "^2",
    "vue-template-compiler": "^2.6.14",
  },
  "peerDependencies": {
    "vue": "^2.6"
  }
}



```

---
layout: section
---

# Bundling & Publishing 

---
cols: '1-1'
titleRow: true
title: Bundling and publish as two packages
---

```js
// Vue 3
import { MyComponent } from 'vue3-my-package'
// Vue 2
import { MyComponent } from 'vue2-my-package'
```

```bash
# Folder Structure
lib-vue3
├─ dist/
  ├─ index.js # Vue 3 Bundle
├─src/
  ├─ index.js
  ├─ MyComponent.vue
lib-vue2
  dist/
  ├─ index.js # Vue 2 Bundle
```
```json
{
  "exports": {
    ".": {
      "import": "dist/index.js"
    },
  }
}
```

::right::

<div class="flex h-full justify-center pt-24">

<ul class="!list-none">
    <li><teenyicons-arrow-up-circle-outline  class="stroke-vgreen" viewBox="-1 -1 17 17"/> Dedicated bundles for Vue 2 and Vue 3</li>
    <li><teenyicons-arrow-up-circle-outline  class="stroke-vgreen" viewBox="-1 -1 17 17"/> Separate packages for Vue 2 and Vue 3</li>
    <li><teenyicons-arrow-up-circle-outline  class="stroke-vgreen" viewBox="-1 -1 17 17"/> Clean separation of dependencies</li>
    <li><teenyicons-arrow-up-circle-outline  class="stroke-vgreen" viewBox="-1 -1 17 17"/> straightforward generation of type declarations</li>
    <li><teenyicons-arrow-down-circle-outline  class="stroke-red-500" viewBox="-1 -1 17 17"/>requires workspace setup</li>
  </ul>

</div>

---
cols: '1-1'
titleRow: true
title: Bundling and publishing as one Package
---

```js
// Vue 3
import { MyComponent } from 'vue-my-package'
// Vue 2
import { MyComponent } from 'vue-my-package/vue2'
```
```bash
dist/
├─ index.js # Vue 3 Bundle
dist-vue2/
├─ index.js # Vue 2 Bundle
src/
├─ index.js
├─ MyComponent.vue
```
```json
{
  "exports": {
    ".": {
      "import": "dist/index.js"
    },
    "./vue2": "dist-vue2/index.js"
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
    <li><teenyicons-arrow-down-circle-outline  class="stroke-red-500" viewBox="-1 -1 17 17"/>requires workspace setup</li>
  </ul>
</div>

---
title: Docs
---
<VueBridgeDocs />

---
layout: big-points
titleRow: true
title: We need Help!
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
</div>

---
layout: outro
preload: false
title: Outro
twitter: '@Linus_Borg'
repository: 'github.com/linusborg'
---

<div class="absolute left-12 top-[200px] right-12 text-center text-light-600">
  <p class="text-4xl !leading-[1.5em]">You made it! We're done!</p>
  <p class="text-4xl !leading-[1.5em]">Questions?</p>
</div>
<div 
  class="absolute bottom-6 right-6 w-[200px] p-3 bg-white rounded-lg bg-opacity-50 mr-0"
  v-motion
  :initial="{ x: 220 }"
  :enter="{ x: 0, transition: { delay: 1000 } }"
  >

<a href="https://www.sli.dev" target="blank" rel="noopener">
  <img src="/slidev-logo.png" alt="Slidev Logo" class="w-36">
</a>

<p class="text-sm !mt-0">This talk was built with Slidev</p>
</div>

---
cols: '1-1'
titleRow: true
title: No bundling - Publishing raw SFCs
---

```js
// Vue 3
import { MyComponent } from 'vue-my-package'
// Vue 2
import { MyComponent } from 'vue-my-package'
```

```bash
# Folder Structure
src/
├─ index.js
├─ MyComponent.vue
```

<code class="!text-xs">package.json</code>

```json
{
  "main": "src/index.js"
}
```

::right::

<div class="flex h-full justify-center pt-24">
  <ul class="!list-none">
    <li><teenyicons-arrow-up-circle-outline  class="stroke-vgreen" viewBox="-1 -1 17 17"/> No build test necessary (for JS)</li>
    <li><teenyicons-arrow-up-circle-outline  class="stroke-vgreen" viewBox="-1 -1 17 17"/> Can be published in one package</li>
    <li><teenyicons-arrow-down-circle-outline  class="stroke-red-500" viewBox="-1 -1 17 17"/> Adds SFC compilation overload to consuming app</li>
    <li><teenyicons-arrow-down-circle-outline  class="stroke-red-500" viewBox="-1 -1 17 17"/> Needs additional tooling when using TS</li>
    <li><teenyicons-arrow-down-circle-outline  class="stroke-red-500" viewBox="-1 -1 17 17"/> Unit tests for both Versions need trickery</li>
  </ul>
</div>

---
layout: big-points
title: Three Levels of support
titleRow: true
---

1. Have them handled for you
2. Have them be pointed out to you
3. Have resources to consult about them.

---

# Linting for compat issues

![Eslint rule error for invalid v-model:arg use](/eslint-vmodel.png)

vs:

![Eslint rule error for invalid v-model:arg use](/eslint-vbind-sync.png)

`@vue-bridge/eslint-config` preset uses rules from `eslint-plugin-vue` for both versions:
