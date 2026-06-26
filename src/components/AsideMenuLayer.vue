<script setup>
import { mdiLogout, mdiClose, mdiConsoleLine } from '@mdi/js'
import { computed } from 'vue'
import AsideMenuList from '@/components/AsideMenuList.vue'
import AsideMenuItem from '@/components/AsideMenuItem.vue'
import BaseIcon from '@/components/BaseIcon.vue'

defineProps({
  menu: {
    type: Array,
    required: true
  }
})

const emit = defineEmits(['menu-click', 'aside-lg-close-click'])

const logoutItem = computed(() => ({
  label: 'Logout',
  icon: mdiLogout,
  color: 'info',
  isLogout: true
}))

const menuClick = (event, item) => {
  emit('menu-click', event, item)
}

const asideLgCloseClick = (event) => {
  emit('aside-lg-close-click', event)
}
</script>

<template>
  <aside
    id="aside"
    class=" w-60 fixed flex z-40 top-0 h-screen transition-position overflow-hidden"
  >
    <div class="aside flex-1 flex flex-col overflow-hidden dark:bg-slate-900">
      <div class="aside-brand flex flex-row h-14 items-center justify-between dark:bg-slate-900">
        <div class="flex-1 lg:text-left lg:pl-6 xl:pl-0">
          <RouterLink to="/sources" class="flex items-center gap-2 pl-6 xl:pl-6">
            <span
              class="inline-flex items-center justify-center w-8 h-8 rounded-lg bg-blue-600 text-white shadow-sm"
            >
              <BaseIcon :path="mdiConsoleLine" size="20" />
            </span>
            <b class="font-black text-lg tracking-tight">Log<span class="text-blue-500">Man</span></b>
          </RouterLink>
        </div>
        <button class="hidden lg:inline-block xl:hidden p-3" @click.prevent="asideLgCloseClick">
          <BaseIcon :path="mdiClose" />
        </button>
      </div>
      <div
        class="flex-1 overflow-y-auto overflow-x-hidden aside-scrollbars dark:aside-scrollbars-[slate]"
      >
        <AsideMenuList :menu="menu" @menu-click="menuClick" />
      </div>

      <ul>
        <AsideMenuItem :item="logoutItem" @menu-click="menuClick" />
      </ul>
    </div>
  </aside>
</template>
