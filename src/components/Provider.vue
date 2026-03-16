<script setup lang="ts">
import type { GlobalTheme, GlobalThemeOverrides } from 'naive-ui'
import {
  darkTheme,
  dateEnUS,
  dateZhCN,
  enUS,
  lightTheme,
  NBackTop,
  NConfigProvider,
  NDialogProvider,
  NGlobalStyle,
  NLoadingBarProvider,
  NMessageProvider,
  NModalProvider,
  NNotificationProvider,
  useDialog,
  useLoadingBar,
  useMessage,
  useModal,
  useNotification,
  zhCN,
} from 'naive-ui'
import { computed, defineComponent, h, provide, ref, watch } from 'vue'
import { useAppStore } from '@/stores/app'

const appStore = useAppStore()
const isScrolled = ref(false)

provide('isScrolled', isScrolled)

const isDark = computed(() => appStore.isDark)

const theme = computed<GlobalTheme | null>(() => {
  return isDark.value ? darkTheme : lightTheme
})

const locale = computed(() => {
  const langMap = {
    'zh-CN': zhCN,
    'en-US': enUS,
  }
  return langMap[appStore.lang] || zhCN
})

const dateLocale = computed(() => {
  const langMap = {
    'zh-CN': dateZhCN,
    'en-US': dateEnUS,
  }
  return langMap[appStore.lang] || dateZhCN
})

function setupNaiveTools() {
  window.$loadingBar = useLoadingBar()
  window.$notification = useNotification()
  window.$message = useMessage()
  window.$dialog = useDialog()
  window.$modal = useModal()
}

const NaiveProviderContent = defineComponent({
  setup() {
    setupNaiveTools()
  },
  render() {
    return h('div', { className: 'naive-tools' })
  },
})

const themeOverride = computed<GlobalThemeOverrides>(() => {
  const settings = appStore.publicSettings?.theme_settings as Record<string, unknown> | undefined
  const primaryColor = isDark.value ? appStore.uiDarkPrimaryColor : appStore.uiLightPrimaryColor

  const primaryColorHover = appStore.personalizationActive
    ? primaryColor
    : isDark.value
        ? (settings?.darkPrimaryColorHover as string) || '#7fe7c4'
        : (settings?.lightPrimaryColorHover as string) || '#36ad6a'

  const primaryColorPressed = appStore.personalizationActive
    ? primaryColor
    : isDark.value
        ? (settings?.darkPrimaryColorPressed as string) || '#5acea7'
        : (settings?.lightPrimaryColorPressed as string) || '#0c7a43'

  return {
    common: {
      primaryColor,
      primaryColorHover,
      primaryColorPressed,
      primaryColorSuppl: primaryColorHover,
      borderRadius: appStore.uiBorderRadius,
      fontFamily: appStore.uiFontFamily,
    },
  }
})

const currentCardBackground = computed(() => {
  return isDark.value ? appStore.uiDarkCardBackground : appStore.uiLightCardBackground
})

const hasSurfaceCustomization = computed(() => {
  const cardBackground = currentCardBackground.value.trim().toLowerCase()
  return cardBackground !== '' && cardBackground !== 'transparent'
})

const customCssText = computed(() => appStore.uiCustomCss.trim())

