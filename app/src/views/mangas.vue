<template>
  <!-- continue reading -->
  <!-- recently released -->
  <!-- recently added -->
  <!-- trending mangas : genre -->
  <!-- trending mangas : author -->
  <!-- top unread -->
  <!-- recently readed  -->
  <div v-show="isLoaded" class="container mx-auto w-full px-8">
    <div v-show="mangaCount<1" class="flex h-screen flex-col items-center justify-center bg-white">
      <div class="text-center">
        <img src="/assets/images/empty.png" alt="Empty Shelf" class="mx-auto mb-6 h-52 w-52">
        <h1 class="mb-4 text-2xl font-semibold text-main-800">It's a Bit Empty Here!</h1>
        <p class="mb-6 text-main-600"> It seems like your manga collection is empty. Don't worry, I'm here to help you.
          <br> Just hit that 'Search Manga' button to get started. 📚✨.</p>
        <router-link to="/search" class="inline-block rounded-md bg-accent-500 px-6 py-3 text-lg text-white">Search a
          manga
        </router-link>
      </div>
    </div>
  </div>

  <div class="container mx-auto" v-if="mangaCount>0 && !storeIsLoading && isLoaded">
    <div class="sticky left-0 top-0 z-20 mx-auto mb-8 h-full w-full bg-white/80 px-8 pb-8 backdrop-blur-xl transition-all duration-100">
      <div class="grid w-full pt-4 sm:grid-cols-3 ">
        <div class="hidden sm:flex">
          <div class="flex items-center">
            <div class="flex-none">
              <p class="text-base"> Collection </p>
              <p class="text-xs tracking-tight"><b>{{ mangaCount }}</b> mangas </p>
            </div>
          </div>
        </div>
        <div>
          <nav class="navbar flex w-full justify-center align-start col-start-2 col-end-3" aria-label="tabs">
            <div class="sm:hidden">
              <label for="tabs" class="sr-only">Select a tab</label>
              <select id="tabs" name="tabs" @change="toggleTabs($event.target.selectedIndex)"
                      class="block w-full rounded-md border-main-300 py-2 pl-3 pr-10 text-base focus:border-accent-500 focus:outline-none focus:ring-accent-500 sm:text-sm">
                <option v-for="tab in tabs" :key="tab.name" :selected="tab.current">{{ tab.name }}</option>
              </select>
            </div>
            <div class="hidden sm:block">
              <div class="border-b border-main-200">
                <nav class="-mb-px flex space-x-8" aria-label="Tabs">
                  <a v-for="tab in tabs" :key="tab.name" href="#"
                     :class="[tab.current ? 'border-accent-500 text-accent-600' : 'border-transparent text-main-500 hover:border-main-300 hover:text-main-700', 'whitespace-nowrap border-b-2 px-1 py-4 text-sm font-medium']"
                     :aria-current="tab.current ? 'page' : undefined"
                     @click="toggleTabs(tab.index)">
                    {{ tab.name }}
                  </a>
                </nav>
              </div>
            </div>
          </nav>
        </div>
      </div>

      <div v-if="tabs[1].current" class="container mx-auto ">
        <Popover class="relative">
          <PopoverButton class="inline-flex items-center rounded-md bg-light-400 px-2 py-1 text-xs font-semibold text-main-600">
            <span>Filters</span>
            <div class="shrink-0 ml-2">
              <TBaseIcon :solid="true" name="AdjustmentsHorizontalIcon" aria-hidden="true" class="h-5 w-5 text-accent-700"/>
            </div>
          </PopoverButton>

          <transition
              enter-active-class="transition-opacity ease-out duration-300"
              enter-from-class="opacity-0"
              enter-to-class="opacity-100"
              leave-active-class="transition-opacity ease-in duration-200"
              leave-from-class="opacity-100"
              leave-to-class="opacity-0"
          >
            <PopoverPanel class="relative origin-top">

              <TBaseFilterWrapper
                  :row-on-xl="true"
                  class="rounded-b-lg bg-light-400 -top-[6px]"
                  @clear="clearFilter"
              >
                <TBaseInputGroup label="Genres">
                  <BaseMultiselect
                      v-model="filters.genres"
                      :options="itemGenres"
                      searchable
                      placeholder="Genres"
                      mode="tags"
                      @update:modelValue="onSearched"
                  />
                </TBaseInputGroup>

                <TBaseInputGroup label="Demographics">
                  <TBaseSelectInput
                      v-model="filters.types"
                      :options="itemPubDemographics"
                      searchable
                      :allow-empty="true"
                      placeholder="Demographics"
                      @click="onSearched"
                  />
                </TBaseInputGroup>
                <div class="relative flex gap-x-12 px-8 md:min-w-[512px]">
                  <TBaseInputGroup label="Publishing years" class="">
                    <div class="pb-6">
                      <vue-slider :data="mangasYears" v-model="filters.years" :value="filters.years" :contained="true"
                                  :enable-cross="false" :marks="mangasYearsMinMax" @change="onSearched"/>
                    </div>
                  </TBaseInputGroup>

                  <TBaseInputGroup label="Score" class="">
                    <div class="pb-6">
                      <vue-slider :data="mangasScores" v-model="filters.scores" :value="filters.scores"
                                  :contained="true"
                                  :enable-cross="false" :marks="mangasScoresMinMax" @change="onSearched"/>
                    </div>
                  </TBaseInputGroup>
                </div>
                <TBaseInputGroup label="Search filter" class="">
                  <TBaseInput placeholder="title, author, tags..." v-model="filters.searchText"
                              @input="onSearched()"></TBaseInput>
                </TBaseInputGroup>
                <TBaseInputGroup :label="`by ${filters.orderByField} (${filters.orderBy})`">
                  <div role="group" aria-label="First group">
                    <TBaseDropdown position="bottom-start" width-class="w-50" position-class="left-0">
                      <template #activator>
                        <TBaseButton variant="gray">
                          <TBaseIcon name="FunnelIcon" class="h-2 w-2"/>
                        </TBaseButton>
                      </template>

                      <div class="">
                        <TBaseDropdownItem class="rounded-md pt-3 hover:rounded-md">
                          <TBaseInputGroup class="-mt-3 font-normal">
                            <TBaseRadio
                                id="filter_title"
                                v-model="filters.orderByField"
                                label="Title"
                                size="sm"
                                name="filter"
                                value="canonicalTitle"
                                @update:modelValue="onSearched()"
                            />
                          </TBaseInputGroup>
                        </TBaseDropdownItem>
                      </div>
                      <div class="">
                        <TBaseDropdownItem class="rounded-md pt-3 hover:rounded-md">
                          <TBaseInputGroup class="-mt-3 font-normal">
                            <TBaseRadio
                                id="filter_score"
                                v-model="filters.orderByField"
                                label="Score"
                                size="sm"
                                name="filter"
                                value="score"
                                @update:modelValue="onSearched()"
                            />
                          </TBaseInputGroup>
                        </TBaseDropdownItem>
                      </div>
                      <div class="">
                        <TBaseDropdownItem class="rounded-md pt-3 hover:rounded-md">
                          <TBaseInputGroup class="-mt-3 font-normal">
                            <TBaseRadio
                                id="filter_publish_date"
                                v-model="filters.orderByField"
                                label="Publication Year"
                                size="sm"
                                name="filter"
                                value="startYear"
                                @update:modelValue="onSearched()"
                            />
                          </TBaseInputGroup>
                        </TBaseDropdownItem>
                      </div>
                    </TBaseDropdown>
                    <TBaseButton class="ml-1" variant="white" @click.prevent="sortData">
                      <TBaseIcon v-if="getSearchOrderBy" name="BarsArrowUpIcon" class="h-5"/>
                      <TBaseIcon v-else name="BarsArrowDownIcon" class="h-5"/>
                    </TBaseButton>
                  </div>
                </TBaseInputGroup>
              </TBaseFilterWrapper>
            </PopoverPanel>
          </transition>
        </Popover>
      </div>
    </div>

    <div v-if="tabs[0].current">

      <section class="px-8 pb-4" v-if="lastMangasRead?.length>0" >
        <TBaseCarousel uID="mbr" title="Continue Reading" :slides="lastMangasRead" :contentLoading="storeIsLoading"/>
      </section>
      <section class="px-8">
        <TBaseCarousel uID="mla" title="Recently Added" :slides="lastAddedMangas" :contentLoading="storeIsLoading"/>
      </section>
      <section class="px-8">
        <TBaseCarousel uID="mlp" title="Recently Published" :slides="lastPublishedMangas"
                       :contentLoading="storeIsLoading"/>
      </section>
      <section v-if="preferredGenres" class="px-8">
        <TBaseCarousel uID="mtg" :title="`Top mangas in ${preferredGenres?.name}`" :slides="topMangasByGenre"
                       :contentLoading="storeIsLoading"/>
      </section>
      <section class="px-8">
        <TBaseCarousel uID="tum" title="Top unread mangas" :slides="topUnreadMangas" :contentLoading="storeIsLoading"/>
      </section>
    </div>

    <div v-if="tabs[1].current">
      <div class="flex">
        <div class="grow">
          <TBaseGrid id="mangas" :key="gridKey">
            <TBasePosterCard v-for="(manga) in filteredMangas" :key="manga.id"
                             :id="`manga-${manga.canonicalTitle[0].toUpperCase()}`"
                             :to="helpers.getMangaRouterTo(manga)"
                             :title="manga.canonicalTitle"
                             :image="helpers.getMangaCover(manga.id, 240,360, 'cover', manga.state)"
                             :progress="manga.readProgress"
                             :tags="manga.genres"
                             :state="parseInt(manga.state)"
                             :contentLoading="parseInt(manga.state) === 1"
                             variant="primary"
            >
            </TBasePosterCard>
          </TBaseGrid>
        </div>
        <div
            class="fixed right-0 top-0 m-1 flex h-screen flex-col items-center justify-center space-y-1 text-accent-500 ">
          <a v-for="letter in filterAlphabet" :key="letter" @click.prevent="scrollTo(letter)" href="#"
             class="cursor-pointer text-sm hover:text-accent-700">
            {{ letter }}
          </a>
        </div>
      </div>
    </div>
  </div>
  <div class="block h-12"></div>

