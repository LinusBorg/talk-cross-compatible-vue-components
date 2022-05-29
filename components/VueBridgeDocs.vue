<script setup>
import { useOnline } from '@vueuse/core'

const isOnline = useOnline()
</script>
<template>
  <iframe v-if="isOnline" src="https://vue-bridge-docs.netlify.app" class="w-full h-full"></iframe>
  <img v-else alt="Fallback Image for VueBridge docs website" src="/vue-bridge-docs.png" class="w-full h-full">
</template>
<style lang="postcss"></style>