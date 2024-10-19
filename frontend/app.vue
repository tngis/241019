<template>
  <NuxtLayout :name="isPublicLayout ? 'public' : 'default'">
    <NuxtPage />
  </NuxtLayout>
  <Toaster position="top-right" rich-colors />
</template>

<script setup lang="ts">
import { useRoute } from 'vue-router'
import NProgress from 'nprogress'

const nuxtApp = useNuxtApp()
const route = useRoute()

nuxtApp.hook('page:start', () => {
  NProgress.configure({ showSpinner: false })
  NProgress.start()
})

nuxtApp.hook('page:finish', () => {
  NProgress.done()
})

const isPublicLayout = computed(() => {
  return route.name
    ? route.name.toString().includes('login') || route.name.toString().includes('terms-conditions') || route.name.toString().includes('virtual-try-on')
    : false
})
</script>
