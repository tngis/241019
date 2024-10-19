<script lang="ts" setup>
import { cn } from '~/services/utils'
import { Icon } from '@iconify/vue'
import { useGlobalStore } from '~/store/global'

const globalStore = useGlobalStore()
const { sidebarOpened, detailsOpened } = storeToRefs(globalStore)

const { isMacOS } = useDevice()

onMounted(() => {
  document.addEventListener('keydown', handleToggleSidebarCommand)
  document.addEventListener('keydown', handleToggleDetailsCommand)
})

onBeforeUnmount(() => {
  document.removeEventListener('keydown', handleToggleSidebarCommand)
  document.removeEventListener('keydown', handleToggleDetailsCommand)
})

const handleToggleSidebarCommand = (event: KeyboardEvent) => {
  const isCommand = isMacOS ? event.metaKey : event.ctrlKey
  if (event.key === 's' && isCommand && event.shiftKey) {
    event.preventDefault()
    event.stopPropagation()
    sidebarOpened.value = !sidebarOpened.value
  }
}
const handleToggleDetailsCommand = (event: KeyboardEvent) => {
  const isCommand = isMacOS ? event.metaKey : event.ctrlKey
  if (event.key === '\\' && isCommand && event.shiftKey) {
    event.preventDefault()
    event.stopPropagation()
    detailsOpened.value = !detailsOpened.value
  }
}
</script>

<template>
  <div class="relative grid size-full min-h-screen grid-cols-12 gap-4 overflow-y-auto p-4 antialiased">
    <layout-sidebar v-if="sidebarOpened" class="col-span-3 hidden bg-background-secondary p-4 md:flex xl:col-span-2" />
    <div v-if="!sidebarOpened" class="absolute left-6 top-6 hidden 2xl:block">
      <Button size="icon" variant="outline" @click="sidebarOpened = true">
        <Icon icon="tabler:layout-sidebar-left-expand" />
      </Button>
    </div>
    <div
      :class="
        cn('h-full max-h-[calc(100vh-32px)]', !sidebarOpened ? 'col-span-12 mx-auto w-full max-w-4xl xl:max-w-7xl' : 'col-span-12 md:col-span-9 xl:col-span-7')
      "
    >
      <slot />
    </div>
    <layout-details
      :class="
        cn(
          'hidden h-full xl:col-span-3 xl:block',
          { 'absolute right-0 z-10 w-1/3 p-4 md:w-1/3 xl:w-1/5': !sidebarOpened },
          { 'h-auto bg-background/80': detailsOpened },
        )
      "
    />
    <layout-footer-menu />
  </div>
</template>
