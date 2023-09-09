<script setup lang="ts">
import type { CSSProperties } from 'vue'
import { computed, ref, watch } from 'vue'
import { NButton, NLayoutSider, NModal, useMessage } from 'naive-ui'
import List from './List.vue'
import Footer from './Footer.vue'
import { useAppStore, useChatStore } from '@/store'
import { useBasicLayout } from '@/hooks/useBasicLayout'
import { PromptStore } from '@/components/common'

const appStore = useAppStore()
const chatStore = useChatStore()
const showModal = ref(false)
const { isMobile } = useBasicLayout()
const show = ref(false)
const message = useMessage()
const collapsed = computed(() => appStore.siderCollapsed)

function handleAdd() {
  chatStore.addHistory({ title: 'New Chat', uuid: Date.now(), isEdit: false })
  if (isMobile.value)
    appStore.setSiderCollapsed(true)
}
const handleDeleteAll = () => {
  showModal.value = true
}
const submitCallback = () => {
  chatStore.clearAll()
  message.success('对话框已清空')
}
function handleUpdateCollapsed() {
  appStore.setSiderCollapsed(!collapsed.value)
}

const getMobileClass = computed<CSSProperties>(() => {
  if (isMobile.value) {
    return {
      position: 'fixed',
      zIndex: 50,
    }
  }
  return {}
})

const mobileSafeArea = computed(() => {
  if (isMobile.value) {
    return {
      paddingBottom: 'env(safe-area-inset-bottom)',
    }
  }
  return {}
})

watch(
  isMobile,
  (val) => {
    appStore.setSiderCollapsed(val)
  },
  {
    immediate: true,
    flush: 'post',
  },
)
</script>

<template>
  <NLayoutSider :collapsed="collapsed" :collapsed-width="0" :width="260" :show-trigger="isMobile ? false : 'arrow-circle'" collapse-mode="transform" position="absolute" bordered :style="getMobileClass" @update-collapsed="handleUpdateCollapsed">
    <div class="flex flex-col h-full" :style="mobileSafeArea">
      <main class="flex flex-col flex-1 min-h-0">
        <div class="p-4" style="display: flex; justify-content: space-between; align-items: center">
          <div class="left">
            <NButton type="success" dashed block @click="handleAdd">
              {{ $t("chat.newChatButton") }}
            </NButton>
          </div>
          <div class="right" @click="handleDeleteAll">
            <NButton v-if="chatStore.chatNumber" type="error" dashed size="medium">
              清空聊天
            </NButton>
          </div>
        </div>
        <div class="flex-1 min-h-0 pb-4 overflow-hidden">
          <List />
        </div>
        <div class="p-4">
          <NButton block @click="show = true">
            {{ $t("store.siderButton") }}
          </NButton>
        </div>
      </main>
      <Footer />
    </div>
  </NLayoutSider>
  <template v-if="isMobile">
    <div v-show="!collapsed" class="fixed inset-0 z-40 w-full h-full bg-black/40" @click="handleUpdateCollapsed" />
  </template>
  <PromptStore v-model:visible="show" />
  <NModal v-model:show="showModal" preset="dialog" title="删除对话框" content="确认清空所有对话框吗?" positive-text="确认" negative-text="算了" @positive-click="submitCallback" />
</template>
