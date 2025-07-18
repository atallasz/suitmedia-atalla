<template>
  <section class="relative w-full overflow-hidden h-[400px]">
    <div
      class="clip-skew relative w-full h-full"
      @scroll.passive="onScroll"
      ref="bannerRef"
    >
      <img
        :src="bannerImage"
        alt="Ideas Banner"
        class="w-full h-full object-cover grayscale brightness-50"
        :style="{ transform: `translateY(${imageOffset}px)` }"
      />
      <div
        class="absolute inset-0 flex flex-col items-center justify-center text-white z-10"
        :style="{ transform: `translateY(${textOffset}px)` }"
      >
        <h1 class="text-4xl font-bold">Ideas</h1>
        <p class="text-lg mt-2">Where all our great things begin</p>
      </div>
    </div>
  </section>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'

// Ganti ini dengan reactive prop dari CMS jika sudah terhubung
const bannerImage = ref(new URL('@/assets/images/logo/logouns.jpg', import.meta.url).href)

const imageOffset = ref(0)
const textOffset = ref(0)
const bannerRef = ref<HTMLElement | null>(null)

const onScroll = () => {
  if (!bannerRef.value) return
  const scrollY = window.scrollY || window.pageYOffset
  imageOffset.value = Math.max(0, scrollY * 0.3)
  textOffset.value = Math.max(0, scrollY * 0.6)
}

onMounted(() => {
  window.addEventListener('scroll', onScroll, { passive: true })
})
onUnmounted(() => {
  window.removeEventListener('scroll', onScroll)
})
</script>

<style scoped>
.clip-skew {
  clip-path: polygon(0 0, 100% 0, 100% 70%, 0% 95%);
  overflow: hidden;
}
</style>
