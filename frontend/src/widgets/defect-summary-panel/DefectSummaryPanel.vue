<script setup lang="ts">
import { computed } from 'vue'

import type { DefectStatistics } from '@/entities/defect'

type DefectSummaryCard = {
  label: string
  value: number
  description: string
  status: string
  tone: 'primary' | 'assigned' | 'processing' | 'verify'
}

const props = defineProps<{
  statistics: DefectStatistics | null
  activeStatus?: string
}>()

const emit = defineEmits<{
  select: [status: string]
}>()

const stats = computed<DefectSummaryCard[]>(() => {
  const source = props.statistics ?? {
    total: 0,
    todo: 0,
    assigned: 0,
    inProgress: 0,
    pendingVerify: 0,
    closed: 0,
    rejected: 0,
  }

  return [
    { label: '缺陷总数', value: source.total, description: '全部缺陷', status: '', tone: 'primary' },
    { label: '待处理', value: source.assigned, description: '已指派待处理', status: 'ASSIGNED', tone: 'assigned' },
    { label: '处理中', value: source.inProgress, description: '正在处理中', status: 'IN_PROGRESS', tone: 'processing' },
    { label: '待验证', value: source.pendingVerify, description: '等待验证结果', status: 'PENDING_VERIFY', tone: 'verify' },
  ]
})

function isActive(status: string) {
  if (!status) {
    return !props.activeStatus
  }

  return props.activeStatus === status
}

function handleSelect(status: string) {
  emit('select', status)
}
</script>

<template>
  <div class="defect-summary-panel">
    <article
      v-for="stat in stats"
      :key="stat.label"
      class="defect-summary-panel__card"
      :class="[
        `defect-summary-panel__card--${stat.tone}`,
        { 'defect-summary-panel__card--active': isActive(stat.status) },
      ]"
      role="button"
      tabindex="0"
      @click="handleSelect(stat.status)"
      @keydown.enter.prevent="handleSelect(stat.status)"
      @keydown.space.prevent="handleSelect(stat.status)"
    >
      <div class="defect-summary-panel__card-head">
        <span>{{ stat.label }}</span>
      </div>
      <strong>{{ stat.value }}</strong>
      <p>{{ stat.description }}</p>
    </article>
  </div>
</template>

<style scoped>
.defect-summary-panel {
  display: grid;
  grid-template-columns: repeat(4, minmax(0, 1fr));
  gap: 16px;
}

.defect-summary-panel__card {
  display: grid;
  gap: 10px;
  min-height: 144px;
  position: relative;
  overflow: hidden;
  padding: 18px 20px;
  border: 1px solid var(--app-border-soft);
  border-left: 4px solid var(--app-primary);
  border-radius: var(--app-radius-sm);
  background: var(--app-bg-panel);
  box-shadow: none;
  cursor: pointer;
  transition: box-shadow 160ms ease;
}

.defect-summary-panel__card:hover {
  box-shadow: var(--app-shadow-card);
}

.defect-summary-panel__card--active {
  border-color: #bfdbfe;
  border-left-color: var(--app-primary);
  background: #ffffff;
  box-shadow: 0 0 0 1px rgba(64, 158, 255, 0.08);
}

.defect-summary-panel__card--active .defect-summary-panel__card-head span,
.defect-summary-panel__card--active .defect-summary-panel__card strong {
  color: #1d4ed8;
}

.defect-summary-panel__card-head {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: var(--app-space-3);
  position: relative;
  z-index: 1;
  margin-bottom: 6px;
}

.defect-summary-panel__card-head span {
  color: var(--app-text-secondary);
  font-size: var(--app-font-size-sm);
  font-weight: 600;
  line-height: var(--app-line-height-sm);
}

.defect-summary-panel__card strong {
  position: relative;
  z-index: 1;
  color: var(--app-text-primary);
  font-size: 28px;
  line-height: 32px;
}

.defect-summary-panel__card p {
  position: relative;
  z-index: 1;
  margin: 0;
  color: var(--app-text-muted);
  font-size: var(--app-font-size-xs);
  line-height: 18px;
}

.defect-summary-panel__card--primary {
  border-left-color: var(--app-primary);
}

.defect-summary-panel__card--primary strong {
  color: var(--app-primary);
}

.defect-summary-panel__card--assigned {
  border-left-color: var(--app-purple);
}

.defect-summary-panel__card--assigned strong {
  color: var(--app-purple);
}

.defect-summary-panel__card--processing {
  border-left-color: var(--app-success);
}

.defect-summary-panel__card--processing strong {
  color: var(--app-success);
}

.defect-summary-panel__card--verify {
  border-left-color: var(--app-warning);
}

.defect-summary-panel__card--verify strong {
  color: var(--app-warning);
}


@media (max-width: 1180px) {
  .defect-summary-panel {
    grid-template-columns: repeat(2, minmax(0, 1fr));
  }
}

@media (max-width: 720px) {
  .defect-summary-panel {
    grid-template-columns: 1fr;
  }
}
</style>
