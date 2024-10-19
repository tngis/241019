<script setup lang="ts">
import type { HTMLAttributes } from 'vue'
import { useVModel } from '@vueuse/core'
import { cn } from '@/services/utils'
import { Icon } from '@iconify/vue'

const props = defineProps<{
  defaultValue?: string | number
  modelValue?: string | number
  type?: string
  class?: HTMLAttributes['class']
}>()

const emits = defineEmits<{
  (e: 'update:modelValue', payload: string | number): void
}>()
const showPassword = ref<boolean>(false)

const modelValue = useVModel(props, 'modelValue', emits, {
  passive: true,
  defaultValue: props.defaultValue,
})

const togglePasswordVisibility = () => {
  showPassword.value = !showPassword.value
}
</script>

<template>
  <div class="relative w-full">
    <input
      v-model="modelValue"
      :type="props.type === 'password' && !showPassword ? 'password' : 'text'"
      :class="
        cn(
          'flex h-9 w-full rounded-xl border border-input bg-transparent px-3 py-1 pr-10 text-sm shadow-sm transition-colors file:border-0 file:bg-transparent file:text-sm file:font-medium placeholder:text-muted-foreground focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-primary disabled:cursor-not-allowed disabled:opacity-50',
          props.class,
        )
      "
    />
    <button v-if="props.type === 'password'" type="button" class="absolute right-2 top-1/2 -translate-y-1/2 transform" @click="togglePasswordVisibility">
      <Icon :icon="showPassword ? 'tabler:eye' : 'tabler:eye-off'" />
    </button>
  </div>
</template>
