<script setup lang="ts">
import {
  NAlert,
  NButton,
  NColorPicker,
  NDrawer,
  NDrawerContent,
  NForm,
  NFormItem,
  NInput,
  NRadioButton,
  NRadioGroup,
  NSlider,
  NSpace,
  NSwitch,
  NTabPane,
  NTabs,
  NText,
} from 'naive-ui'
import { computed } from 'vue'
import { useAppStore } from '@/stores/app'

const props = defineProps<{
  show: boolean
}>()

const emit = defineEmits<{
  'update:show': [value: boolean]
}>()

const appStore = useAppStore()

const drawerVisible = computed({
  get: () => props.show,
  set: value => emit('update:show', value),
})

function resetPersonalization() {
  appStore.resetPersonalization()
  window.$message?.success('个性化设置已重置')
}
</script>

<template>
  <NDrawer v-model:show="drawerVisible" :width="420" placement="right" resizable>
    <NDrawerContent title="个性化设置" closable>
      <div class="flex flex-col gap-4">
        <NAlert type="info" :show-icon="false">
          仅登录状态可用，配置会按当前账号保存在本地浏览器。
        </NAlert>

        <div class="flex items-center justify-between rounded-2xl border border-$n-border-color p-4">
          <div class="flex flex-col gap-1">
            <NText strong>启用个性化主题</NText>
            <NText depth="3">开启后会覆盖站点默认背景和部分界面样式。</NText>
          </div>
          <NSwitch v-model:value="appStore.personalization.enabled" />
        </div>

        <NTabs type="line" animated>
          <NTabPane name="background" tab="背景">
            <NForm label-placement="top">
              <NFormItem label="启用自定义背景">
                <NSwitch v-model:value="appStore.personalization.backgroundEnabled" />
              </NFormItem>
              <NFormItem label="背景类型">
                <NRadioGroup v-model:value="appStore.personalization.backgroundType">
                  <NRadioButton value="image">
                    图片
                  </NRadioButton>
                  <NRadioButton value="video">
                    视频
                  </NRadioButton>
                </NRadioGroup>
              </NFormItem>
              <NFormItem label="浅色模式背景地址">
                <NInput
                  v-model:value="appStore.personalization.lightBackgroundUrl"
                  placeholder="https://example.com/light-background.jpg"
                />
              </NFormItem>
              <NFormItem label="深色模式背景地址">
                <NInput
                  v-model:value="appStore.personalization.darkBackgroundUrl"
                  placeholder="https://example.com/dark-background.jpg"
                />
              </NFormItem>
              <NFormItem label="背景模糊">
                <NSlider v-model:value="appStore.personalization.backgroundBlur" :min="0" :max="40" />
              </NFormItem>
              <NFormItem label="背景遮罩">
                <NSlider v-model:value="appStore.personalization.backgroundOverlay" :min="0" :max="100" />
              </NFormItem>
            </NForm>
          </NTabPane>

          <NTabPane name="ui" tab="界面">
            <NForm label-placement="top">
              <NFormItem label="主题模式">
                <NRadioGroup v-model:value="appStore.themeMode">
                  <NRadioButton value="auto">
                    自动
                  </NRadioButton>
                  <NRadioButton value="light">
                    浅色
                  </NRadioButton>
                  <NRadioButton value="dark">
                    深色
                  </NRadioButton>
                </NRadioGroup>
              </NFormItem>
              <NFormItem label="浅色主色">
                <NColorPicker v-model:value="appStore.personalization.lightPrimaryColor" />
              </NFormItem>
              <NFormItem label="深色主色">
                <NColorPicker v-model:value="appStore.personalization.darkPrimaryColor" />
              </NFormItem>
              <NFormItem :label="`圆角 ${appStore.personalization.borderRadius}px`">
                <NSlider v-model:value="appStore.personalization.borderRadius" :min="0" :max="32" />
              </NFormItem>
              <NFormItem :label="`卡片透明度 ${appStore.personalization.cardOpacity}%`">
                <NSlider v-model:value="appStore.personalization.cardOpacity" :min="45" :max="100" />
              </NFormItem>
              <NFormItem label="浅色卡片背景">
                <NInput
                  v-model:value="appStore.personalization.lightCardBackground"
                  placeholder="transparent 或 rgba(255,255,255,0.72)"
                />
              </NFormItem>
              <NFormItem label="深色卡片背景">
                <NInput
                  v-model:value="appStore.personalization.darkCardBackground"
                  placeholder="transparent 或 rgba(24,24,28,0.82)"
                />
              </NFormItem>
              <NFormItem :label="`界面模糊 ${appStore.personalization.surfaceBlur}px`">
                <NSlider v-model:value="appStore.personalization.surfaceBlur" :min="0" :max="32" />
              </NFormItem>
              <NFormItem label="字体族">
                <NInput
                  v-model:value="appStore.personalization.fontFamily"
                  placeholder='"MiSans VF", sans-serif'
                />
              </NFormItem>
              <NFormItem label="自定义 CSS">
                <NInput
                  v-model:value="appStore.personalization.customCss"
                  type="textarea"
                  :autosize="{ minRows: 6, maxRows: 12 }"
                  placeholder=".n-card { border-width: 0; }"
                />
              </NFormItem>
            </NForm>
          </NTabPane>
        </NTabs>

        <NSpace justify="end">
          <NButton @click="resetPersonalization">
            重置
          </NButton>
          <NButton type="primary" @click="drawerVisible = false">
            完成
          </NButton>
        </NSpace>
      </div>
    </NDrawerContent>
  </NDrawer>
</template>
