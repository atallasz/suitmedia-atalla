<template>
  <div class="max-w-screen-xl mx-auto px-4 py-12">
    <div class="flex flex-wrap justify-between items-center mb-6">
      <p class="text-sm text-gray-700">
        Showing {{ totalItems > 0 ? startItem + 1 : 0 }} - {{ endItem }} of {{ totalItems }}
      </p>

      <div class="flex gap-4">
        <div class="flex items-center gap-2">
          <label for="perPage" class="text-sm text-gray-700">Show per page:</label>
          <select
            id="perPage"
            v-model.number="perPage"
            class="border rounded-full px-4 py-1 text-sm text-gray-700 focus:outline-none focus:ring-1 focus:ring-gray-400"
          >
            <option v-for="n in perPageOptions" :key="n" :value="n">{{ n }}</option>
          </select>
        </div>

        <div class="flex items-center gap-2">
          <label for="sortBy" class="text-sm text-gray-700">Sort by:</label>
          <select
            id="sortBy"
            v-model="sortBy"
            class="border rounded-full px-4 py-1 text-sm text-gray-700 focus:outline-none focus:ring-1 focus:ring-gray-400"
          >
            <option value="newest">Newest</option>
            <option value="oldest">Oldest</option>
          </select>
        </div>
      </div>
    </div>

    <div v-if="isLoading" class="text-center py-10">Loading...</div>
    <div v-else-if="allPosts.length === 0" class="text-center py-10 text-gray-500">No posts found.</div>
    <div v-else class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-6">
      <div
        v-for="post in allPosts"
        :key="post.id"
        class="bg-white rounded-lg shadow-md overflow-hidden"
      >
        <div class="w-full aspect-[4/3] bg-gray-200">
          <img
            :src="getMediumImageUrl(post)"
            :alt="post.title"
            class="w-full h-full object-cover"
            loading="lazy"
          />
        </div>
        <div class="p-4">
          <p class="text-xs text-gray-500 mb-2">
            {{ new Date(post.published_at).toLocaleDateString() }}
          </p>
          <h3 class="text-sm font-medium text-gray-800 line-clamp-2">
            {{ post.title }}
          </h3>
        </div>
      </div>
    </div>

    <div v-if="totalPages > 1" class="flex justify-center mt-10">
      <nav class="inline-flex items-center space-x-1">
        <button class="px-2 py-1 text-gray-400 hover:text-black" :disabled="currentPage === 1" @click="goToPage(1)">
          &laquo;
        </button>
        <button class="px-2 py-1 text-gray-400 hover:text-black" :disabled="currentPage === 1" @click="goToPage(currentPage - 1)">
          &lsaquo;
        </button>
        <button
          v-for="page in totalPages"
          :key="page"
          class="px-3 py-1 rounded-full"
          :class="page === currentPage ? 'bg-orange-500 text-white' : 'text-gray-600 hover:text-black'"
          @click="goToPage(page)"
        >
          {{ page }}
        </button>
        <button class="px-2 py-1 text-gray-400 hover:text-black" :disabled="currentPage === totalPages" @click="goToPage(currentPage + 1)">
          &rsaquo;
        </button>
        <button class="px-2 py-1 text-gray-400 hover:text-black" :disabled="currentPage === totalPages" @click="goToPage(totalPages)">
          &raquo;
        </button>
      </nav>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, watch, onMounted } from 'vue'
import axios from 'axios'

interface Post {
  id: number;
  title: string;
  published_at: string;
  medium_image: { url: string }[];
}

const allPosts = ref<Post[]>([])
const isLoading = ref(true)
const totalItems = ref(0) // Untuk menyimpan jumlah total item dari server

const perPageOptions = [10, 20, 50]
const perPage = ref(Number(localStorage.getItem('ideasPerPage')) || 10)
const sortBy = ref(localStorage.getItem('ideasSortBy') || 'newest')
const currentPage = ref(Number(localStorage.getItem('ideasCurrentPage')) || 1)

// Dihapus: `sortedPosts` & `paginatedPosts` karena sorting & pagination ditangani server

// Disesuaikan untuk menggunakan totalItems dari server
const totalPages = computed(() => Math.ceil(totalItems.value / perPage.value))
const startItem = computed(() => (currentPage.value - 1) * perPage.value)
const endItem = computed(() => Math.min(currentPage.value * perPage.value, totalItems.value))

const apiBaseUrl = 'https://suitmedia-backend.suitdev.com'

function goToPage(page: number) {
  if (page < 1 || page > totalPages.value || page === currentPage.value) return
  currentPage.value = page
  fetchPosts() // Ambil data untuk halaman baru
}

function formatImageUrl(path: unknown): string {
  if (!path || typeof path !== 'string') return '' // Fallback jika path tidak valid
  if (path.startsWith('http')) return path
  return `${apiBaseUrl}${path}`
}

// Diperbaiki: Menggunakan formatImageUrl untuk memastikan URL absolut
function getMediumImageUrl(post: Post): string {
  if (Array.isArray(post.medium_image) && post.medium_image.length > 0) {
    const path = post.medium_image[0].url
    return formatImageUrl(path)
  }
  return '' // Fallback jika tidak ada gambar
}

const fetchPosts = async () => {
  isLoading.value = true
  try {
    const response = await axios.get('/api/ideas', {
      headers: {
        'Accept': 'application/json',
      },
      params: {
        'page[number]': currentPage.value,
        'page[size]': perPage.value,
        append: ['small_image', 'medium_image'],
        sort: sortBy.value === 'newest' ? '-published_at' : 'published_at',
      },
    })
    // Mengambil data dan total item dari respons API
    allPosts.value = response.data.data
    // CATATAN: Asumsi API mengembalikan 'meta.total'. Sesuaikan jika berbeda.
    totalItems.value = response.data.meta.total 
  } catch (error) {
    if (axios.isAxiosError(error)) {
      console.error('Axios error:', error.message)
      if (error.response) {
        console.error('Status:', error.response.status)
        console.error('Data:', error.response.data)
      }
    } else {
      console.error('Unknown error:', error)
    }
    // Reset state jika terjadi error
    allPosts.value = []
    totalItems.value = 0
  } finally {
    isLoading.value = false
  }
}

onMounted(() => {
  fetchPosts()
})

watch([perPage, sortBy], () => {
  currentPage.value = 1
  fetchPosts()
})

watch([perPage, sortBy, currentPage], () => {
  localStorage.setItem('ideasPerPage', perPage.value.toString())
  localStorage.setItem('ideasSortBy', sortBy.value)
  localStorage.setItem('ideasCurrentPage', currentPage.value.toString())
})
</script>

<style scoped>
.line-clamp-2 {
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
}
</style>