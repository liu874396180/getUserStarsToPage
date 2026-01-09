<script setup lang="ts">
import Fuse from 'fuse.js'

const allStars = inject('allStars')
const searchQuery = ref('')
const fuse = new Fuse(allStars.value, {
  keys: [
    { name: 'name', weight: 1 },
    { name: 'description', weight: 0.8 },
    { name: 'language', weight: 0.6 }
  ],
  threshold: 0.6,
  includeScore: true,
  minMatchCharLength: 1,
  useExtendedSearch: true
})

// Add new computed property for language counts
const languageCounts = computed(() => {
  const counts = {}
  allStars.value.forEach(star => {
    if (star.language) {
      counts[star.language] = (counts[star.language] || 0) + 1
    }
  })
  return counts
})

// Add selected language state
const selectedLanguage = ref('')

// Modify search results to include language filtering
const searchResults = computed(() => {
  let filtered = allStars.value
  
  // Apply language filter
  if (selectedLanguage.value) {
    filtered = filtered.filter(star => star.language === selectedLanguage.value)
  }
  
  // Apply search filter
  if (searchQuery.value && fuse) {
    const results = fuse.search(searchQuery.value)
    filtered = results.map(result => result.item)
  }
  
  return filtered || []
})

// Add language selection handler
const selectLanguage = (language: string) => {
  selectedLanguage.value = language === selectedLanguage.value ? '' : language
}

const openStarPage = (url: string) => {
  window.open(url)
}

const getMyStars = () => {
  window.open('https://github.com/liu874396180/getUserStarsToPage')
}

// 添加菜单状态控制
const isMobileMenuOpen = ref(false)

// 添加切换菜单方法
const toggleMobileMenu = () => {
  isMobileMenuOpen.value = !isMobileMenuOpen.value
}
</script>

<template>
  <div class="flex">
    <div
      class="sticky top-0 z-20 lg:m-4 group max-sm:w-full flex flex-col flex-shrink-0 lg:w-64 lg:group-[.is-open]:w-64 lg:group-[.is-open]:translate-x-0 -translate-x-full -ml-4 my-4 max-w-64 lg:translate-x-0 transition-all duration-500 ease-in-out bg-white/90 lg:bg-white/40 backdrop-blur-md rounded-2xl h-[calc(100vh-2rem)] border border-t-white/10 border-b-gray-200">
      <button
        class="absolute -right-4 top-4 lg:hidden w-11 h-11 flex items-center justify-center rounded-full bg-white text-black border hover:bg-white/80 transition-colors duration-200"><i
          class="ri-arrow-right-s-line text-xl"></i></button>
      <div class="lg:hidden flex items-center justify-between pl-6 -pr-12 pt-4"><span
          class="text-lg font-semibold">Menus</span></div>
      <div class="lg:hidden mx-4 m-4 relative"><button
          class="w-full px-4 py-3 text-sm font-medium rounded-full border bg-white/50 text-black border-gray-300 hover:bg-white/80 transition-colors duration-200 flex items-center justify-between"><span
            class="flex items-center"><i class="mr-2 apps"></i>Category</span><i
            class="ri-arrow-down-s-line transition-transform duration-200 "></i></button></div>
      <div class="overflow-y-auto flex-grow scrollbar-hide px-4 py-4">
        <ul class="space-y-2">
          <li @click="selectedLanguage = ''"
            :class="['text-gray-700 hover:text-black hover:bg-gray-100 group flex gap-x-2 rounded-xl px-3 py-2 text-sm leading-6 font-semibold cursor-pointer transition-all duration-100 ease-in-out',
            !selectedLanguage ? 'bg-white/80 text-black' : 'text-gray-500']">
            <i class="ri-apps-2-line"></i>
            <span class="flex-grow truncate">All Languages</span>
            <span class="text-xs ml-auto py-1 px-2 rounded-md bg-gray-50 border-gray-200 border">
              {{ allStars.length }}
            </span>
          </li>
          <li v-for="language in Object.keys(languageCounts)" :key="language"
            @click="selectLanguage(language)"
            :class="['text-gray-700 hover:text-black hover:bg-gray-100 group flex gap-x-2 rounded-xl px-3 py-2 text-sm leading-6 font-semibold cursor-pointer transition-all duration-100 ease-in-out',
            selectedLanguage === language ? 'bg-white/80 text-black' : 'text-gray-500']">
            <i class="ri-code-line"></i>
            <span class="flex-grow truncate">{{ language || 'Unknown' }}</span>
            <span class="text-xs ml-auto py-1 px-2 rounded-md bg-gray-50 border-gray-200 border">
              {{ languageCounts[language] }}
            </span>
          </li>
        </ul>
      </div>
    </div>
    <div class="flex flex-col">
      <div
        class="sticky top-0 z-10 flex flex-row my-4 justify-between items-center py-3.5 px-4  bg-white/40 backdrop-blur-md rounded-2xl">
        <div class="flex items-center justify-center">
          <a href="/" class="flex items-center"><span class="text-2xl font-serif text-black">U Stars</span></a>
        </div>
        <div class="items-center min-w-96 md:flex hidden">
          <input placeholder="Search Github Name Or Description"
            class="text-sm w-full px-4 py-3 rounded-full border border-gray-200 focus:outline-none focus:ring-1 focus:ring-black"
            type="text" v-model="searchQuery">
        </div>
        <div class="flex items-center lg:gap-4 gap-2">
          <Button @click="getMyStars">
            Get U Stars
          </Button>
        </div>
      </div>
      <div class="text-center my-4 text-gray-600 text-lg font-light hidden">
        Discover and organize your GitHub stars with ease
      </div>
      <div class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-4">
        <div v-for="star in searchResults" :key="star.id"
          class="rounded-md p-2 sm:p-4 hover:p-3 flex flex-col border border-gray-200 hover:bg-white/80 hover:border transition-all duration-200 ease-in-out relative cursor-pointer bg-white/40"
          @click="openStarPage(star.html_url)">
          <div class="flex items-center justify-between">
            <div class="flex items-center">
              <span class="text-md font-medium">{{ star.name }}</span>
            </div>
            <div class="text-sm text-gray-500">{{ star.stargazers_count }}</div>
          </div>

          <div class="text-sm text-gray-500 mt-2 line-clamp-2 min-h-10">{{ star.description }}</div>

          <div class="flex items-center justify-end mt-2">
            <div class="text-sm text-gray-500">{{ star.language }}</div>
          </div>

          <!-- star chart -->
          <!-- <div class="flex items-center justify-center">
        <img :src="`https://starchart.cc/${star.full_name}.svg`" alt="star chart" class="w-full h-full">
      </div> -->
        </div>

        <div class="flex items-center justify-center" v-if="searchResults.length === 0">
          <div class="text-sm text-gray-500">No stars found</div>
        </div>
      </div>

    </div>
  </div>

</template>


<style scoped>
/* 隐藏滚动条 */
::-webkit-scrollbar {
  display: none;
}
</style>
