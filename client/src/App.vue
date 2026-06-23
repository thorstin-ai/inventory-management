<template>
  <div class="app-shell" :class="{ 'sidebar-open': sidebarOpen, 'sidebar-collapsed': collapsed }">
    <AppSidebar :open="sidebarOpen" :collapsed="collapsed" @close="sidebarOpen = false" @toggle-collapsed="toggleCollapsed" />

    <!-- Mobile backdrop -->
    <div class="sidebar-backdrop" @click="sidebarOpen = false"></div>

    <div class="app-main">
      <header class="topbar">
        <!-- Hamburger — mobile only -->
        <button
          class="topbar-menu"
          @click="sidebarOpen = true"
          aria-label="Open navigation"
        >
          <svg width="20" height="20" viewBox="0 0 20 20" fill="none" xmlns="http://www.w3.org/2000/svg">
            <path d="M3 5H17" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
            <path d="M3 10H17" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
            <path d="M3 15H17" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
          </svg>
        </button>

        <h1 class="topbar-title">{{ currentPageTitle }}</h1>

        <div class="topbar-actions">
          <LanguageSwitcher />
          <ProfileMenu
            @show-profile-details="showProfileDetails = true"
            @show-tasks="showTasks = true"
          />
        </div>
      </header>

      <FilterBar />

      <main class="content">
        <router-view />
      </main>
    </div>

    <ProfileDetailsModal
      :is-open="showProfileDetails"
      @close="showProfileDetails = false"
    />

    <TasksModal
      :is-open="showTasks"
      :tasks="tasks"
      @close="showTasks = false"
      @add-task="addTask"
      @delete-task="deleteTask"
      @toggle-task="toggleTask"
    />
  </div>
</template>

<script>
import { ref, onMounted, onUnmounted, computed } from 'vue'
import { useRoute } from 'vue-router'
import { api } from './api'
import { useAuth } from './composables/useAuth'
import { useI18n } from './composables/useI18n'
import FilterBar from './components/FilterBar.vue'
import ProfileMenu from './components/ProfileMenu.vue'
import ProfileDetailsModal from './components/ProfileDetailsModal.vue'
import TasksModal from './components/TasksModal.vue'
import LanguageSwitcher from './components/LanguageSwitcher.vue'
import AppSidebar from './components/AppSidebar.vue'

export default {
  name: 'App',
  components: {
    FilterBar,
    ProfileMenu,
    ProfileDetailsModal,
    TasksModal,
    LanguageSwitcher,
    AppSidebar
  },
  setup() {
    const { currentUser } = useAuth()
    const { t } = useI18n()
    const route = useRoute()

    const showProfileDetails = ref(false)
    const showTasks = ref(false)
    const apiTasks = ref([])

    // Mobile sidebar drawer state
    const sidebarOpen = ref(false)

    // Current page title from route
    const currentPageTitle = computed(() => {
      const pathTitleMap = {
        '/': t('nav.overview'),
        '/inventory': t('nav.inventory'),
        '/orders': t('nav.orders'),
        '/spending': t('nav.finance'),
        '/demand': t('nav.demandForecast'),
        '/reports': t('nav.reports')
      }
      return pathTitleMap[route.path] || ''
    })

    // Merge mock tasks from currentUser with API tasks
    const tasks = computed(() => {
      return [...currentUser.value.tasks, ...apiTasks.value]
    })

    const loadTasks = async () => {
      try {
        apiTasks.value = await api.getTasks()
      } catch (err) {
        console.error('Failed to load tasks:', err)
      }
    }

    const addTask = async (taskData) => {
      try {
        const newTask = await api.createTask(taskData)
        // Add new task to the beginning of the array
        apiTasks.value.unshift(newTask)
      } catch (err) {
        console.error('Failed to add task:', err)
      }
    }

    const deleteTask = async (taskId) => {
      try {
        // Check if it's a mock task (from currentUser)
        const isMockTask = currentUser.value.tasks.some(t => t.id === taskId)

        if (isMockTask) {
          // Remove from mock tasks
          const index = currentUser.value.tasks.findIndex(t => t.id === taskId)
          if (index !== -1) {
            currentUser.value.tasks.splice(index, 1)
          }
        } else {
          // Remove from API tasks
          await api.deleteTask(taskId)
          apiTasks.value = apiTasks.value.filter(t => t.id !== taskId)
        }
      } catch (err) {
        console.error('Failed to delete task:', err)
      }
    }

    const toggleTask = async (taskId) => {
      try {
        // Check if it's a mock task (from currentUser)
        const mockTask = currentUser.value.tasks.find(t => t.id === taskId)

        if (mockTask) {
          // Toggle mock task status
          mockTask.status = mockTask.status === 'pending' ? 'completed' : 'pending'
        } else {
          // Toggle API task
          const updatedTask = await api.toggleTask(taskId)
          const index = apiTasks.value.findIndex(t => t.id === taskId)
          if (index !== -1) {
            apiTasks.value[index] = updatedTask
          }
        }
      } catch (err) {
        console.error('Failed to toggle task:', err)
      }
    }

    // Sidebar collapse state — desktop user preference
    const userCollapsed = ref(localStorage.getItem('sidebar-collapsed') === 'true')
    const isCompact = ref(false) // <=1024px
    const isMobile  = ref(false) // <=768px
    let mqCompact, mqMobile
    const syncMq = () => {
      isCompact.value = mqCompact.matches
      isMobile.value  = mqMobile.matches
    }

    // desktop honors user; 768–1024 forces icons-only; <=768 is a drawer (not "collapsed")
    const collapsed = computed(() => !isMobile.value && (isCompact.value || userCollapsed.value))

    const toggleCollapsed = () => {
      userCollapsed.value = !userCollapsed.value
      localStorage.setItem('sidebar-collapsed', String(userCollapsed.value))
    }

    onMounted(() => {
      loadTasks()

      mqCompact = window.matchMedia('(max-width: 1024px)')
      mqMobile  = window.matchMedia('(max-width: 768px)')
      syncMq()
      mqCompact.addEventListener('change', syncMq)
      mqMobile.addEventListener('change', syncMq)
    })

    onUnmounted(() => {
      mqCompact && mqCompact.removeEventListener('change', syncMq)
      mqMobile  && mqMobile.removeEventListener('change', syncMq)
    })

    return {
      t,
      showProfileDetails,
      showTasks,
      tasks,
      addTask,
      deleteTask,
      toggleTask,
      sidebarOpen,
      currentPageTitle,
      collapsed,
      toggleCollapsed
    }
  }
}
</script>