watch(
  [
    themeOverride,
    isDark,
    () => appStore.personalizationActive,
    () => appStore.uiCardOpacity,
    currentCardBackground,
    () => appStore.uiSurfaceBlur,
    () => appStore.uiFontFamily,
  ],
  ([overrides, dark, personalized, cardOpacity, cardBackground, surfaceBlur, fontFamily]) => {
    const root = document.documentElement
    const alpha = Math.min(Math.max(cardOpacity / 100, 0.45), 1)
    const surfaceBackground = cardBackground.toLowerCase() === 'transparent'
      ? 'transparent'
      : cardBackground
    const surfaceBorder = dark
      ? 'rgba(255, 255, 255, 0.08)'
      : 'rgba(15, 23, 42, 0.08)'
    const surfaceShadow = hasSurfaceCustomization.value
      ? '0 20px 45px rgba(0, 0, 0, 0.35)'
      : dark
          ? '0 20px 45px rgba(0, 0, 0, 0.35)'
          : '0 20px 45px rgba(15, 23, 42, 0.12)'

    if (overrides.common?.primaryColor) {
      root.style.setProperty('--primary-color', overrides.common.primaryColor)
    }
    if (overrides.common?.primaryColorHover) {
      root.style.setProperty('--primary-color-hover', overrides.common.primaryColorHover)
    }
    if (overrides.common?.primaryColorPressed) {
      root.style.setProperty('--primary-color-pressed', overrides.common.primaryColorPressed)
    }

    root.style.setProperty('--app-surface-background', surfaceBackground)
    root.style.setProperty('--app-surface-border', surfaceBorder)
    root.style.setProperty('--app-surface-shadow', hasSurfaceCustomization.value ? surfaceShadow : 'none')
    root.style.setProperty('--app-surface-blur', personalized && hasSurfaceCustomization.value ? `${surfaceBlur}px` : '0px')
    root.style.setProperty('--app-surface-opacity', `${alpha}`)
    root.style.setProperty('--app-font-family', fontFamily)

    root.classList.toggle('dark', dark)
    root.classList.toggle('personalized-ui', personalized)
    root.classList.toggle('surface-customized', personalized && hasSurfaceCustomization.value)
  },
  { immediate: true },
)

watch(
  [() => appStore.backgroundEnabled, isDark],
  ([enabled, dark]) => {
    const body = document.body

    if (enabled) {
      body.style.setProperty('background-color', 'transparent', 'important')
      return
    }

    body.style.removeProperty('background-color')
    body.style.backgroundColor = dark ? 'rgb(16, 16, 20)' : '#fff'
  },
  { immediate: true },
)
</script>

<template>
  <NConfigProvider :theme="theme" :theme-overrides="themeOverride" :locale="locale" :date-locale="dateLocale">
    <NBackTop :visibility-height="1" class="z-9999" @update:show="isScrolled = $event" />
    <NGlobalStyle />
    <NLoadingBarProvider>
      <NDialogProvider>
        <NNotificationProvider>
          <NMessageProvider>
            <NModalProvider>
              <slot />
              <component :is="'style'" v-if="customCssText">
                {{ customCssText }}
              </component>
              <NaiveProviderContent />
            </NModalProvider>
          </NMessageProvider>
        </NNotificationProvider>
      </NDialogProvider>
    </NLoadingBarProvider>
  </NConfigProvider>
</template>

<style>
:root {
  --app-surface-background: transparent;
  --app-surface-border: rgba(15, 23, 42, 0.08);
  --app-surface-shadow: none;
  --app-surface-blur: 0px;
  --app-surface-opacity: 1;
  --app-font-family: "MiSans VF", sans-serif;
}

html.personalized-ui body {
  font-family: var(--app-font-family);
}

html.personalized-ui.surface-customized .n-card,
html.personalized-ui.surface-customized .n-alert,
html.personalized-ui.surface-customized .n-base-selection,
html.personalized-ui.surface-customized .n-drawer,
html.personalized-ui.surface-customized .n-modal,
html.personalized-ui.surface-customized .n-popover {
  backdrop-filter: blur(var(--app-surface-blur));
  box-shadow: var(--app-surface-shadow);
}

html.personalized-ui.surface-customized .n-card,
html.personalized-ui.surface-customized .n-alert,
html.personalized-ui.surface-customized .n-base-selection,
html.personalized-ui.surface-customized .n-drawer,
html.personalized-ui.surface-customized .n-modal,
html.personalized-ui.surface-customized .n-popover {
  background: color-mix(in srgb, var(--app-surface-background) calc(var(--app-surface-opacity) * 100%), transparent) !important;
  border-color: var(--app-surface-border) !important;
}
</style>
