<script setup lang="ts">
import {
  ArrowDown,
  Cpu,
  Files,
  Grid,
  Monitor,
  Setting,
  SwitchButton,
  Tools,
  User,
  Warning,
} from '@element-plus/icons-vue'
import { ChevronDown, Layers } from '@lucide/vue'
import { ElMessage } from 'element-plus'
import { computed, onMounted, ref, watch } from 'vue'
import { useRoute, useRouter } from 'vue-router'

import { useSession } from '@/entities/session'
import { useWorkspaceContext, workspaceApi, type WorkspaceItem } from '@/entities/workspace'
import { useLogout } from '@/features/auth-logout'
import { getRequestErrorMessage } from '@/shared/api/error'

const router = useRouter()
const route = useRoute()
const { currentUser } = useSession()
const { loading: logoutLoading, errorMessage: logoutErrorMessage, logout } = useLogout()
const { selectedWorkspaceCode, setSelectedWorkspaceCode } = useWorkspaceContext()
const switchableWorkspaces = ref<WorkspaceItem[]>([])
const workspaceLoading = ref(false)
const workspaceErrorMessage = ref('')
const NAV_GROUP_STORAGE_KEY = 'app:navigation-groups-expanded-v1'

const headerTitle = computed(() => {
  return typeof route.meta.title === 'string' && route.meta.title ? route.meta.title : '前端 2.0 重建'
})

const userDisplayName = computed(() => {
  const user = currentUser.value
  return user?.displayName || user?.username || '当前用户'
})

const userRoleText = computed(() => currentUser.value?.roleCode || '已登录')

const workspaceOptions = computed(() => {
  const options = switchableWorkspaces.value.map((item) => ({
    label: item.workspaceName || item.workspaceCode,
    value: item.workspaceCode,
  }))

  if (!options.some(item => item.value === 'ALL')) {
    options.unshift({ label: '全部空间', value: 'ALL' })
  }

  return options
})

const selectedWorkspaceName = computed(() => {
  return workspaceOptions.value.find(item => item.value === selectedWorkspaceCode.value)?.label || '全部空间'
})

const breadcrumbItems = computed(() => {
  for (const item of navigationItems) {
    const matchedChild = item.children?.find(child => matchesNavigationPath(child.path))
    if (matchedChild) {
      return item.label === matchedChild.label ? [item.label] : [item.label, matchedChild.label]
    }

    if (matchesNavigationPath(item.path)) {
      return item.label === headerTitle.value ? [item.label] : [item.label, headerTitle.value]
    }
  }

  return [headerTitle.value]
})

const navigationTargetQuery = computed(() => ({
  workspace: selectedWorkspaceCode.value,
}))

interface NavigationItem {
  path: string
  label: string
  icon: typeof Grid
  children?: Array<{
    path: string
    label: string
  }>
}

type NavigationGroupExpandedState = Record<string, boolean>

const navigationItems: NavigationItem[] = [
  { path: '/', label: '工作台', icon: Grid },
  { path: '/config-center', label: '配置中心', icon: Setting },
  {
    path: '/cases',
    label: '用例中心',
    icon: Files,
    children: [
      { path: '/cases/manage', label: '用例管理' },
      { path: '/cases/ai-generate', label: 'AI 用例生成' },
      { path: '/cases/ai-records', label: 'AI 生成记录' },
      { path: '/cases/ai-config', label: 'AI 配置' },
    ],
  },
  { path: '/bugs', label: '缺陷管理', icon: Warning },
  { path: '/automation/api', label: '接口自动化', icon: Cpu },
  { path: '/automation/web', label: 'Web UI 自动化', icon: Monitor },
  { path: '/automation/app', label: 'APP 自动化', icon: Tools },
  { path: '/settings', label: '系统设置', icon: User },
]

function readNavigationGroupExpandedState(): NavigationGroupExpandedState {
  if (typeof window === 'undefined') {
    return {}
  }

  const raw = window.localStorage.getItem(NAV_GROUP_STORAGE_KEY)
  if (!raw) {
    return {}
  }

  try {
    return JSON.parse(raw) as NavigationGroupExpandedState
  } catch {
    return {}
  }
}

const navigationGroupExpandedState = ref<NavigationGroupExpandedState>(readNavigationGroupExpandedState())

function matchesNavigationPath(path: string) {
  return route.path === path || route.path.startsWith(`${path}/`)
}

function hasNavigationChildren(item: NavigationItem) {
  return Boolean(item.children?.length)
}

