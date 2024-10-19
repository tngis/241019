<script setup lang="ts">
import { v4 as uuidv4 } from 'uuid'
import { useForm } from 'vee-validate'
import { toTypedSchema } from '@vee-validate/zod'
import * as z from 'zod'
import { Icon } from '@iconify/vue'
import { Textarea } from '@/components/ui/textarea'
import type { Chat } from '@/types/chat'
import type { AnswerReq } from '@/services/chat/wear'
import { useChatApi } from '@/services/chat'
import { useChatStore } from '~/store/chat'
import { copyToClip, idToken, useViewPort } from '~/services'
import { toast } from 'vue-sonner'
import { useGlobalStore } from '~/store/global'
import { cn } from '~/services/utils'

definePageMeta({
  middleware: ['auth'],
})

const route = useRoute()
const mode = computed(() => route.query.mode || 'reservation')
const { isMacOS } = useDevice()
const chatStore = useChatStore()
const globalStore = useGlobalStore()
const { sidebarOpened, detailsOpened } = storeToRefs(globalStore)
const { chatlogs, retrievedLogs } = storeToRefs(chatStore)

const formSchema = toTypedSchema(
  z.object({
    userInput: z.string(),
  }),
)

const { handleSubmit, setFieldValue } = useForm({
  validationSchema: formSchema,
})

const loading = ref<boolean>(false)
const chatContainerRef = ref<HTMLElement>()
const textareaRef = ref()
const showScrollButton = ref<boolean>(false)
const conversation_id = ref<string>(uuidv4().slice(0, 7))

const isMobile = computed(() => {
  const { isMobile } = useViewPort()
  return isMobile
})

onMounted(() => {
  document.addEventListener('keydown', handleDeleteCommand)
  document.addEventListener('keydown', handleFocusCommand)
  document.addEventListener('keydown', handleCopyCommand)
  const chatContainer = chatContainerRef.value
  if (chatContainer) {
    chatContainer.addEventListener('scroll', checkScroll)
  }
  setHelloMessage()
})

onBeforeUnmount(() => {
  document.removeEventListener('keydown', handleDeleteCommand)
  document.removeEventListener('keydown', handleFocusCommand)
  document.removeEventListener('keydown', handleCopyCommand)
  const chatContainer = chatContainerRef.value as HTMLElement
  if (chatContainer) {
    chatContainer.removeEventListener('scroll', checkScroll)
  }
})

watch(mode, async () => {
  await handleClear()
})

watch(loading, () => {
  scrollToBottom()
})

const sendChat = handleSubmit(async (values) => {
  const question = values.userInput
  idToken().catch((err) => {
    toast.error(err)
  })
  setFieldValue('userInput', '')
  loading.value = true

  await createBotMessage({ type: 'user', text: question })

  const payload = {
    conversation_id: conversation_id.value,
    question,
    stream: true,
  } as AnswerReq

  const loadingBotMessage = { type: 'bot' as 'bot' | 'user' | 'error', text: '', loading: true }
  await createBotMessage(loadingBotMessage)
  const loadingIndex = chatlogs.value.indexOf(loadingBotMessage)
  try {
    let msgFull = ''
    let body: any
    if (mode.value === 'reservation') {
      body = await useChatApi()
        .reservationAnswer(payload)
        .catch(async (err) => {
          await createBotMessage({ type: 'error', text: err, loading: false }, loadingIndex)
          return
        })
    } else {
      body = await useChatApi()
        .wearAnswer(payload)
        .catch(async (err) => {
          await createBotMessage({ type: 'error', text: err, loading: false }, loadingIndex)
          return
        })
    }

    const reader = body?.getReader()
    if (!reader) {
      return
    }
    if (loadingIndex === -1) {
      return
    }

    await createBotMessage({ type: 'bot', text: msgFull, loading: false }, loadingIndex)

    const decoder = new TextDecoder()
    let done = false
    while (!done) {
      const { value, done: readDone } = await reader.read()
      done = readDone
      if (value) {
        msgFull += decoder.decode(value)
        await createBotMessage({ type: 'bot', text: msgFull, loading: false }, loadingIndex)
      }
    }

    if (msgFull.length === 0) {
      await createBotMessage({ type: 'error', text: 'Token limit Error', loading: false }, loadingIndex)
    }
  } catch (err) {
    await createBotMessage({ type: 'error', text: err as string, loading: false }, loadingIndex)
  } finally {
    useChatApi()
      .retrieve({ id: conversation_id.value })
      .then((resp) => {
        retrievedLogs.value = resp
      })
    loading.value = false
  }
})

const createBotMessage = (chat: Chat, index?: number): Promise<void> => {
  return new Promise<void>((resolve, reject) => {
    try {
      if (index) {
        chatlogs.value[index] = chat
      } else {
        chatlogs.value.push(chat)
      }
      resolve()
    } catch (error) {
      reject(error)
    } finally {
      scrollToBottom()
    }
  })
}

const handleEnterCommand = (event: KeyboardEvent) => {
  const isCommand = isMacOS ? event.metaKey : event.ctrlKey
  if (event.key === 'Enter' && isCommand) {
    event.preventDefault()
    event.stopPropagation()
    sendChat()
  }
}

const handleDeleteCommand = async (event: KeyboardEvent) => {
  const isCommand = isMacOS ? event.metaKey : event.ctrlKey
  if (event.key === 'Backspace' && isCommand && event.shiftKey) {
    event.preventDefault()
    event.stopPropagation()
    await handleClear()
  }
}

