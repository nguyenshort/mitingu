<template>
  <div class="h-full w-full relative flex flex-col">
    <div class="flex items-center mb-5">
      <h4 class="flex-shrink-0 text-[18px] text-gray-700 mb-0">
        {{ $t('chat') }}
      </h4>

      <button
          class="bg-gray-100 text-gray-500 text-[18px] p-1 rounded-full transform bottom-2 md:hidden relative ml-auto"
          @click="roomStore.toogleTab('chat')"
      >
        <i-material-symbols-close />
      </button>
    </div>
    <div class="relative flex-full overflow-y-auto scrollbar-hide" id="chat-tab" ref="listRef">

      <message-item
          v-for='message in messages'
          :key='message.id'
          :message='message'
          class='mb-5 last:mb-0'
      ></message-item>

    </div>

    <div class='bg-primary-50 rounded-lg mt-3'>

      <form class='w-full flex items-center px-3 py-2' @submit.prevent='submit'>
        <button class='text-gray-500 flex-shrink-0'>
          <i-uil-comment-image />
        </button>

        <input
            v-model='content'
            type='text'
            :placeholder='$t("writeMessage")'
            class='w-full px-2 ml-2 border-l focus:outline-0 bg-transparent text-sx'
        />

      </form>

    </div>

  </div>
</template>

<script lang="ts" setup>
import { useRTDB } from '@vueuse/firebase/useRTDB'
import { MessageDocument } from '@entities/message'
import {v4 as uuidv4} from "uuid"

const roomStore = useRoomStore()
const route = useRoute()
const rawMessage = useRTDB<Record<string, MessageDocument>>(dbRef(getDatabase(), `room/${route.params.id}/chat`))

const messages = computed<MessageDocument[]>(() => {
  return Object.values(rawMessage.value ?? {}).sort((a, b) => a.createdAt - b.createdAt)
})

const userStore = useUserStore()

const content = ref('')
const loading = ref(false)

const submit = async () => {
  if (!content.value || loading.value) {
    return
  }

  loading.value = true
  try {

    const id = uuidv4()
    await dbSet(dbRef(getDatabase(), `room/${route.params.id}/chat/` + id), {
      id,
      user: {
        id: userStore.user?.id,
        name: userStore.user?.name,
        avatar: userStore.user?.avatar,
      },
      content: content.value,
      createdAt: Date.now()
    } as MessageDocument)

    content.value = ''
  } catch (e) {
    //
  }
  loading.value = false

}

const listRef = ref<HTMLDivElement>()
const audio = new Audio('/mp3/message-notification-sfx.mp3');
const count = ref(0)

watch(rawMessage, () => {
  nextTick(() => {
    if (listRef.value) {
      listRef.value.scrollTop = listRef.value.scrollHeight

      if(count.value > 0) {
        audio.play()
      }
      count.value++

    }
  })
})
</script>

<style scoped>
#chat-tab {
  height: calc(100% - 38px - 28px - 10px);
}
</style>
