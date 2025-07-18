<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'
import { useRoute } from 'vue-router'

const showHeader = ref(true)
const lastScrollY = ref(window.scrollY)
const transparent = ref(false)
const route = useRoute()

const handleScroll = () => {
  const currentScrollY = window.scrollY
  transparent.value = currentScrollY > 0
  if (currentScrollY > lastScrollY.value && currentScrollY > 80) {
    showHeader.value = false // scroll down, hide
  } else {
    showHeader.value = true // scroll up, show
  }
  lastScrollY.value = currentScrollY
}

onMounted(() => {
  window.addEventListener('scroll', handleScroll)
})
onUnmounted(() => {
  window.removeEventListener('scroll', handleScroll)
})

const menus = [
  { name: 'Work', path: '/work' },
  { name: 'About', path: '/about' },
  { name: 'Services', path: '/services' },
  { name: 'Ideas', path: '/ideas' },
  { name: 'Careers', path: '/careers' },
  { name: 'Contact', path: '/contact' }
]
</script>

<template>
  <transition name="slide-down">
    <nav
      v-show="showHeader"
      :class="[
        'fixed top-0 left-0 z-50 w-full flex items-center px-12 h-[80px] transition-colors duration-300',
        transparent ? 'bg-[#EC6125]/80 backdrop-blur' : 'bg-[#EC6125]'
      ]"
    >
      <img
        src="@/assets/images/logo/suitmedia1.png"
        alt="Join Sistem Logo"
        class="h-20 w-auto"
      >
      <!-- Container menu, bisa digeser dengan mx-*, ml-*, mr-* sesuai kebutuhan -->
      <div class="flex-1 flex items-center">
        <ul class="flex gap-4 justify-start ml-[50rem] mt-2">
          <li v-for="menu in menus" :key="menu.path">
            <router-link
              :to="menu.path"
              :class="[
                'text-white relative pb-2 text-lg',
                route.path === menu.path
                  ? 'text-x-yellow border-b-2 border-x-yellow'
                  : 'border-b-2 border-transparent hover:border-white hover:text-white'
              ]"
              style="display: inline-block;"
            >
              {{ menu.name }}
            </router-link>
          </li>
        </ul>
      </div>
    </nav>
  </transition>
</template>

<style scoped>
.slide-down-enter-active,
.slide-down-leave-active {
  transition: transform 0.3s;
}
.slide-down-enter-from,
.slide-down-leave-to {
  transform: translateY(-100%);
}
</style>
