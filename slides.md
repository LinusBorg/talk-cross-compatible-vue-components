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
<div class="cliff-item top-[300px] left-[90px] text-vgreen text-6xl transform -rotate-6">Vue 3</div>
<div class="cliff-item top-[60px] right-[150px] text-vgreen text-4xl transform -rotate-6">Vue 2</div>

---
layout: big-points
title: List of Gaps
---

<v-clicks>

1. Writing <span class="text-vgreen">code</span> that respects breaking changes
2. Writing <span class="text-vgreen">tests</span> that run against both versions
3. Deciding how to <span class="text-vgreen">bundle and publish</span> the project
4. Managing potentially <span class="text-vgreen">conflicting dependencies</span>

</v-clicks>

---

# Introducing Vue-Bridge

---
layout: section
---

# 1. Writing the code

---
layout: big-points
title: Three Levels of support
titleRow: true
---

1. Have them handled for you
2. Have them be pointed out to you
3. Have resources to consult about them.

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

# Linting for compat issues

![Eslint rule error for invalid v-model:arg use](/eslint-vmodel.png)

vs:

![Eslint rule error for invalid v-model:arg use](/eslint-vbind-sync.png)

`@vue-bridge/eslint-config` preset uses rules from `eslint-plugin-vue` for both versions:


---
layout: section
---

# Unit testing against both Versions

---
layout: big-points
---
# Challenges for unit Testing

* Two versions of `@vue/test-utils` required (`v1` vs. `v2`)
* Subtle API diffferences between those versions

<p v-click>How to run the same tests against two different versions</p>

---
layout: section
---

# Bundling & Publishing 

---
cols: '1-1'
titleRow: true
title: No bundling - Publishing raw SFCs
---

Folder Structure:

```bash
src/
├─ index.js
├─ MyComponent.vue
```

`package.json`

```json
{
  "main": "src/index.js"
}
```

::right::

<div class="flex h-full justify-center pt-24">

+ -> No build test necessary (for JS)
+ -> Can be published in one package
+ -> Adds SFC compilation overload to consuming app 
+ -> Needs additional tooling when using TS

</div>

---
cols: '1-1'
titleRow: true
title: Bundling, publish as one Package
---

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
```js
// Vue 3
import { MyComponent } from 'vue-my-package'
// Vue 2
import { MyComponent } from 'vue-my-package/vue2'
```

::right::

<div class="flex h-full justify-center pt-24">

+ -> Dedicated bundles for Vue 2 and Vue 3
+ -> One package to install regardless of target version
+ -> Possibly more conflicts with `peerDependencies`
+ -> Trickier to get right with type declarations
+ -> requires workspace setup

</div>

---
cols: '1-1'
titleRow: true
title: Bundling, publish as two packages
---

```bash
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
```js
// Vue 3
import { MyComponent } from 'vue3-my-package'
// Vue 2
import { MyComponent } from 'vue2-my-package'
```

::right::

<div class="flex h-full justify-center pt-24">

+ -> Dedicated bundles for Vue 2 and Vue 3
+ -> Separate packages for Vue 2 and Vue 3
+ -> Clean separation of dependencies
+ -> requires workspace setup

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
* Convert vite-plugin into unplugin
* ... and so much more!

---
layout: big-points
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
titleRow: true
title: 
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