function isNavigationItemActive(item: NavigationItem) {
  if (item.children?.length) {
    return item.children.some(child => matchesNavigationPath(child.path)) || matchesNavigationPath(item.path)
  }
  return matchesNavigationPath(item.path)
}

function isNavigationChildActive(path: string) {
  return matchesNavigationPath(path)
}

function persistNavigationGroupExpandedState() {
  if (typeof window === 'undefined') {
    return
  }

  window.localStorage.setItem(NAV_GROUP_STORAGE_KEY, JSON.stringify(navigationGroupExpandedState.value))
}

function isNavigationGroupExpanded(item: NavigationItem) {
  if (!hasNavigationChildren(item)) {
    return false
  }

  const stored = navigationGroupExpandedState.value[item.path]
  if (typeof stored === 'boolean') {
    return stored
  }

  return isNavigationItemActive(item)
}

function toggleNavigationGroup(item: NavigationItem) {
  if (!hasNavigationChildren(item)) {
    return
  }

  navigationGroupExpandedState.value = {
    ...navigationGroupExpandedState.value,
    [item.path]: !isNavigationGroupExpanded(item),
  }
  persistNavigationGroupExpandedState()
}

function resolveInitialWorkspaceCode(items: WorkspaceItem[]) {
  const routeWorkspace = Array.isArray(route.query.workspace) ? route.query.workspace[0] : route.query.workspace
  if (routeWorkspace && (routeWorkspace === 'ALL' || items.some(item => item.workspaceCode === routeWorkspace))) {
    return routeWorkspace
  }

  if (
    selectedWorkspaceCode.value
    && (selectedWorkspaceCode.value === 'ALL' || items.some(item => item.workspaceCode === selectedWorkspaceCode.value))
  ) {
    return selectedWorkspaceCode.value
  }

  const selected = items.find((item) => item.current || item.isCurrent || item.default || item.isDefault)
  return selected?.workspaceCode || items[0]?.workspaceCode || 'ALL'
}

async function loadSwitchableWorkspaces() {
  workspaceLoading.value = true
  workspaceErrorMessage.value = ''
  try {
    const items = await workspaceApi.getSwitchableWorkspaces()
    switchableWorkspaces.value = items
    setSelectedWorkspaceCode(resolveInitialWorkspaceCode(items))
  } catch (error) {
    workspaceErrorMessage.value = getRequestErrorMessage(error)
    switchableWorkspaces.value = []
  } finally {
    workspaceLoading.value = false
  }
}

async function handleWorkspaceChange(value: string) {
  if (!value) {
    return
  }

  setSelectedWorkspaceCode(value)
  if (route.path.startsWith('/bugs')) {
    await router.replace({
      path: route.path,
      query: {
        ...route.query,
        workspace: value,
      },
      hash: route.hash,
    })
  }
}

async function handleLogout() {
  if (logoutLoading.value) {
    return
  }

  try {
    await logout()
    await router.replace('/login')
  } catch {
    ElMessage.error(logoutErrorMessage.value || '退出登录失败，请稍后重试')
  }
}

watch(
  () => route.query.workspace,
  (value) => {
    const routeWorkspace = Array.isArray(value) ? value[0] : value
    if (routeWorkspace && routeWorkspace !== selectedWorkspaceCode.value) {
      setSelectedWorkspaceCode(routeWorkspace)
    }
  },
)

onMounted(() => {
  void loadSwitchableWorkspaces()
})
</script>

