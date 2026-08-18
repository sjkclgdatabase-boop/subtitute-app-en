<template>
  <aside 
    :class="isSidebarCollapsed ? 'w-20' : 'w-72'" 
    class="bg-slate-900 text-slate-300 flex flex-col h-screen sticky top-0 left-0 shrink-0 z-40 border-r border-slate-800 print:hidden shadow-xl transition-all duration-300"
  >
    
    <!-- Top: Hamburger Toggle Button + Logo + School Name -->
    <div class="p-4 border-b border-slate-800 flex items-center justify-between gap-2 select-none">
      
      <!-- Logo & School Name (Hidden when collapsed) -->
      <div v-show="!isSidebarCollapsed" class="flex items-center gap-2.5 cursor-pointer overflow-hidden flex-1 min-w-0" @click="router.push('/')">
        <img 
          :src="currentLogo" 
          alt="Logo" 
          class="w-10 h-10 object-contain rounded-xl bg-slate-800 p-1 shadow-sm shrink-0"
        />
        <div class="flex flex-col justify-center overflow-hidden min-w-0">
          <span class="font-bold text-[14px] tracking-tight text-white leading-tight truncate">
            {{ currentSchoolName }}
          </span>
          <span class="text-[8px] font-bold text-indigo-400 tracking-wider mt-0.5 uppercase">
            SMART ACADEMIC MANAGEMENT
          </span>
        </div>
      </div>

      <!-- Centered small Logo when collapsed -->
      <div v-show="isSidebarCollapsed" class="mx-auto cursor-pointer" @click="router.push('/')" title="Home">
        <img :src="currentLogo" alt="Logo" class="w-9 h-9 object-contain rounded-xl bg-slate-800 p-1 shrink-0 shadow-sm" />
      </div>

      <!-- ⭐️ Sidebar Collapse/Expand SVG Icon Button -->
      <button 
        @click="toggleSidebar" 
        class="w-9 h-9 rounded-xl bg-slate-800/80 hover:bg-slate-700 text-slate-300 hover:text-white flex items-center justify-center transition cursor-pointer shrink-0 shadow-sm"
        :title="isSidebarCollapsed ? 'Expand Sidebar' : 'Collapse Sidebar'"
      >
        <svg class="w-5 h-5 transition-transform duration-300" :class="isSidebarCollapsed ? 'rotate-180' : ''" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
          <rect x="3" y="3" width="18" height="18" rx="4" />
          <line x1="9" y1="3" x2="9" y2="21" />
        </svg>
      </button>
    </div>

    <!-- Middle: Scrollable Navigation Menu -->
    <div class="flex-1 overflow-y-auto px-3 py-4 space-y-1.5 scrollbar-thin scrollbar-thumb-slate-700">
      <router-link 
        v-for="(item, index) in navItems" 
        :key="index"
        :to="item.path" 
        class="flex items-center gap-3.5 px-3.5 py-3 rounded-2xl text-xs font-bold transition-all duration-200 text-slate-400 hover:text-white hover:bg-slate-800/80 group/item relative"
        exact-active-class="!text-white !bg-indigo-600 shadow-md shadow-indigo-600/20"
        :title="isSidebarCollapsed ? item.name : ''"
      >
        <span class="text-base shrink-0 group-hover/item:scale-110 transition-transform">{{ item.icon }}</span>
        <span v-show="!isSidebarCollapsed" class="truncate">{{ item.name }}</span>
      </router-link>
    </div>

    <!-- Bottom Actions Area: Multi-language Switch + Logout -->
    <div class="p-3 border-t border-slate-800 bg-slate-950/40 space-y-1.5">
      
      <!-- 1. Switch to Chinese Version -->
      <button 
        @click="switchToLanguage('https://subtitute-app.vercel.app')" 
        class="w-full flex items-center gap-3.5 px-3.5 py-2.5 rounded-2xl text-xs font-bold text-indigo-400 hover:bg-indigo-500/10 hover:text-indigo-300 transition-colors cursor-pointer"
        :title="isSidebarCollapsed ? '中文' : ''"
      >
        <span class="text-base shrink-0">🇨🇳</span>
        <span v-show="!isSidebarCollapsed" class="truncate">中文</span>
      </button>

      <!-- 2. Switch to Malay Version -->
      <button 
        @click="switchToLanguage('https://subtitute-app-bm.vercel.app')" 
        class="w-full flex items-center gap-3.5 px-3.5 py-2.5 rounded-2xl text-xs font-bold text-cyan-400 hover:bg-cyan-500/10 hover:text-cyan-300 transition-colors cursor-pointer"
        :title="isSidebarCollapsed ? 'Bahasa Melayu' : ''"
      >
        <span class="text-base shrink-0">🇲🇾</span>
        <span v-show="!isSidebarCollapsed" class="truncate">Bahasa Melayu</span>
      </button>

      <!-- 3. Logout Button -->
      <button 
        @click="logout" 
        class="w-full flex items-center justify-between px-3.5 py-2.5 rounded-2xl text-xs font-bold text-rose-400 hover:bg-rose-500/10 hover:text-rose-300 transition-colors cursor-pointer"
        :title="isSidebarCollapsed ? 'Logout' : ''"
      >
        <div class="flex items-center gap-3.5 truncate">
          <span class="text-base shrink-0">🚪</span>
          <span v-show="!isSidebarCollapsed" class="truncate">LOGOUT</span>
        </div>
        <svg v-show="!isSidebarCollapsed" class="w-4 h-4 shrink-0" fill="none" viewBox="0 0 24 24" stroke="currentColor">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M17 16l4-4m0 0l-4-4m4 4H7m6 4v1a3 3 0 01-3 3H6a3 3 0 01-3-3V7a3 3 0 013-3h4a3 3 0 013 3v1" />
        </svg>
      </button>
    </div>

  </aside>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { useRouter } from 'vue-router'
