<script setup>
import { computed } from 'vue';
import { images } from '../images'

const props = defineProps({
  image: {
    type: String,
    required: true
  }
})
const imageUrl = computed(() => `url(".${images[props.image].default}")`)
</script>
<template>
  <div class="slidev-layout bg-image h-full dark:text-dark-600 relative">
    <slot />
  </div>
</template>
<style scoped lang="postcss">
  .bg-image {
    background-image: v-bind(imageUrl);
    background-position: center center;
    background-size: cover;
  }
</style>