<style>
/* ============================================================
   DESIGN TOKENS — Industrial Slate
   ============================================================ */
:root {
  /* Type */
  --font-display: 'Space Grotesk', 'Inter', system-ui, sans-serif;
  --font-body: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  --font-mono: 'IBM Plex Mono', 'SFMono-Regular', Consolas, monospace;
  --text-xs: .75rem;
  --text-sm: .8125rem;
  --text-base: .875rem;
  --text-md: .9375rem;
  --text-lg: 1.125rem;
  --text-xl: 1.375rem;
  --text-2xl: 1.75rem;
  --text-3xl: 2rem;
  --fw-normal: 400;
  --fw-medium: 500;
  --fw-semibold: 600;
  --fw-bold: 700;
  --tracking-tight: -0.02em;
  --tracking-wide: .04em;

  /* Spacing (4px scale) */
  --space-1: .25rem;
  --space-2: .5rem;
  --space-3: .75rem;
  --space-4: 1rem;
  --space-5: 1.25rem;
  --space-6: 1.5rem;
  --space-8: 2rem;
  --space-10: 2.5rem;
  --space-12: 3rem;

  /* Radius */
  --radius-sm: 6px;
  --radius-md: 8px;
  --radius-lg: 10px;
  --radius-full: 9999px;

  /* Elevation */
  --shadow-xs: 0 1px 2px rgba(20, 24, 31, .05);
  --shadow-sm: 0 1px 2px rgba(20, 24, 31, .06), 0 1px 3px rgba(20, 24, 31, .08);
  --shadow-md: 0 2px 4px -1px rgba(20, 24, 31, .06), 0 4px 10px -2px rgba(20, 24, 31, .08);

  /* Industrial Slate palette */
  --graphite: #14181F;
  --graphite-raised: #1E242E;
  --graphite-line: rgba(231, 236, 243, .08);
  --paper: #F7F8FA;
  --surface: #FFFFFF;
  --surface-muted: #F1F3F6;
  --hairline: #E3E6EB;
  --hairline-strong: #D3D8E0;
  --ink: #1A1F28;
  --ink-soft: #3C434F;
  --muted: #6B7480;
  --subtle: #9AA2AD;

  /* Signal amber — THE single accent, use sparingly */
  --amber: #E8A33D;
  --amber-strong: #D8902A;
  --amber-tint: #FBF1DF;

  /* Sidebar */
  --sidebar-bg: var(--graphite);
  --sidebar-text: #AEB6C2;
  --sidebar-text-muted: #6E7682;
  --sidebar-active-text: #FFFFFF;
  --sidebar-active-bg: rgba(232, 163, 61, .12);

  /* Status (functional) */
  --success: #2E9E6B;
  --success-tint: #E2F3EA;
  --success-ink: #1C6B47;
  --warning: #E8A33D;
  --warning-tint: #FBF1DF;
  --warning-ink: #8A5E16;
  --danger: #D6453D;
  --danger-tint: #FBE6E5;
  --danger-ink: #962019;
  --info: #3B6EA5;
  --info-tint: #E4ECF5;
  --info-ink: #234D78;

  /* Layout */
  --sidebar-width: 248px;
  --sidebar-width-collapsed: 76px;
  --topbar-height: 64px;
  --content-max: 1440px;
  --content-pad-x: var(--space-8);
  --content-pad-y: var(--space-6);
  --transition: 150ms ease;
  --transition-fast: 110ms ease;
}

