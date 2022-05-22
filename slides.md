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