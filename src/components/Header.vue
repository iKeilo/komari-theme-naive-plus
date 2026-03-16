<script setup lang="ts">
import { NAvatar, NButton, NFlex, NH3, NPopover } from 'naive-ui'
import { computed, h, inject, ref } from 'vue'
import { useRouter } from 'vue-router'
import { useAppStore } from '@/stores/app'
import LoginDialog from './LoginDialog.vue'
import PersonalizationDrawer from './PersonalizationDrawer.vue'

const router = useRouter()
const appStore = useAppStore()

const isScrolled = inject<ReturnType<typeof ref<boolean>>>('isScrolled', ref(false))
const siteFavicon = ref('/favicon.ico')
const showPersonalizationDrawer = ref(false)

const containerStyle = computed(() => {
  if (appStore.fullWidth) {
    return {}
  }

  return {
    maxWidth: appStore.maxPageWidth,
    marginInline: 'auto',
  }
})

const actionButtons = computed(() => {
  const buttons = [
    {
      title: appStore.themeMode === 'auto' ? '自动主题' : appStore.themeMode === 'light' ? '浅色主题' : '深色主题',
      icon: appStore.themeMode === 'auto' ? 'i-icon-park-outline-dark-mode' : appStore.themeMode === 'light' ? 'i-icon-park-outline-sun-one' : 'i-icon-park-outline-moon',
      action: 'toggleTheme',
      disabled: false,
      label: '',
    },
  ]

  if (appStore.isLoggedIn) {
    buttons.push({
      title: '个性化设置',
      icon: 'i-icon-park-outline-setting',
      action: 'openPersonalizationDrawer',
      disabled: false,
      label: '主题',
    })
    buttons.push({
      title: '后台管理',
      icon: 'i-icon-park-outline-setting',
      action: 'jumpToSetting',
      disabled: false,
      label: '',
    })
  }
  else if (appStore.showLoginButton) {
    buttons.push({
      title: '登录',
      icon: 'i-icon-park-outline-login',
      action: 'openLoginDialog',
      disabled: false,
      label: '',
    })
  }

  return buttons
})

function handleButtonClick(action: string) {
  switch (action) {
    case 'toggleTheme':
      appStore.updateThemeMode()
      break
    case 'openPersonalizationDrawer':
      showPersonalizationDrawer.value = true
      break
    case 'jumpToSetting':
      location.href = '/admin'
      break
    case 'openLoginDialog':
      window.$modal.create({
        title: '登录',
        preset: 'dialog',
        showIcon: false,
        content: () => h(LoginDialog),
      })
      break
  }
}
</script>

<template>
  <div class="transition-all duration-200 top-0 position-sticky z-10" :class="isScrolled ? 'bg-$n-color shadow-sm backdrop-blur-md' : 'bg-transparent'">
    <div class="px-4 flex-between h-16" :style="containerStyle">
      <NFlex class="flex-center cursor-pointer" @click="router.push('/')">
        <NAvatar :src="siteFavicon" round />
        <NH3 class="m-0">
          {{ appStore.publicSettings?.sitename || 'Komari Monitor' }}
        </NH3>
      </NFlex>
      <NFlex class="flex gap-4">
        <NPopover v-for="button in actionButtons" :key="button.action" :disabled="button.disabled">
          <template #trigger>
            <NButton
              :disabled="button.disabled"
              :class="button.label ? 'px-3 h-8 rounded-full flex items-center' : 'p-2 h-8 w-8'"
              text
              @click="handleButtonClick(button.action)"
            >
              <div :class="button.icon" />
              <span v-if="button.label" class="ml-1 text-xs font-medium">
                {{ button.label }}
              </span>
            </NButton>
          </template>
          <template #default>
            {{ button.title }}
          </template>
        </NPopover>
      </NFlex>
    </div>
  </div>
  <PersonalizationDrawer v-model:show="showPersonalizationDrawer" />
</template>
