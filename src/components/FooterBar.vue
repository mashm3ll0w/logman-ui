<script setup>
import { containerMaxW } from '@/config.js'
import BaseLevel from '@/components/BaseLevel.vue'

const year = new Date().getFullYear()
</script>

<template>
  <footer class="py-2 px-6" :class="containerMaxW">
    <BaseLevel>
      <div class="text-center md:text-left">
        <b>Log<span class="text-blue-500">Man</span></b>
        <span class="text-gray-500">&nbsp;&copy;{{ year }} — real-time log streaming</span>
      </div>
      <div class="md:py-2"></div>
    </BaseLevel>
  </footer>
</template>