<template>
  <div class="app-layout">
    <aside class="app-layout__sidebar">
      <div class="app-layout__brand">
        <span class="app-layout__brand-copy">
          <strong>自动化测试平台</strong>
        </span>
      </div>

      <nav class="app-layout__nav" aria-label="主导航">
        <div
          v-for="item in navigationItems"
          :key="item.path"
          class="app-layout__nav-group"
          :class="{
            'is-active': isNavigationItemActive(item),
            'is-expanded': isNavigationGroupExpanded(item),
          }"
        >
          <button
            v-if="hasNavigationChildren(item)"
            type="button"
            class="app-layout__nav-item app-layout__nav-button"
            :class="{
              'is-active': isNavigationItemActive(item),
              'has-children': true,
            }"
            :aria-expanded="isNavigationGroupExpanded(item)"
            @click="toggleNavigationGroup(item)"
          >
            <el-icon class="app-layout__nav-icon">
              <component :is="item.icon" />
            </el-icon>
            <span>{{ item.label }}</span>
          </button>
          <RouterLink
            v-else
            class="app-layout__nav-item"
            :class="{ 'is-active': isNavigationItemActive(item) }"
            :to="{ path: item.path, query: navigationTargetQuery }"
          >
            <el-icon class="app-layout__nav-icon">
              <component :is="item.icon" />
            </el-icon>
            <span>{{ item.label }}</span>
          </RouterLink>

          <div v-if="item.children?.length && isNavigationGroupExpanded(item)" class="app-layout__nav-children">
            <RouterLink
              v-for="child in item.children"
              :key="child.path"
              class="app-layout__nav-child"
              :class="{ 'is-active': isNavigationChildActive(child.path) }"
              :to="{ path: child.path, query: navigationTargetQuery }"
            >
              <span>{{ child.label }}</span>
            </RouterLink>
          </div>
        </div>
      </nav>
    </aside>

    <section class="app-layout__body">
      <header class="app-layout__header">
        <div class="app-layout__header-copy">
          <div class="app-layout__header-main">
            <nav class="app-layout__breadcrumb" aria-label="面包屑">
              <span
                v-for="(item, index) in breadcrumbItems"
                :key="`${item}-${index}`"
                class="app-layout__breadcrumb-item"
              >
                {{ item }}
              </span>
            </nav>
            <div class="app-layout__header-title">{{ headerTitle }}</div>
          </div>
          <span class="app-layout__header-divider" />
          <el-dropdown
            trigger="click"
            popper-class="workspace-dropdown-menu"
            :disabled="workspaceOptions.length === 0 || workspaceLoading"
            @command="handleWorkspaceChange"
          >
            <button class="app-layout__workspace-button" type="button" :aria-busy="workspaceLoading">
              <Layers class="app-layout__workspace-icon" />
              <span class="app-layout__workspace-name">{{ selectedWorkspaceName }}</span>
              <ChevronDown class="app-layout__workspace-caret" />
            </button>
            <template #dropdown>
              <el-dropdown-menu>
                <el-dropdown-item
                  v-for="item in workspaceOptions"
                  :key="item.value"
                  :command="item.value"
                >
                  {{ item.label }}
                </el-dropdown-item>
              </el-dropdown-menu>
            </template>
          </el-dropdown>
        </div>

        <div class="app-layout__header-actions">
        <el-dropdown trigger="click" @command="handleLogout">
          <button
            class="app-layout__user"
            type="button"
            :aria-busy="logoutLoading"
            :disabled="logoutLoading"
          >
            <span class="app-layout__user-avatar">{{ userDisplayName.slice(0, 1).toUpperCase() }}</span>
            <span class="app-layout__user-main">
              <span class="app-layout__user-name">{{ userDisplayName }}</span>
              <span class="app-layout__user-role">{{ userRoleText }}</span>
            </span>
            <el-icon class="app-layout__user-arrow">
              <ArrowDown />
            </el-icon>
          </button>

          <template #dropdown>
            <el-dropdown-menu>
              <el-dropdown-item command="logout" :disabled="logoutLoading">
                <el-icon>
                  <SwitchButton />
                </el-icon>
                {{ logoutLoading ? '正在退出' : '退出登录' }}
              </el-dropdown-item>
            </el-dropdown-menu>
          </template>
        </el-dropdown>
        </div>
      </header>

      <main class="app-layout__main">
        <RouterView />
      </main>
    </section>
  </div>
</template>

<style scoped>
.app-layout {
  display: flex;
  min-width: var(--app-page-min-width);
  min-height: 100dvh;
  background: var(--app-bg-page);
  color: var(--app-text-primary);
}

.app-layout__sidebar {
  position: fixed;
  inset: 0 auto 0 0;
  width: var(--app-sidebar-width);
  border-right: 1px solid var(--app-sidebar-border);
  background: var(--app-sidebar-bg);
  box-shadow: var(--app-shadow-brand);
}

.app-layout__brand {
  display: flex;
  align-items: center;
  gap: var(--app-space-3);
  height: var(--app-toolbar-height);
  padding: 0 var(--app-space-6);
  border-bottom: 1px solid var(--app-sidebar-border);
  color: var(--app-text-inverse);
}

.app-layout__brand-mark {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 34px;
  height: 34px;
  border-radius: var(--app-radius-sm);
  background: #409eff;
  color: var(--app-text-inverse);
  font-size: var(--app-font-size-md);
  font-weight: 800;
}