</template>
<script>
import { useMangaStore } from '@/stores/mangasStore'
import { useStatStore } from '@/stores/statsStore'
import helpers from '@/stores/utils'
import { Menu, MenuButton, Popover, PopoverButton, PopoverPanel } from '@headlessui/vue'
import { computed, onMounted, reactive, ref } from 'vue'
import { debounce } from 'lodash'
import { itemPubDemographics, itemGenres } from '@/global'
import { pageTitle } from '@/global.js'

const tabs = ref([
  { index: 0, name: 'Recommended', href: '#', current: true },
  { index: 1, name: 'Library', href: '#', current: false }
])

export default {
  name: 'Mangas',
  components: {
    Menu,
    MenuButton,
    Popover,
    PopoverButton,
    PopoverPanel
  },
  setup () {
    const filters = reactive({
      orderBy: 'asc',
      orderByField: 'canonicalTitle',
      searchText: null,
      yearsMarks: [],
      years: [],
      scores: [],
      types: [],
      genres: []
    })
    const isLoaded = ref(false)
    // #region new pinia implementation
    const statsStore = useStatStore()
    const stats = computed(() => statsStore.getStats)
    const mangasStore = useMangaStore()
    const storeIsLoading = computed(() => mangasStore.getIsLoading)
    const topUnreadMangas = computed(() => mangasStore.getMangaUnread)
    const lastMangasRead = computed(() => mangasStore.getMangasRead('primary'))
    const lastAddedMangas = computed(() => mangasStore.getLastAddedMangas('primary'))
    const lastPublishedMangas = computed(() => mangasStore.getLastPublishedMangas)
    const mangaCount = computed(() => mangasStore.getMangaCount)
    const mangasYearsMinMax = computed(() => mangasStore.getYearsMinMAX)
    const mangasYears = computed(() => mangasStore.getYears)
    const mangasScoresMinMax = computed(() => mangasStore.getScoresMinMAX)
    const mangasScores = computed(() => mangasStore.getScores)

    const topMangasByGenre = computed(() => {
      const mangaStore = useMangaStore()
      return mangaStore.getMangasByGenre(preferredGenres.value.name)
    })

    const filteredMangas = computed(() => {
      const mangaStore = useMangaStore()
      return mangaStore.getMangasFiltered(filters)
    })
    // #endregion

    const gridKey = ref(0)
    const openTab = ref(0)

    // computed
    const getSearchOrderBy = computed(() => {
      return filters.orderBy === 'asc' || filters.orderBy == null
    })

    const getSearchOrderName = computed(() =>
      getSearchOrderBy.value ? 'ascending' : 'descending'
    )

    const filterAlphabet = computed(() => {
      const letters = filteredMangas.value.map((m) => {
        if (m.canonicalTitle && m.canonicalTitle.length > 0) {
          const firstLetter = m?.canonicalTitle[0].toUpperCase() || ''
          if (firstLetter.match(/[a-z]/i)) {
            return firstLetter
          } else {
            return '#'
          }
        } else {
          return ''
        }
      })
      // remove duplicates
      const uniqueLetters = [...new Set(letters)]
      // sort alphabetically
      uniqueLetters.sort((a, b) => a.localeCompare(b))
      return uniqueLetters
    })

    const preferredGenres = computed(() => {
      return stats.value.genres[0]
    })

    function toggleTabs (tabNumber) {
      tabs.value[openTab.value].current = false
      tabs.value[tabNumber].current = true
      openTab.value = tabNumber
    }

    function scrollTo (letter) {
      const element = document.getElementById(`manga-${letter}`)
      if (element) {
        const headerOffset = 132
        const elementPosition = element.getBoundingClientRect().top
        const offsetPosition = elementPosition + window.scrollY - headerOffset

        window.scrollTo({
          top: offsetPosition,
          behavior: 'smooth'
        })
      }
    }

    async function onSearched () {
      gridKey.value += 1
    }

    onSearched = debounce(onSearched, 500)

    function sortData () {
      filters.orderBy = filters.orderBy === 'asc' ? 'desc' : 'asc'
      onSearched()
    }

    sortData = debounce(sortData, 500)

    function clearFilter () {
      filters.orderBy = 'desc'
      filters.orderByField = 'canonicalTitle'
      filters.searchText = null
      filters.yearsMarks = []
      filters.types = []
      filters.genres = []

      filters.years = mangasYearsMinMax.value
      filters.scores = mangasScoresMinMax.value
      filters.yearsMarks = [filters.years[0], filters.years[1]]
      sortData()
    }

    onMounted(() => {
      statsStore.fetchStats()
      mangasStore.fetchMangas().then(() => {
        document.title = 'Mangas - Teemii'
        pageTitle.value = 'Collection - ' + mangaCount.value + ' mangas'
        clearFilter()
        // set is loaded after 300 ms
        setTimeout(() => {
          isLoaded.value = true
        }, 100)
      })
    })

    // expose
    return {
      // pinia
      storeIsLoading,
      lastMangasRead,
      lastAddedMangas,
      lastPublishedMangas,
      mangaCount,
      mangasYearsMinMax,
      mangasYears,
      mangasScoresMinMax,
      mangasScores,
      filteredMangas,
      // ..
      clearFilter,
      sortData,
      getSearchOrderBy,
      getSearchOrderName,
      itemPubDemographics,
      itemGenres,
      gridKey,
      onSearched,
      filters,
      topMangasByGenre,
      preferredGenres,
      topUnreadMangas,
      scrollTo,
      tabs,
      helpers,
      toggleTabs,
      openTab,
      filterAlphabet,
      isLoaded
    }
  }
}
</script>
