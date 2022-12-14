<template>
  <div
      ref="containerRef"
      class="grid-container w-full h-full overflow-y-auto scrollbar-hide relative z-10"
      :class="['_' + count]"
  >

    <div
        ref="layoutRef"
        class="dynamic-layout flex-wrap h-full justify-center items-center flex"
        :class="['_' + count, roomStore.mode]"
    >
      <div
          v-for="(item, index) in items"
          :key="item.id"
          class="relative item"
          :class="['_' + index]"
          :style="itemStyle"
      >
        <div class="w-full h-full p-2">
          <div class="w-full h-full bg-white rounded-lg overflow-hidden">
            <slot :item="item" :key="index"></slot>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script lang="ts" setup>
import {UID} from "agora-rtc-sdk-ng"

const roomStore = useRoomStore()

const props = defineProps<{
  items: (any & {
    uid: UID
  })[]
  active?: number
}>()

const count = computed(() => props.items.length)

const layoutRef = ref<HTMLDivElement>()
const containerRef = ref<HTMLDivElement>()
const { width: containerWidth, height: containerHeight } = useElementSize(containerRef)

const itemStyle = computed(() => {

  if(!containerHeight.value || !containerWidth.value) {
    return {}
  }

  if(roomStore.mode === 'flexible' && count.value >= 5) {
    /**
     * @param z số phần tử / hàng
     * @param range
     */
    const autoFit = (z: number, range: [number, number]): any => {
      // Tổng hiện tích
      const S = containerWidth.value * containerHeight.value
      // Số người/phần tử
      const n = props.items.length
      // chia lấy dư để tính bù trừ
      const remainder = n % z
      // số phần tử đã bù trừ
      // Flex sẽ gây xuống hàng
      const n2 = remainder > 0 ? n + z - remainder : n

      // điện tích mỗi phần tử
      const s = S / n2

      const _width = containerWidth.value / z
      const _height = s / _width

      const ratio = _width / _height

      // tính lại nếu cặp cạnh ko thoả điều kiện
      // sử dụng để quy và tăng số cột / hàng
      if(ratio >= range[0] && range[1] >= ratio) {
        return {
          S,
          s: s,
          n: n2,
          width: _width,
          height: _height
        }
      }
      return autoFit(z + 1, range)
    }

    const data = autoFit(1, [0.6, 1.5])

    return {
      height: data.height + 'px',
      width: data.width + 'px'
    }
  }

  if(count.value === 1) {
    if(containerWidth.value > containerHeight.value) {
      return {
        width: containerHeight.value * 12 / 7 + 'px',
        height: containerHeight.value + 'px',
      }
    }
    return {
      width: containerWidth.value + 'px',
      height: containerHeight.value + 'px',
    }
  } else if(count.value === 2) {
    // 2 chiều trên PC & đổi chiều trên mobile
    if(containerWidth.value > containerHeight.value) {
      return {
        height: containerHeight.value + 'px',
        width: (containerWidth.value / 2) + 'px'
      }
    } else {
      return {
        height: (containerHeight.value / 2) + 'px',
        width: containerWidth.value + 'px'
      }
    }
  } else {

    if(containerWidth.value <= 640) {
      const _sizeBase = containerWidth.value / 2
      return {
        width: _sizeBase + 'px',
        height: _sizeBase + 'px'
      }
    }

    //  chia 4 theo tỉ lệ 7/12
    if(containerWidth.value > containerHeight.value) {
      const _sizeBase = containerHeight.value / 2
      return {
        width: _sizeBase * 12 / 7 + 'px',
        height: _sizeBase + 'px'
      }
    } else {
      const _sizeBase = containerWidth.value / 2
      return {
        width: _sizeBase * 12 / 7 + 'px',
        height: _sizeBase + 'px'
      }
    }
  }
})

</script>
