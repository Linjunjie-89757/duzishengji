<script setup lang="ts">
import type { ConfigStat } from '../model/types'

defineProps<{
  stat: ConfigStat
}>()
</script>

<template>
  <article class="config-stat-card" :class="`config-stat-card--${stat.tone ?? 'default'}`">
    <div class="config-stat-card__head">
      <span>{{ stat.label }}</span>
    </div>
    <strong>{{ stat.value }}</strong>
  </article>
</template>

<style scoped>
.config-stat-card {
  display: flex;
  min-height: 100px;
  flex-direction: column;
  justify-content: space-between;
  position: relative;
  overflow: hidden;
  padding: 18px 20px;
  border: 1px solid var(--app-border-soft);
  border-radius: var(--app-radius-sm);
  background: var(--app-bg-panel);
  box-shadow: none;
  transition: border-color 160ms ease, background-color 160ms ease;
}

.config-stat-card:hover {
  border-color: var(--app-border);
  background: #fff;
}

.config-stat-card__head {
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  gap: var(--app-space-3);
}

.config-stat-card span {
  color: var(--app-text-muted);
  font-size: var(--app-font-size-md);
  font-weight: 600;
}

.config-stat-card strong {
  position: relative;
  z-index: 1;
  color: var(--app-primary);
  font-size: 30px;
  font-weight: 600;
  line-height: 36px;
}

.config-stat-card--success strong {
  color: var(--app-success);
}

.config-stat-card--warning strong {
  color: var(--app-warning);
}

.config-stat-card--danger strong {
  color: var(--app-danger);
}

.config-stat-card--purple strong {
  color: var(--app-purple);
}
</style>
