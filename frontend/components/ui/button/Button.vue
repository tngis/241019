<script setup lang="ts">
import type { HTMLAttributes } from 'vue'
import { Primitive, type PrimitiveProps } from 'radix-vue'
import { type ButtonVariants, buttonVariants } from '.'
import { cn } from '@/services/utils'

interface Props extends PrimitiveProps {
  variant?: ButtonVariants['variant']
  size?: ButtonVariants['size']
  class?: HTMLAttributes['class']
  loading?: boolean
}

const props = withDefaults(defineProps<Props>(), {
  as: 'button',
  loading: false,
})

const isLoading = ref(false)

const handleClick = () => {
  isLoading.value = true
  // Simulate an async operation
  // setTimeout(() => {
  //   isLoading.value = false
  // }, 2000)
}
</script>

<template>
  <Primitive :as="as" :as-child="asChild" :class="cn(buttonVariants({ variant, size }), 'flex gap-2', props.class)">
    <template v-if="props.loading">
      <!-- <OrigamiSpinner :is-loading="props.loading" /> -->
      <loading-spinner :size="20" :isLoading="props.loading" />
    </template>
    <slot v-else />
  </Primitive>
</template>