/* ============================================================
   RESET
   ============================================================ */
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: var(--font-body);
  background: var(--paper);
  color: var(--ink);
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

/* ============================================================
   SHELL LAYOUT
   ============================================================ */
.app-shell {
  display: grid;
  grid-template-columns: var(--sidebar-width) 1fr;
  min-height: 100vh;
  background: var(--paper);
  transition: grid-template-columns var(--transition);
}

.app-shell.sidebar-collapsed {
  grid-template-columns: var(--sidebar-width-collapsed) 1fr;
}

.app-main {
  display: flex;
  flex-direction: column;
  min-width: 0; /* prevent wide tables blowing out the grid */
}

/* Mobile backdrop — hidden by default */
.sidebar-backdrop {
  display: none;
}

/* ============================================================
   TOPBAR
   ============================================================ */
.topbar {
  position: sticky;
  top: 0;
  z-index: 50;
  height: var(--topbar-height);
  background: var(--surface);
  border-bottom: 1px solid var(--hairline);
  display: flex;
  align-items: center;
  gap: var(--space-4);
  padding: 0 var(--content-pad-x);
}

.topbar-menu {
  display: none; /* desktop: hidden */
  align-items: center;
  justify-content: center;
  width: 36px;
  height: 36px;
  background: none;
  border: 1px solid var(--hairline);
  border-radius: var(--radius-md);
  color: var(--ink-soft);
  cursor: pointer;
  flex-shrink: 0;
  transition: background var(--transition-fast), border-color var(--transition-fast);
}

.topbar-menu:hover {
  background: var(--surface-muted);
  border-color: var(--hairline-strong);
}

.topbar-menu:focus-visible {
  outline: 2px solid var(--amber);
  outline-offset: 2px;
}

.topbar-title {
  font-family: var(--font-display);
  font-size: var(--text-lg);
  font-weight: var(--fw-semibold);
  color: var(--ink);
  letter-spacing: var(--tracking-tight);
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.topbar-actions {
  display: flex;
  align-items: center;
  gap: var(--space-3);
  margin-left: auto;
}

/* ============================================================
   CONTENT AREA
   ============================================================ */
.content {
  flex: 1;
  width: 100%;
  max-width: var(--content-max);
  margin: 0 auto;
  padding: var(--content-pad-y) var(--content-pad-x);
}

/* ============================================================
   PAGE HEADER
   ============================================================ */
.page-header {
  margin-bottom: var(--space-6);
}

.page-header h2 {
  font-family: var(--font-display);
  font-size: var(--text-3xl);
  font-weight: var(--fw-bold);
  color: var(--ink);
  margin-bottom: var(--space-2);
  letter-spacing: var(--tracking-tight);
}

.page-header p {
  color: var(--muted);
  font-size: var(--text-base);
}

/* ============================================================
   STATS GRID + STAT CARD
   ============================================================ */
.stats-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: var(--space-5);
  margin-bottom: var(--space-6);
}

.stat-card {
  background: var(--surface);
  padding: var(--space-5);
  border-radius: var(--radius-lg);
  border: 1px solid var(--hairline);
  box-shadow: var(--shadow-sm);
  transition: box-shadow var(--transition), border-color var(--transition);
}

.stat-card:hover {
  border-color: var(--hairline-strong);
  box-shadow: var(--shadow-md);
}

