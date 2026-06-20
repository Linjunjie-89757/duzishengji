<script setup lang="ts">
import { Loading } from '@element-plus/icons-vue'

withDefaults(
  defineProps<{
    text?: string
  }>(),
  {
    text: '加载中...',
  },
)
</script>

<template>
  <div class="app-loading-state" role="status" aria-live="polite">
    <el-icon class="app-loading-state__spinner">
      <Loading />
    </el-icon>
    <div class="app-loading-state__text">{{ text }}</div>
  </div>
</template>

<style scoped>
.app-loading-state {
  display: flex;
  min-height: 160px;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: var(--app-space-6);
  border: 0;
  border-radius: var(--app-radius-sm);
  background: #fafafa;
}

.app-loading-state__spinner {
  color: var(--app-primary);
  font-size: 24px;
  animation: app-loading-rotate 1s linear infinite;
}

.app-loading-state__text {
  margin-top: var(--app-space-3);
  color: var(--app-text-muted);
  font-size: var(--app-font-size-sm);
  text-align: center;
}

@keyframes app-loading-rotate {
  to {
    transform: rotate(360deg);
  }
}
</style>
