<script setup lang="ts">
import type { HTMLAttributes } from 'vue'
import { useVModel } from '@vueuse/core'
import { cn } from '@/services/utils'

interface AutosizeTextareaProps {
  class?: HTMLAttributes['class']
  defaultValue?: string | number
  placeholder?: string
  disabled?: boolean
  modelValue?: string | number
  rows?: string | number
  maxRows?: number
  minRows?: number
}

const props = defineProps<AutosizeTextareaProps>()
const emits = defineEmits<{
  (e: 'update:modelValue', payload: string | number): void
}>()

const modelValue = useVModel(props, 'modelValue', emits, {
  passive: true,
  defaultValue: props.defaultValue,
})

const textAreaRef = ref<HTMLTextAreaElement | null>(null)
const triggerAutoSize = ref<string | number | undefined>(modelValue.value)
const isFocused = ref(false)

const maxRows = props.maxRows ?? Number.MAX_SAFE_INTEGER
const minRows = props.minRows ?? 1

const useAutosizeTextArea = () => {
  const init = ref(true)

  watch(triggerAutoSize, () => {
    if (textAreaRef.value) {
      const lineHeight = parseInt(getComputedStyle(textAreaRef.value).lineHeight || '20', 10)
      const offsetBorder = 2
      const minHeight = minRows * lineHeight
      const maxHeight = maxRows * lineHeight

      if (init.value) {
        textAreaRef.value.style.minHeight = `${minHeight + offsetBorder}px`
        if (maxHeight > minHeight) {
          textAreaRef.value.style.maxHeight = `${maxHeight}px`
        }
        init.value = false
      }
      textAreaRef.value.style.height = `${minHeight + offsetBorder}px`
      const scrollHeight = textAreaRef.value.scrollHeight
      if (scrollHeight > maxHeight) {
        textAreaRef.value.style.height = `${maxHeight}px`
      } else {
        textAreaRef.value.style.height = `${scrollHeight + offsetBorder}px`
      }
    }
  })
}

useAutosizeTextArea()

watch(modelValue, (newValue) => {
  triggerAutoSize.value = newValue
})

onMounted(() => {
  triggerAutoSize.value = modelValue.value
})

const setFocused = () => {
  isFocused.value = true
  textAreaRef.value?.focus()
}
defineExpose({ setFocused })
</script>

<template>
  <div :class="cn('flex flex-row items-center rounded-2xl border border-input bg-transparent px-3 py-2 shadow-sm', { 'ring-2 ring-primary': isFocused })">
    <slot name="prepend" />
    <textarea
      ref="textAreaRef"
      v-model="modelValue"
      :placeholder="props.placeholder ?? 'Type message here ...'"
      :rows="props.rows ?? 1"
      :disabled="disabled"
      :class="
        cn('flex w-full resize-none gap-y-2 bg-transparent text-sm placeholder:text-muted-foreground focus-visible:outline-none', props.class, {
          'cursor-not-allowed opacity-50': props.disabled,
        })
      "
      @focus="isFocused = true"
      @blur="isFocused = false"
    />
    <div>
      <slot name="append" />
    </div>
  </div>
</template>