.stat-label {
  color: var(--muted);
  font-size: var(--text-xs);
  font-weight: var(--fw-semibold);
  text-transform: uppercase;
  letter-spacing: var(--tracking-wide);
  margin-bottom: var(--space-3);
}

.stat-value {
  font-family: var(--font-mono);
  font-variant-numeric: tabular-nums;
  font-size: var(--text-2xl);
  font-weight: var(--fw-semibold);
  color: var(--ink);
  letter-spacing: var(--tracking-tight);
}

.stat-card.warning .stat-value {
  color: var(--warning);
}

.stat-card.success .stat-value {
  color: var(--success);
}

.stat-card.danger .stat-value {
  color: var(--danger);
}

.stat-card.info .stat-value {
  color: var(--info);
}

/* ============================================================
   CARD
   ============================================================ */
.card {
  background: var(--surface);
  border-radius: var(--radius-lg);
  padding: var(--space-5);
  border: 1px solid var(--hairline);
  box-shadow: var(--shadow-sm);
  margin-bottom: var(--space-5);
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: var(--space-4);
  padding-bottom: var(--space-4);
  border-bottom: 1px solid var(--hairline);
}

.card-title {
  font-family: var(--font-display);
  font-size: var(--text-lg);
  font-weight: var(--fw-semibold);
  color: var(--ink);
  letter-spacing: var(--tracking-tight);
}

/* ============================================================
   TABLE
   ============================================================ */
.table-container {
  overflow-x: auto;
}

table {
  width: 100%;
  border-collapse: collapse;
}

thead {
  background: var(--surface-muted);
  border-top: 1px solid var(--hairline);
  border-bottom: 1px solid var(--hairline);
}

th {
  text-align: left;
  padding: var(--space-2) var(--space-3);
  font-weight: var(--fw-semibold);
  color: var(--muted);
  font-size: var(--text-xs);
  text-transform: uppercase;
  letter-spacing: var(--tracking-wide);
}

td {
  padding: var(--space-2) var(--space-3);
  border-top: 1px solid var(--hairline);
  color: var(--ink-soft);
  font-size: var(--text-base);
}

tbody tr {
  transition: background-color var(--transition-fast);
}

tbody tr:hover {
  background: var(--surface-muted);
}

/* Numeric utility class */
.num {
  font-family: var(--font-mono);
  font-variant-numeric: tabular-nums;
}

/* ============================================================
   BADGES
   ============================================================ */
.badge {
  display: inline-block;
  padding: .3125rem .625rem;
  border-radius: var(--radius-sm);
  font-size: var(--text-xs);
  font-weight: var(--fw-semibold);
  text-transform: uppercase;
  letter-spacing: var(--tracking-wide);
}

.badge.success {
  background: var(--success-tint);
  color: var(--success-ink);
}

.badge.warning {
  background: var(--warning-tint);
  color: var(--warning-ink);
}

.badge.danger {
  background: var(--danger-tint);
  color: var(--danger-ink);
}

.badge.info {
  background: var(--info-tint);
  color: var(--info-ink);
}

.badge.increasing {
  background: var(--success-tint);
  color: var(--success-ink);
}

.badge.decreasing {
  background: var(--danger-tint);
  color: var(--danger-ink);
}

.badge.stable {
  background: var(--info-tint);
  color: var(--info-ink);
}

.badge.high {
  background: var(--danger-tint);
  color: var(--danger-ink);
}

.badge.medium {
  background: var(--warning-tint);
  color: var(--warning-ink);
}

.badge.low {
  background: var(--info-tint);
  color: var(--info-ink);
}

/* ============================================================
   LOADING + ERROR STATES
   ============================================================ */
.loading {
  text-align: center;
  padding: var(--space-12);
  color: var(--muted);
  font-size: var(--text-base);
}

.error {
  background: var(--danger-tint);
  border: 1px solid var(--danger);
  color: var(--danger-ink);
  padding: var(--space-4);
  border-radius: var(--radius-md);
  margin: var(--space-4) 0;
  font-size: var(--text-base);
}

/* ============================================================
   RESPONSIVE — 768px: mobile drawer
   ============================================================ */
@media (max-width: 768px) {
  .app-shell {
    grid-template-columns: 1fr;
  }

  .topbar-menu {
    display: flex;
  }

  /* Backdrop visible when drawer is open */
  .app-shell.sidebar-open .sidebar-backdrop {
    display: block;
    position: fixed;
    inset: 0;
    z-index: 150;
    background: rgba(20, 24, 31, .45);
  }
}

@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    transition: none !important;
  }
}
</style>