.app-layout__brand-copy {
  display: flex;
  min-width: 0;
  flex-direction: column;
  gap: 2px;
  line-height: 1.15;
}

.app-layout__brand-copy strong {
  overflow: hidden;
  font-size: var(--app-font-size-lg);
  font-weight: 600;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.app-layout__brand-copy small {
  color: var(--app-sidebar-text-muted);
  font-size: 11px;
  font-weight: 600;
  letter-spacing: 0;
  text-transform: uppercase;
}

.app-layout__nav {
  display: flex;
  flex-direction: column;
  gap: 2px;
  padding: var(--app-space-4) 12px;
}

.app-layout__nav-group {
  display: flex;
  flex-direction: column;
  gap: 2px;
}

.app-layout__nav-item {
  display: flex;
  align-items: center;
  gap: var(--app-space-3);
  min-height: 42px;
  padding: 0 var(--app-space-3);
  border-radius: var(--app-radius-sm);
  color: var(--app-sidebar-text);
  font-size: var(--app-font-size-md);
  font-weight: 500;
  text-decoration: none;
  transition: background-color 160ms ease, box-shadow 160ms ease, color 160ms ease;
}

.app-layout__nav-button {
  width: 100%;
  border: 0;
  background: transparent;
  cursor: pointer;
  font: inherit;
  text-align: left;
}

.app-layout__nav-item.has-children {
  font-weight: 500;
}

.app-layout__nav-item:hover {
  background: rgba(255, 255, 255, 0.05);
  color: var(--app-text-inverse);
}

.app-layout__nav-item.is-active {
  background: var(--app-sidebar-active-soft);
  color: var(--app-sidebar-active);
  font-weight: 500;
}

.app-layout__nav-icon {
  width: 18px;
  font-size: 18px;
}

.app-layout__nav-children {
  display: flex;
  flex-direction: column;
  gap: 2px;
  padding-left: 34px;
}

.app-layout__nav-child {
  display: inline-flex;
  align-items: center;
  min-height: 32px;
  padding: 0 12px;
  border-radius: var(--app-radius-sm);
  color: var(--app-sidebar-text-muted);
  font-size: var(--app-font-size-sm);
  line-height: var(--app-line-height-sm);
  text-decoration: none;
  transition: background-color 160ms ease, color 160ms ease;
}

.app-layout__nav-child:hover {
  background: rgba(255, 255, 255, 0.08);
  color: var(--app-text-inverse);
}

.app-layout__nav-child.is-active {
  background: var(--app-sidebar-active-soft);
  color: var(--app-sidebar-active);
}

.app-layout__body {
  flex: 1;
  min-width: 0;
  margin-left: var(--app-sidebar-width);
}

.app-layout__header {
  position: sticky;
  top: 0;
  z-index: 10;
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: var(--app-space-4);
  height: var(--app-toolbar-height);
  padding: 0 var(--app-space-5);
  border-bottom: 1px solid var(--app-border);
  background: var(--app-bg-panel);
  box-shadow: none;
}

.app-layout__header-copy {
  display: inline-flex;
  min-width: 0;
  align-items: center;
  gap: var(--app-space-3);
}

.app-layout__header-main {
  display: flex;
  min-width: 0;
  flex-direction: column;
  gap: 2px;
}

.app-layout__breadcrumb {
  display: inline-flex;
  min-width: 0;
  align-items: center;
  gap: 6px;
  color: var(--app-text-muted);
  font-size: 12px;
  font-weight: 400;
  line-height: 16px;
}

.app-layout__breadcrumb-item {
  display: inline-flex;
  min-width: 0;
  align-items: center;
  gap: 6px;
}

.app-layout__breadcrumb-item:not(:last-child)::after {
  color: var(--app-text-subtle);
  content: "/";
}

.app-layout__breadcrumb-item:last-child {
  overflow: hidden;
  color: var(--app-text-muted);
  text-overflow: ellipsis;
  white-space: nowrap;
}

.app-layout__header-actions {
  display: inline-flex;
  flex: 0 0 auto;
  align-items: center;
  gap: var(--app-space-3);
  min-width: 0;
}

.app-layout__header-divider {
  flex: 0 0 auto;
  width: 1px;
  height: 20px;
  background: var(--app-border);
}

.app-layout__workspace-button {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  max-width: 220px;
  min-width: 0;
  height: 32px;
  padding: 0 10px;
  border: 1px solid var(--app-border-soft);
  border-radius: var(--app-radius-sm);
  background: var(--app-bg-panel);
  color: var(--app-text-secondary);
  cursor: pointer;
  font: inherit;
  font-size: var(--app-font-size-sm);
  line-height: 20px;
  transition: background-color 150ms ease, border-color 150ms ease;
}

.app-layout__workspace-button:hover {
  background: var(--app-bg-muted);
  border-color: var(--app-primary);
}

.app-layout__workspace-button:disabled {
  cursor: not-allowed;
  opacity: 0.7;
}

.app-layout__workspace-button:focus-visible {
  outline: 0;
  border-color: var(--app-primary);
  box-shadow: 0 0 0 1px var(--app-primary) inset;
}

.app-layout__workspace-icon {
  flex: 0 0 auto;
  width: 14px;
  height: 14px;
  color: var(--app-primary);
}

.app-layout__workspace-name {
  overflow: hidden;
  min-width: 0;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.app-layout__workspace-caret {
  flex: 0 0 auto;
  width: 14px;
  height: 14px;
  color: var(--app-text-muted);
}

.app-layout__header-title {
  overflow: hidden;
  font-size: var(--app-font-size-lg);
  font-weight: 600;
  line-height: 22px;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.app-layout__user {
  display: inline-flex;
  flex: 0 0 auto;
  align-items: center;
  gap: var(--app-space-2);
  max-width: 260px;
  min-height: 36px;
  padding: 0;
  border: 0;
  border-radius: var(--app-radius-sm);
  background: transparent;
  color: var(--app-text-primary);
  cursor: pointer;
  transition: border-color 160ms ease, background-color 160ms ease;
}

.app-layout__user:disabled {
  cursor: wait;
  opacity: 0.72;
}

.app-layout__user:hover {
  border-color: transparent;
  background: transparent;
}

.app-layout__user-avatar {
  display: inline-flex;
  flex: 0 0 auto;
  align-items: center;
  justify-content: center;
  width: 28px;
  height: 28px;
  border-radius: 50%;
  background: #d1d5db;
  color: var(--app-text-inverse);
  font-size: var(--app-font-size-sm);
  font-weight: 700;
}

.app-layout__user-main {
  display: flex;
  min-width: 0;
  flex-direction: column;
  align-items: flex-start;
  line-height: 1.2;
}

.app-layout__user-name {
  max-width: 150px;
  overflow: hidden;
  font-size: var(--app-font-size-sm);
  font-weight: 600;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.app-layout__user-role {
  max-width: 150px;
  overflow: hidden;
  color: var(--app-text-muted);
  font-size: var(--app-font-size-xs);
  text-overflow: ellipsis;
  white-space: nowrap;
}

.app-layout__user-arrow {
  flex: 0 0 auto;
  color: var(--app-text-muted);
  font-size: 14px;
}

.app-layout__main {
  min-height: calc(100dvh - var(--app-toolbar-height));
  padding: var(--app-space-4);
}

@media (max-width: 1024px) {
  .app-layout {
    flex-direction: column;
    min-width: 0;
  }

  .app-layout__sidebar {
    position: static;
    width: 100%;
  }

  .app-layout__nav {
    gap: var(--app-space-2);
    padding: var(--app-space-3);
  }

  .app-layout__nav-item,
  .app-layout__nav-child {
    min-width: 0;
  }

  .app-layout__nav-children {
    padding-left: 30px;
  }

  .app-layout__body {
    margin-left: 0;
  }

  .app-layout__header {
    padding: 0 var(--app-space-4);
  }

  .app-layout__main {
    padding: var(--app-space-4);
  }
}

@media (max-width: 720px) {
  .app-layout__header {
    align-items: flex-start;
    height: auto;
    min-height: var(--app-toolbar-height);
    padding-block: var(--app-space-3);
  }

  .app-layout__header-copy {
    flex: 1 1 auto;
    flex-wrap: wrap;
    gap: var(--app-space-2);
  }

  .app-layout__header-title {
    max-width: 100%;
  }

  .app-layout__user {
    max-width: 156px;
  }

  .app-layout__header-actions {
    gap: var(--app-space-2);
  }

  .app-layout__header-divider {
    display: none;
  }

  .app-layout__workspace-button {
    max-width: 132px;
  }

  .app-layout__user-name,
  .app-layout__user-role {
    max-width: 72px;
  }
}
</style>
