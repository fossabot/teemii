<template>
  <Menu as="div" class="relative z-50 inline-block text-left overflow-visible">
    <MenuButton :class="containerClass">
      <p v-if="!iconOnly">{{ caption }}</p>
      <ChevronDownIcon
          v-if="!iconOnly"
          class="-mr-1 ml-2 h-5 w-5 text-violet-200 hover:text-violet-100"
          aria-hidden="true"
      />
      <span v-if="iconOnly" class="sr-only">Open options menu</span>
      <EllipsisVerticalIcon v-if="iconOnly" class="h-5 w-5" aria-hidden="true"/>
    </MenuButton>
    <transition
        enter-active-class="transition duration-200 ease-out"
        enter-from-class="transform scale-95 opacity-0"
        enter-to-class="transform scale-100 opacity-100"
        leave-active-class="transition duration-75 ease-in"
        leave-from-class="transform scale-100 opacity-100"
        leave-to-class="transform scale-95 opacity-0"
    >
      <MenuItems
          class="absolute right-0 z-50 mt-2 w-56 origin-top-right divide-y divide-main-100 border border-main-200 bg-white shadow-lg ring-1 ring-black ring-opacity-5 focus:outline-none"
      >
        <BaseMenuItem
            v-for="(menu, index) in menus"
            v-slot="{ selected, active }"
            :key="index"
            :caption="menu.caption"
            :menuClick="menu.menuClick"
            :enabled="menu.enabled"
            :icon="menu.icon"
            as="template"
        >
        </BaseMenuItem>
      </MenuItems>
    </transition>
  </Menu>
</template>

<script setup>
import { Menu, MenuButton, MenuItems } from '@headlessui/vue'
import { ChevronDownIcon, EllipsisVerticalIcon } from '@heroicons/vue/24/solid'
import BaseMenuItem from './TBaseMenuItem.vue'
import { ref, onMounted, computed, useSlots } from 'vue'

const props = defineProps({
  iconOnly: {
    type: Boolean,
    default: true,
    required: false
  },
  caption: {
    type: String,
    default: '',
    required: false
  }
})

const isLoaded = ref(false)

const emit = defineEmits(['clicked'])
const slots = useSlots()
const menus = computed(() => slots.default().map((menu) => menu.props))

const containerClass = computed(() => {
  if (!props.iconOnly) { return 'focus:outline-none inline-flex w-full justify-center rounded-md bg-main-900 px-4 py-2 text-sm font-medium text-white hover:bg-accent-900 focus-visible:ring-2 focus-visible:ring-white focus-visible:ring-opacity-75' } else if (props.iconOnly) { return 'inline-flex items-center rounded-md border-light-600 bg-white p-2 text-sm font-medium text-main-400 shadow-sm hover:bg-light-100 focus:outline-none focus:ring-2 focus:ring-accent-500 focus:ring-offset-2' }
})

onMounted(() => {
  isLoaded.value = true
})
</script>
