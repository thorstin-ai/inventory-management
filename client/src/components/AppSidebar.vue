<template>
  <aside class="sidebar" :class="{ 'sidebar-drawer-open': open, 'is-collapsed': collapsed }">
    <!-- Brand block -->
    <div class="sidebar-brand">
      <div class="brand-mark" aria-hidden="true">
        <svg width="28" height="28" viewBox="0 0 28 28" fill="none" xmlns="http://www.w3.org/2000/svg">
          <!-- Stacked cubes glyph -->
          <path d="M14 3L23 8V14L14 19L5 14V8L14 3Z" stroke="var(--amber)" stroke-width="1.5" stroke-linejoin="round"/>
          <path d="M14 19L23 14M14 19L5 14M14 19V25" stroke="var(--amber)" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
          <path d="M5 14L14 9L23 14" stroke="var(--amber)" stroke-width="1.5" stroke-linejoin="round"/>
        </svg>
      </div>
      <div class="brand-text">
        <span class="brand-name">{{ t('nav.companyName') }}</span>
        <span class="brand-subtitle">{{ t('nav.subtitle') }}</span>
      </div>
    </div>

    <!-- Nav links -->
    <nav class="sidebar-nav" aria-label="Main navigation">
      <router-link
        v-for="item in navItems"
        :key="item.to"
        :to="item.to"
        class="nav-item"
        :class="{ 'nav-item--active': isActive(item.to) }"
        :aria-current="isActive(item.to) ? 'page' : undefined"
        :aria-label="t(item.labelKey)"
        @click="handleNavClick"
      >
        <span class="nav-icon" aria-hidden="true" v-html="item.icon"></span>
        <span class="nav-label">{{ t(item.labelKey) }}</span>
      </router-link>
    </nav>

    <!-- Footer collapse toggle — hidden at <=1024px via media query -->
    <div class="sidebar-footer">
      <button
        class="sidebar-toggle"
        type="button"
        @click="$emit('toggle-collapsed')"
        :aria-expanded="!collapsed"
        :aria-label="collapsed ? 'Expand sidebar' : 'Collapse sidebar'"
      >
        <span class="toggle-chevron" :class="{ 'is-rotated': collapsed }" aria-hidden="true">
          <svg width="16" height="16" viewBox="0 0 16 16" fill="none" xmlns="http://www.w3.org/2000/svg">
            <path d="M10 12L6 8L10 4" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
          </svg>
        </span>
        <span class="toggle-label">Collapse</span>
      </button>
    </div>
  </aside>
</template>

<script setup>
import { computed } from 'vue'
import { useRoute } from 'vue-router'
import { useI18n } from '../composables/useI18n'

const props = defineProps({
  open: {
    type: Boolean,
    default: false
  },
  collapsed: {
    type: Boolean,
    default: false
  }
})

const emit = defineEmits(['close', 'toggle-collapsed'])

const route = useRoute()
const { t } = useI18n()

const isActive = (path) => {
  if (path === '/') return route.path === '/'
  return route.path === path
}

const handleNavClick = () => {
  emit('close')
}

