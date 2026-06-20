<script setup lang="ts">
import type { ApiAutomationStat } from '@/entities/api-automation'
import AppLoadingState from '@/shared/ui/app-loading-state/AppLoadingState.vue'

defineProps<{
  stats: ApiAutomationStat[]
  loading?: boolean
}>()
</script>

<template>
  <section class="api-automation-summary-panel">
    <AppLoadingState v-if="loading && stats.length === 0" text="正在统计接口资产" />
    <article
      v-for="item in stats"
      v-else
      :key="item.label"
      class="api-automation-summary-panel__item"
      :class="`api-automation-summary-panel__item--${item.tone || 'default'}`"
    >
      <span>{{ item.label }}</span>
      <strong>{{ item.value }}</strong>
    </article>
  </section>
</template>

<style scoped>
.api-automation-summary-panel {
  display: grid;
  grid-template-columns: repeat(5, minmax(120px, 1fr));
  gap: var(--app-space-4);
}

.api-automation-summary-panel__item {
  min-width: 0;
  padding: var(--app-space-4);
  border: 1px solid var(--app-border-soft);
  border-left: 4px solid var(--app-primary);
  border-radius: var(--app-radius-sm);
  background: var(--app-bg-panel);
}

.api-automation-summary-panel__item span {
  display: block;
  color: var(--app-text-secondary);
  font-size: var(--app-font-size-md);
}

.api-automation-summary-panel__item strong {
  display: block;
  margin-top: var(--app-space-2);
  color: var(--app-text-primary);
  font-size: 28px;
  font-weight: 600;
  line-height: 1;
}

.api-automation-summary-panel__item--primary strong {
  color: var(--app-primary);
}

.api-automation-summary-panel__item--success strong {
  color: var(--app-success);
}

.api-automation-summary-panel__item--success {
  border-left-color: var(--app-success);
}

.api-automation-summary-panel__item--danger strong {
  color: var(--app-danger);
}

.api-automation-summary-panel__item--danger {
  border-left-color: var(--app-danger);
}

.api-automation-summary-panel__item--purple strong {
  color: var(--app-purple);
}

.api-automation-summary-panel__item--purple {
  border-left-color: var(--app-purple);
}

@media (max-width: 980px) {
  .api-automation-summary-panel {
    grid-template-columns: repeat(2, minmax(0, 1fr));
  }
}

@media (max-width: 520px) {
  .api-automation-summary-panel {
    grid-template-columns: 1fr;
  }
}
</style>
