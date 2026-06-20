<script setup lang="ts">
import { Document } from '@element-plus/icons-vue'

withDefaults(
  defineProps<{
    title?: string
    description?: string
  }>(),
  {
    title: '暂无数据',
    description: '当前没有可展示的内容。',
  },
)
</script>

<template>
  <div class="app-empty-state">
    <div class="app-empty-state__icon" aria-hidden="true">
      <el-icon><Document /></el-icon>
    </div>
    <div class="app-empty-state__title">{{ title }}</div>
    <p class="app-empty-state__description">{{ description }}</p>
    <div v-if="$slots.actions" class="app-empty-state__actions">
      <slot name="actions" />
    </div>
  </div>
</template>

<style scoped>
.app-empty-state {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  min-height: 204px;
  padding: var(--app-space-6);
  border: 0;
  border-radius: var(--app-radius-sm);
  background: #fafafa;
  text-align: center;
}

.app-empty-state__icon {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 56px;
  height: 56px;
  border-radius: 0;
  background: transparent;
  color: var(--app-text-muted);
  font-size: 56px;
  opacity: 0.54;
}

.app-empty-state__title {
  margin-top: var(--app-space-3);
  color: var(--app-text-primary);
  font-size: var(--app-font-size-md);
  font-weight: 600;
}

.app-empty-state__description {
  max-width: 420px;
  margin: var(--app-space-2) 0 0;
  color: var(--app-text-muted);
  font-size: var(--app-font-size-sm);
  line-height: var(--app-line-height-md);
}

.app-empty-state__actions {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: var(--app-space-2);
  margin-top: var(--app-space-4);
}
</style>