// Inline SVG icons — stroke=currentColor, stroke-width 1.5, no fill
const icons = {
  overview: `<svg width="18" height="18" viewBox="0 0 18 18" fill="none" xmlns="http://www.w3.org/2000/svg">
    <rect x="2" y="2" width="6" height="6" rx="1" stroke="currentColor" stroke-width="1.5"/>
    <rect x="10" y="2" width="6" height="6" rx="1" stroke="currentColor" stroke-width="1.5"/>
    <rect x="2" y="10" width="6" height="6" rx="1" stroke="currentColor" stroke-width="1.5"/>
    <rect x="10" y="10" width="6" height="6" rx="1" stroke="currentColor" stroke-width="1.5"/>
  </svg>`,

  inventory: `<svg width="18" height="18" viewBox="0 0 18 18" fill="none" xmlns="http://www.w3.org/2000/svg">
    <path d="M9 2L15.5 5.5V12.5L9 16L2.5 12.5V5.5L9 2Z" stroke="currentColor" stroke-width="1.5" stroke-linejoin="round"/>
    <path d="M9 16V9.5" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
    <path d="M15.5 5.5L9 9.5L2.5 5.5" stroke="currentColor" stroke-width="1.5" stroke-linejoin="round"/>
  </svg>`,

  orders: `<svg width="18" height="18" viewBox="0 0 18 18" fill="none" xmlns="http://www.w3.org/2000/svg">
    <rect x="3" y="2" width="12" height="14" rx="1.5" stroke="currentColor" stroke-width="1.5"/>
    <path d="M6 6H12" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
    <path d="M6 9H12" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
    <path d="M6 12H9" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
  </svg>`,

  finance: `<svg width="18" height="18" viewBox="0 0 18 18" fill="none" xmlns="http://www.w3.org/2000/svg">
    <circle cx="9" cy="9" r="6.5" stroke="currentColor" stroke-width="1.5"/>
    <path d="M9 5.5V6.5" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
    <path d="M9 11.5V12.5" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
    <path d="M7 7.5C7 7.5 7 6.5 9 6.5C11 6.5 11 8 9 9C7 10 7 11.5 9 11.5C11 11.5 11 10.5 11 10.5" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
  </svg>`,

  demand: `<svg width="18" height="18" viewBox="0 0 18 18" fill="none" xmlns="http://www.w3.org/2000/svg">
    <path d="M2 13L6.5 8L10 11L16 4" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
    <path d="M12 4H16V8" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
  </svg>`,

  reports: `<svg width="18" height="18" viewBox="0 0 18 18" fill="none" xmlns="http://www.w3.org/2000/svg">
    <rect x="3" y="2" width="12" height="14" rx="1.5" stroke="currentColor" stroke-width="1.5"/>
    <path d="M6 6H12" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
    <path d="M6 9H12" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
    <path d="M6 12H8" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
    <path d="M10 10.5L12 13L15 10" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
  </svg>`
}

const navItems = [
  { to: '/', labelKey: 'nav.overview', icon: icons.overview },
  { to: '/inventory', labelKey: 'nav.inventory', icon: icons.inventory },
  { to: '/orders', labelKey: 'nav.orders', icon: icons.orders },
  { to: '/spending', labelKey: 'nav.finance', icon: icons.finance },
  { to: '/demand', labelKey: 'nav.demandForecast', icon: icons.demand },
  { to: '/reports', labelKey: 'nav.reports', icon: icons.reports }
]
</script>

<style scoped>
.sidebar {
  position: sticky;
  top: 0;
  height: 100vh;
  display: flex;
  flex-direction: column;
  background: var(--sidebar-bg);
  border-right: 1px solid var(--graphite-line);
  padding: var(--space-5) var(--space-3);
  overflow-y: auto;
  overflow-x: hidden;
  flex-shrink: 0;
  transition: transform var(--transition), padding var(--transition), align-items var(--transition);
}

/* Brand block */
.sidebar-brand {
  display: flex;
  align-items: center;
  gap: var(--space-3);
  padding: var(--space-2) var(--space-3);
  margin-bottom: var(--space-6);
  flex-shrink: 0;
  transition: padding var(--transition), gap var(--transition), justify-content var(--transition);
}

.brand-mark {
  flex-shrink: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 32px;
  height: 32px;
}

.brand-text {
  display: flex;
  flex-direction: column;
  gap: 2px;
  min-width: 0;
  transition: opacity var(--transition);
}

