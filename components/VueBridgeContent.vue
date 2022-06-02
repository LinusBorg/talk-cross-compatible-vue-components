<script setup>
const initial = { y: 60, opacity : 0 }
const enter = { y: 0, opacity: 1 }
</script>
<template>
  <div class="grid grid-cols-2 grid-rows-3 w-full h-[400px] gap-5">
    <div v-click class="bg-light-400 dark:bg-dark-100 rounded-md p-4">
      <code class="!text-md p-1">@vue-bridge/runtime</code>
      <p>
        Polyfills some bridging code at runtime. Tiny (600b)
      </p>
    </div>
    <div v-click class="bg-light-400 dark:bg-dark-100 rounded-md p-4">
      <code class="!text-md p-1">@vue-bridge/testing</code>
      <p>
        Wrapper for Vue's test utils, providing unified API
      </p>
    </div>
    <div v-click class="bg-light-400 dark:bg-dark-100 rounded-md p-4">
      <code class="!text-md p-1">@vue-bridge/eslint-config</code>
      <p>
        Small set of helpful eslint rules
      </p>
    </div>
    <div v-click class="bg-light-400 dark:bg-dark-100 rounded-md p-4">
      <code class="!text-md p-1">@vue-bridge/vite-plugin</code>
      <p>
        Vite Plugin providing additional opitimizations at build time for edge cases
      </p>
    </div>
    <div v-click class="bg-light-400 dark:bg-dark-100 rounded-md p-4 col-span-2">
      Documentation & Reference Material
      <div v-if="$slidev.nav.clicks >= 4" class="flex justify-around">
        <div class="p-4" v-motion :initial="initial" :enter="enter" :delay="0">Project Templates</div>
        <div class="p-4" v-motion :initial="initial" :enter="enter" :delay="100">How-To Guides</div>
        <div class="p-4" v-motion :initial="initial" :enter="enter" :delay="200">Learning</div>
        <div class="p-4" v-motion :initial="initial" :enter="enter" :delay="300">Package References</div>
      </div>
    </div>
  </div>
</template>
<style lang="postcss"></style>