const handleFocusCommand = (event: KeyboardEvent) => {
  if (event.key === 'Escape' && event.shiftKey) {
    if (textareaRef.value) {
      event.preventDefault()
      event.stopPropagation()
      textareaRef.value.setFocused()
    }
  }
}

const handleCopyCommand = (event: KeyboardEvent) => {
  const isCommand = isMacOS ? event.metaKey : event.ctrlKey
  if (event.key === 'c' && isCommand && event.shiftKey) {
    event.preventDefault()
    event.stopPropagation()
    copyToClip(chatlogs.value[chatlogs.value.length - 1].text).then(() => {
      toast.success('Copied to clipboard!')
    })
  }
}

const handleClear = (): Promise<void> => {
  return new Promise<void>((resolve, reject) => {
    try {
      if (!loading.value) {
        chatStore.$reset()
        conversation_id.value = uuidv4().slice(0, 7)
      }
      setHelloMessage()
      resolve()
    } catch (error) {
      reject(error)
    } finally {
      scrollToBottom()
    }
  })
}

const setHelloMessage = () => {
  chatlogs.value =
    mode.value === 'reservation'
      ? [
          {
            text: 'こんにちは！ AIスコアラー（ベータ版）です。 今日は千葉でゴルフ場を探しています。 最適なコースを見つけるために、いくつか質問させていただきます。まず、ユーザーのレベルを教えていただけませんか?​',
            type: 'bot',
            loading: false,
          },
        ]
      : [
          {
            text: 'こんにちは！ AIスコアラー（ベータ版）です。シャツを探しの方ですね。最適な商品を見つけるために、いくつか質問させていただきます。 まず、メンズとレディスのどちらを探していますか?​',
            type: 'bot',
            loading: false,
          },
        ]
}

const checkScroll = () => {
  if (chatContainerRef.value) {
    const atBottom = Math.ceil(chatContainerRef.value.scrollHeight - chatContainerRef.value.scrollTop) === chatContainerRef.value.clientHeight
    showScrollButton.value = !atBottom
  } else {
    showScrollButton.value = false
  }
}

const scrollToBottom = () => {
  if (chatContainerRef.value) {
    chatContainerRef.value.scrollTop = chatContainerRef.value.scrollHeight
  }
}

const toggleSidebar = () => {
  sidebarOpened.value = !sidebarOpened.value
}
</script>

<template>
  <div class="relative flex size-full flex-col space-y-2">
    <div class="flex w-full items-center justify-between">
      <div class="flex items-center gap-4 px-0 py-2 md:px-4">
        <Sheet>
          <SheetTrigger v-if="!sidebarOpened || isMobile" as-child class="block 2xl:hidden">
            <div>
              <Button size="icon" variant="outline" @click="toggleSidebar">
                <Icon icon="tabler:layout-sidebar-left-expand" />
              </Button>
            </div>
          </SheetTrigger>
          <SheetContent v-if="isMobile" side="left">
            <SheetTitle />
            <SheetDescription />
            <layout-sidebar />
          </SheetContent>
        </Sheet>
        <span class="text-xl font-bold">{{ mode === 'reservation' ? 'Reservation AI' : 'Wear AI' }}</span>
      </div>
      <HoverCard>
        <HoverCardTrigger>
          <Button
            variant="outline"
            :size="isMobile ? 'icon' : 'default'"
            :disabled="loading"
            @click="
              async () => {
                await handleClear()
              }
            "
          >
            <span v-if="!isMobile" class="mr-2 text-sm">Clear History</span>
            <Icon icon="tdesign:clear-formatting-1" />
          </Button>
        </HoverCardTrigger>
        <HoverCardContent class="flex gap-4 bg-background-secondary px-4 py-2" align="end">
          <div class="flex items-center justify-center space-x-[2px] text-sm text-input">
            <Icon icon="carbon:mac-shift" />
            <Icon icon="carbon:mac-command" />
            <Icon icon="mdi:clear-outline" />
          </div>
        </HoverCardContent>
      </HoverCard>
    </div>
    <div ref="chatContainerRef" class="flex-1 space-y-4 overflow-hidden overflow-y-auto scroll-smooth">
      <template v-for="(chat, idx) in chatlogs" :key="idx">
        <chat-user-container v-if="chat.type === 'user'" :message="chat.text" />
        <chat-bot-container v-if="chat.type === 'bot' || chat.type === 'error'" :message="chat.text" :error="chat.type === 'error'" :loading="chat.loading" />
      </template>
    </div>
    <div class="mx-auto flex max-w-lg items-center space-x-2 md:max-w-2xl">
      <Button v-if="showScrollButton" class="absolute bottom-24 z-10 bg-primary opacity-80" color="primary" size="icon" @click="scrollToBottom">
        <Icon icon="tabler:chevron-down" />
      </Button>
    </div>
    <layout-details :class="cn('-bottom-5 flex items-center justify-center xl:hidden')" />
    <form class="flex flex-col space-y-4" @submit.prevent="sendChat" @keydown.enter="handleEnterCommand">
      <FormField v-slot="{ componentField }" name="userInput">
        <FormItem>
          <FormControl>
            <Textarea ref="textareaRef" autofocus rows="1" :max-rows="4" v-bind="componentField" :disabled="loading">
              <template #append>
                <Button size="icon" :disabled="loading" @click="sendChat">
                  <Icon icon="carbon:send-alt" />
                </Button>
              </template>
            </Textarea>
          </FormControl>
          <FormMessage />
        </FormItem>
      </FormField>
    </form>
  </div>
</template>