.brand-name {
  font-family: var(--font-display);
  font-size: var(--text-md);
  font-weight: var(--fw-bold);
  color: var(--sidebar-active-text);
  letter-spacing: var(--tracking-tight);
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.brand-subtitle {
  font-size: var(--text-xs);
  color: var(--sidebar-text-muted);
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

/* Nav */
.sidebar-nav {
  display: flex;
  flex-direction: column;
  gap: var(--space-1);
  flex: 1;
}

.nav-item {
  display: flex;
  align-items: center;
  gap: var(--space-3);
  padding: var(--space-2) var(--space-3);
  border-radius: var(--radius-md);
  text-decoration: none;
  color: var(--sidebar-text);
  font-family: var(--font-body);
  font-size: var(--text-base);
  font-weight: var(--fw-medium);
  transition: background var(--transition-fast), color var(--transition-fast), padding var(--transition), gap var(--transition);
  position: relative;
  outline: none;
}

.nav-item:hover:not(.nav-item--active) {
  background: var(--graphite-raised);
  color: #ffffff;
}

.nav-item:focus-visible {
  box-shadow: 0 0 0 2px var(--amber);
}

/* Active treatment — amber left-edge tick */
.nav-item--active {
  background: var(--sidebar-active-bg);
  color: var(--sidebar-active-text);
}

.nav-item--active::before {
  content: '';
  position: absolute;
  left: 0;
  top: 50%;
  transform: translateY(-50%);
  width: 3px;
  height: 60%;
  background: var(--amber);
  border-radius: 0 var(--radius-sm) var(--radius-sm) 0;
}

/* Icon */
.nav-icon {
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
  width: 18px;
  height: 18px;
  color: inherit;
}

.nav-label {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  transition: opacity var(--transition);
}

/* ==============================
   Footer collapse toggle
   ============================== */
.sidebar-footer {
  flex-shrink: 0;
  margin-top: var(--space-2);
  padding-top: var(--space-2);
  border-top: 1px solid var(--graphite-line);
}

.sidebar-toggle {
  display: flex;
  align-items: center;
  width: 100%;
  gap: var(--space-3);
  padding: var(--space-2) var(--space-3);
  border-radius: var(--radius-md);
  background: none;
  border: none;
  color: var(--sidebar-text-muted);
  cursor: pointer;
  font: inherit;
  font-size: var(--text-sm);
  transition: background var(--transition-fast), color var(--transition-fast);
}

.sidebar-toggle:hover {
  background: var(--graphite-raised);
  color: #fff;
}

.sidebar-toggle:focus-visible {
  box-shadow: 0 0 0 2px var(--amber);
  outline: none;
}

.toggle-chevron {
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
  transition: transform var(--transition);
}

.toggle-chevron.is-rotated {
  transform: rotate(180deg);
}

.toggle-label {
  white-space: nowrap;
  overflow: hidden;
}

/* ==============================
   Collapsed state (class-driven)
   ============================== */
.sidebar.is-collapsed {
  align-items: center;
  padding-left: var(--space-2);
  padding-right: var(--space-2);
}

.sidebar.is-collapsed .brand-text {
  display: none;
}

.sidebar.is-collapsed .sidebar-brand {
  justify-content: center;
  gap: 0;
}

.sidebar.is-collapsed .nav-item {
  justify-content: center;
  gap: 0;
  padding: var(--space-3) var(--space-2);
}

.sidebar.is-collapsed .nav-label {
  display: none;
}

.sidebar.is-collapsed .toggle-label {
  display: none;
}

.sidebar.is-collapsed .sidebar-toggle {
  justify-content: center;
}

/* ==============================
   Responsive: hide footer toggle at <=1024px (auto-collapsed by media query)
   ============================== */
@media (max-width: 1024px) {
  .sidebar-footer {
    display: none;
  }
}

/* ==============================
   Responsive: mobile drawer at 768px
   ============================== */
@media (max-width: 768px) {
  .sidebar {
    position: fixed;
    top: 0;
    left: 0;
    bottom: 0;
    z-index: 200;
    width: var(--sidebar-width);
    transform: translateX(-100%);
    align-items: flex-start;
    padding: var(--space-5) var(--space-3);
  }

  /* When mobile drawer is open */
  .sidebar.sidebar-drawer-open {
    transform: translateX(0);
  }

  /* Ensure full layout in open drawer (override any is-collapsed styles) */
  .sidebar.sidebar-drawer-open .sidebar-brand {
    padding: var(--space-2) var(--space-3);
    justify-content: flex-start;
    gap: var(--space-3);
  }

  .sidebar.sidebar-drawer-open .brand-text {
    display: flex;
  }

  .sidebar.sidebar-drawer-open .brand-subtitle {
    display: block;
  }

  .sidebar.sidebar-drawer-open .nav-item {
    justify-content: flex-start;
    padding: var(--space-2) var(--space-3);
    gap: var(--space-3);
  }

  .sidebar.sidebar-drawer-open .nav-label {
    display: block;
  }
}

@media (prefers-reduced-motion: reduce) {
  .sidebar,
  .nav-item,
  .toggle-chevron {
    transition: none;
  }
}
</style>