import { supabase } from '../services/supabase'
import { useToast } from '../utils/toast'

const router = useRouter()
const toast = useToast()

const currentLogo = ref('/logo.png')
const currentSchoolName = ref('SMART ACADEMIC MANAGEMENT SYSTEM')

// Sidebar collapse state control (default expanded as false)
const isSidebarCollapsed = ref(false)

const toggleSidebar = () => {
  isSidebarCollapsed.value = !isSidebarCollapsed.value
}

const loadSidebarIdentity = async () => {
  const cachedLogo = localStorage.getItem('school_logo')
  const cachedName = localStorage.getItem('school_name')
  
  if (cachedLogo && cachedLogo.trim() !== '') currentLogo.value = cachedLogo
  if (cachedName && cachedName.trim() !== '') currentSchoolName.value = cachedName

  try {
    const { data } = await supabase.from('school_settings').select('*').eq('id', 1).single()
    if (data) {
      if (data.logo_url && data.logo_url.trim() !== '') {
        currentLogo.value = data.logo_url
        localStorage.setItem('school_logo', data.logo_url)
      }
      if (data.school_name && data.school_name.trim() !== '') {
        currentSchoolName.value = data.school_name
        localStorage.setItem('school_name', data.school_name)
      }
    }
  } catch (err) {
    console.error('FAILED TO LOAD SIDEBAR SETTINGS:', err)
  }
}

onMounted(() => {
  loadSidebarIdentity()
})

const navItems = [
  { name: 'DASHBOARD OVERVIEW', path: '/', icon: '📊' },
  { name: 'TEACHER LEAVE ENTRY', path: '/leave-entry', icon: '📝' },
  { name: 'TEACHER PROFILES', path: '/teachers', icon: '👩‍🏫' },
  { name: 'MASTER TIMETABLE', path: '/timetable', icon: '📅' },
  { name: 'CLASS TIMETABLE', path: '/class-timetable', icon: '🏫' }, // 🟢 英文版新增的班级课表选单
  { name: 'SUBSTITUTE RECORDS', path: '/records', icon: '🔄' },
  { name: 'MMI REPORT CENTER', path: '/statistics', icon: '📈' },
  { name: 'MMI INTERRUPTION LOG', path: '/mmi-interruption', icon: '⚠️' }, 
  { name: 'ANALYSIS CENTER', path: '/subject-analysis', icon: '🎯' },
  { name: 'SYSTEM SETTINGS', path: '/settings', icon: '⚙️' } 
]

// Unified cross-domain seamless jump with Token forwarding
const switchToLanguage = async (targetUrl) => {
  try {
    const { data: { session } } = await supabase.auth.getSession()
    if (session) {
      window.location.href = `${targetUrl}/?access_token=${session.access_token}&refresh_token=${session.refresh_token}`
    } else {
      window.location.href = targetUrl
    }
  } catch (error) {
    toast.error("Failed to switch: " + error.message)
  }
}

const logout = async () => {
  try {
    await supabase.auth.signOut()
    toast.success("Successfully logged out")
    window.location.href = '/login'
  } catch (error) {
    toast.error("Logout failed: " + error.message)
  }
}
</script>