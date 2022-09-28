<template>
  <div class="user-media relative w-full h-full">
    <div class="w-full h-full bg-white rounded-lg overflow-hidden relative z-10">
      <div ref="videoRef" class="w-full h-full" />
    </div>

    <div class="z-20 absolute bottom-0 left-0 flex items-center transform translate-x-4 -translate-y-4 text-white bg-[#03030385] text-[12px] px-3 rounded-2xl py-1">
      <i-ri-user-4-fill />
      <span>
        {{ _userDocument?.name }}
      </span>
    </div>

  </div>
</template>

<script lang="ts" setup>
import {UserDocument} from "@entities/user";
import {UID} from "agora-rtc-sdk-ng";

const agoraStore = useAgoraStore()
const userStore = useUserStore()
const axios = useAxios()

interface MediaProtocol {
  play: (element?: HTMLElement) => void
}

const props = defineProps<{
  uid: UID
  userData?: UserDocument
  video?: MediaProtocol
  audio?: MediaProtocol
}>()

const _userDocument = ref<UserDocument|undefined>()

const videoRef = ref<HTMLDivElement>()

onMounted(() => nextTick(() => {
  props.video?.play(videoRef.value!)
}))

const debouncedRebuild = useDebounceFn(() => nextTick(() => props.video?.play(videoRef.value!)), 300)

watch(() => props.video, () => debouncedRebuild())

const getUserDetail = async () => {
  const index = agoraStore.mapRemoteUsers.findIndex(user => user.uid === props.uid)
  if (index !== -1) {
    const result: any = await axios.get('/smileeye/detailUser/' + props.uid)
    agoraStore.mapRemoteUsers[index].userData = result
    _userDocument.value = result
  }
}
onMounted(() => getUserDetail())

onMounted(() => nextTick(() => {
  props.audio?.play()
}))
watch(() => props.audio, () => props.audio?.play())

</script>

<style></style>