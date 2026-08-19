<template>
  <!-- 🌟 核心：给最外层一个定死的最小宽度 (min-w-[1024px]) -->
  <div class="p-4 sm:p-8 mx-auto min-h-screen space-y-8 min-w-[1024px]">
    
    <!-- Header Section -->
    <div class="bg-white rounded-3xl p-6 sm:p-8 shadow-sm ring-1 ring-slate-900/5 flex justify-between items-center gap-6">
      <div class="space-y-2 max-w-3xl">
        <h1 class="text-xl sm:text-2xl md:text-3xl font-extrabold tracking-tight text-transparent bg-clip-text bg-gradient-to-r from-slate-900 via-indigo-800 to-violet-800 flex items-center gap-3">
          <LayoutDashboard class="w-8 h-8 text-indigo-700 shrink-0" />
          SUBSTITUTE TEACHER MANAGEMENT CENTER
        </h1>
        <p class="text-slate-500 text-xs sm:text-sm font-medium leading-relaxed">
          REAL-TIME TEACHER LEAVE MONITORING & SUBSTITUTE TASK ASSIGNMENT TRACKING
        </p>
      </div>
      <router-link to="/leave-entry" class="group relative inline-flex items-center justify-center px-6 py-3 text-xs sm:text-sm font-bold text-white transition-all duration-200 bg-slate-900 rounded-2xl hover:bg-slate-800 hover:shadow-lg hover:shadow-slate-900/20 shrink-0">
        <span>NEW LEAVE ENTRY</span>
        <Plus class="w-4 h-4 ml-2 group-hover:translate-x-1 transition-transform shrink-0" />
      </router-link>
    </div>

    <!-- Core Metrics Cards -->
    <div class="grid grid-cols-3 gap-6">
      <div class="bg-white rounded-3xl p-6 shadow-sm ring-1 ring-slate-900/5 hover:shadow-md transition-all">
        <div class="flex items-center justify-between">
          <p class="text-slate-500 text-xs font-bold uppercase tracking-wider">PENDING SUBSTITUTE TASKS</p>
          <div class="w-10 h-10 rounded-2xl bg-amber-50 text-amber-600 flex items-center justify-center ring-1 ring-amber-500/20">
            <Clock3 class="w-5 h-5" />
          </div>
        </div>
        <p class="text-3xl font-black text-slate-900 mt-4">{{ pendingCount }}</p>
      </div>
      
      <div class="bg-white rounded-3xl p-6 shadow-sm ring-1 ring-slate-900/5 hover:shadow-md transition-all">
        <div class="flex items-center justify-between">
          <p class="text-slate-500 text-xs font-bold uppercase tracking-wider">ASSIGNMENT COMPLETED</p>
          <div class="w-10 h-10 rounded-2xl bg-emerald-50 text-emerald-600 flex items-center justify-center ring-1 ring-emerald-500/20">
            <CircleCheck class="w-5 h-5" />
          </div>
        </div>
        <p class="text-3xl font-black text-slate-900 mt-4">{{ assignedCount }}</p>
      </div>
      
      <div class="bg-white rounded-3xl p-6 shadow-sm ring-1 ring-slate-900/5 hover:shadow-md transition-all">
        <div class="flex items-center justify-between">
          <p class="text-slate-500 text-xs font-bold uppercase tracking-wider">TOTAL TASKS</p>
          <div class="w-10 h-10 rounded-2xl bg-indigo-50 text-indigo-600 flex items-center justify-center ring-1 ring-indigo-500/20">
            <ChartColumnBig class="w-5 h-5" />
          </div>
        </div>
        <p class="text-3xl font-black text-slate-900 mt-4">{{ leaveRequests.length }}</p>
      </div>
    </div>

    <!-- Filter & View Switcher Bar -->
    <div class="flex justify-between items-center gap-4 bg-white p-6 rounded-3xl shadow-sm ring-1 ring-slate-900/5">
      <div class="flex items-center gap-3">
        <div class="flex bg-slate-100 p-1.5 rounded-2xl">
          <button 
            @click="viewMode = 'today'"
            :class="viewMode === 'today' ? 'bg-slate-900 text-white shadow-sm' : 'text-slate-500 hover:text-slate-900'"
            class="px-4 py-2 rounded-xl text-xs font-bold transition-all cursor-pointer flex items-center gap-1.5"
          >
            <CalendarDays class="w-4 h-4" />
            TODAY ONLY
          </button>
          <button 
            @click="viewMode = 'grouped'"
            :class="viewMode === 'grouped' ? 'bg-slate-900 text-white shadow-sm' : 'text-slate-500 hover:text-slate-900'"
            class="px-4 py-2 rounded-xl text-xs font-bold transition-all cursor-pointer flex items-center gap-1.5"
          >
            <FolderOpen class="w-4 h-4" />
            GROUP BY DATE
          </button>
        </div>

        <input 
          v-if="viewMode === 'today'"
          type="date" 
          v-model="targetDate"
          class="bg-slate-50 border border-slate-200 px-3.5 h-11 rounded-2xl text-xs font-bold text-slate-700 focus:outline-none focus:ring-2 focus:ring-indigo-500 cursor-pointer"
        />
      </div>

      <div class="text-xs text-slate-500 font-bold">
        CURRENT DISPLAY: <span class="text-slate-900">{{ displayedRequests.length }}</span> LEAVE RECORDS
      </div>
    </div>

    <!-- Data Table Area -->
    <div class="bg-white rounded-3xl shadow-sm ring-1 ring-slate-900/5 overflow-hidden">
      
      <!-- Mode 1: Today Only -->
      <div v-if="viewMode === 'today'">
        <table class="w-full text-left border-collapse whitespace-nowrap">
          <thead>
            <tr class="bg-slate-50 border-b border-slate-100 text-slate-500 text-xs uppercase tracking-widest font-semibold select-none">
              <th class="py-4 pl-6 pr-4 cursor-pointer hover:text-indigo-600 transition" @click="handleSort('teacher')">
                ABSENT TEACHER <span v-if="sortKey === 'teacher'" class="text-indigo-600 font-bold ml-1">{{ sortOrder === 'asc' ? '▲' : '▼' }}</span>
              </th>
              <th class="p-4 cursor-pointer hover:text-indigo-600 transition" @click="handleSort('class')">
                CLASS / SUBJECT <span v-if="sortKey === 'class'" class="text-indigo-600 font-bold ml-1">{{ sortOrder === 'asc' ? '▲' : '▼' }}</span>
              </th>
              <th class="p-4 cursor-pointer hover:text-indigo-600 transition" @click="handleSort('period')">
                TIME SLOT <span v-if="sortKey === 'period'" class="text-indigo-600 font-bold ml-1">{{ sortOrder === 'asc' ? '▲' : '▼' }}</span>
              </th>
              <th class="p-4 cursor-pointer hover:text-indigo-600 transition" @click="handleSort('status')">
                TASK STATUS <span v-if="sortKey === 'status'" class="text-indigo-600 font-bold ml-1">{{ sortOrder === 'asc' ? '▲' : '▼' }}</span>
              </th>
              <th class="p-4 text-left cursor-pointer hover:text-indigo-600 transition" @click="handleSort('substitute')">
                SUBSTITUTE TEACHER <span v-if="sortKey === 'substitute'" class="text-indigo-600 font-bold ml-1">{{ sortOrder === 'asc' ? '▲' : '▼' }}</span>
              </th>
              <th class="py-4 pr-6 pl-4 text-right">ACTIONS</th>
            </tr>
          </thead>
          <tbody class="divide-y divide-slate-100 text-xs font-medium text-slate-800">
            <tr v-if="displayedRequests.length === 0">
              <td colspan="6" class="py-12 text-center text-slate-400 font-medium">NO TEACHER LEAVE RECORDS FOR DATE ({{ targetDate }}) </td>
            </tr>
            <tr v-for="req in displayedRequests" :key="req.id" class="hover:bg-slate-50/50 transition-colors group">
              <td class="py-4 pl-6 pr-4 font-bold text-slate-900">
                {{ teachersMap[req.teacher_id]?.name || 'LOADING...' }}
              </td>
              <td class="p-4">
                <div class="flex items-center gap-2">
                  <span class="text-slate-900 font-semibold">{{ req.class_name }}</span>
                  <span v-if="req.class_name.includes('/')" class="px-1.5 py-0.5 bg-violet-100 text-violet-700 rounded text-[10px] font-bold">COMBINED CLASS</span>
                  <span class="text-slate-300">·</span>
                  <span class="text-slate-500">{{ req.subject }}</span>
                </div>
              </td>
              <!-- 🌟 格式化为带有空格和上标的样式 -->
              <td class="p-4">
                <span class="inline-flex items-center px-2.5 py-1 rounded-lg bg-slate-100 text-slate-700 font-bold text-xs" v-html="formatOrdinalPeriodHtml(req.period)"></span>
              </td>
              <td class="p-4">
                <span :class="statusClass(req.status)" class="inline-flex items-center px-2.5 py-1 rounded-full text-xs font-semibold ring-1 ring-inset">
                  {{ req.status === 'pending' ? 'PENDING' : 'ASSIGNED' }}
                </span>
              </td>
              <td class="p-4 text-left">
                <span v-if="req.status === 'assigned'" class="text-xs font-bold text-indigo-700 bg-indigo-50 px-3 py-1 rounded-full border border-indigo-100 inline-block">
                  {{ getSubstituteDisplay(req.id) }}
                </span>
                <span v-else class="text-xs text-slate-400 font-medium">-</span>
              </td>
              <td class="py-4 pr-6 pl-4 text-right">
                <button 
                  v-if="req.status === 'pending'"
                  @click="openRecommendModal(req)"
                  class="inline-flex items-center justify-center px-4 py-2 text-xs font-bold text-indigo-700 bg-indigo-50 rounded-xl hover:bg-indigo-100 transition-all cursor-pointer whitespace-nowrap gap-1.5"
                >
                  <Sparkles class="w-3.5 h-3.5 text-indigo-600" />
                  Smart & Manual Scheduling
                </button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>

      <!-- Mode 2: Grouped by Date -->
      <div v-else class="p-6 space-y-4">
        <div v-if="Object.keys(groupedRequests).length === 0" class="py-12 text-center text-slate-400 font-medium">
          NO LEAVE RECORDS AVAILABLE
        </div>

        <div v-for="(requests, date) in groupedRequests" :key="date" class="border border-slate-200 rounded-2xl overflow-hidden shadow-sm">
          <button 
            @click="toggleDateFold(date)"
            class="w-full flex justify-between items-center px-6 py-4 bg-slate-50 hover:bg-slate-100 transition text-left cursor-pointer"
          >
            <div class="flex items-center gap-3">
              <span class="font-bold text-slate-900 text-sm flex items-center gap-2">
                <CalendarDays class="w-4 h-4 text-slate-500" />
                {{ date }}
              </span>
              <span class="text-xs px-2.5 py-0.5 rounded-full bg-indigo-100 text-indigo-700 font-bold">
                {{ requests.length }} TASK SLOTS
              </span>
            </div>
            <span class="text-slate-400 text-xs font-bold">
              {{ foldedDates[date] ? 'EXPAND ▼' : 'COLLAPSE ▲' }}
            </span>
          </button>

          <div v-show="!foldedDates[date]" class="bg-white">
            <table class="w-full text-left border-collapse whitespace-nowrap">
              <thead>
                <tr class="bg-slate-50/50 border-b border-slate-100 text-slate-500 text-xs uppercase tracking-widest font-semibold select-none">
                  <th class="py-3 pl-6 pr-4 w-48 cursor-pointer hover:text-indigo-600 transition" @click="handleSort('teacher')">
                    ABSENT TEACHER <span v-if="sortKey === 'teacher'" class="text-indigo-600 font-bold ml-1">{{ sortOrder === 'asc' ? '▲' : '▼' }}</span>
                  </th>
                  <th class="p-3 cursor-pointer hover:text-indigo-600 transition" @click="handleSort('class')">
                    CLASS / SUBJECT <span v-if="sortKey === 'class'" class="text-indigo-600 font-bold ml-1">{{ sortOrder === 'asc' ? '▲' : '▼' }}</span>
                  </th>
                  <th class="p-3 cursor-pointer hover:text-indigo-600 transition" @click="handleSort('period')">
                    TIME SLOT <span v-if="sortKey === 'period'" class="text-indigo-600 font-bold ml-1">{{ sortOrder === 'asc' ? '▲' : '▼' }}</span>
                  </th>
                  <th class="p-3 cursor-pointer hover:text-indigo-600 transition" @click="handleSort('status')">
                    STATUS <span v-if="sortKey === 'status'" class="text-indigo-600 font-bold ml-1">{{ sortOrder === 'asc' ? '▲' : '▼' }}</span>
                  </th>
                  <th class="p-3 text-left cursor-pointer hover:text-indigo-600 transition" @click="handleSort('substitute')">
                    SUBSTITUTE TEACHER <span v-if="sortKey === 'substitute'" class="text-indigo-600 font-bold ml-1">{{ sortOrder === 'asc' ? '▲' : '▼' }}</span>
                  </th>
                  <th class="py-3 pr-6 pl-3 text-right">ACTIONS</th>
                </tr>
              </thead>
              <tbody class="divide-y divide-slate-100 text-xs font-medium text-slate-800">
                <tr v-for="req in requests" :key="req.id" class="hover:bg-slate-50/50 transition-colors group">
                  <td class="py-3 pl-6 pr-4 text-slate-900 font-bold w-48">
                    {{ teachersMap[req.teacher_id]?.name || 'LOADING...' }}
                  </td>
                  <td class="p-3">
                    <div class="flex items-center gap-2">
                      <span class="text-slate-900 font-semibold">{{ req.class_name }}</span>
                      <span v-if="req.class_name.includes('/')" class="px-1.5 py-0.5 bg-violet-100 text-violet-700 rounded text-[10px] font-bold">COMBINED CLASS</span>
                      <span class="text-slate-300">·</span>
                      <span class="text-slate-500">{{ req.subject }}</span>
                    </div>
                  </td>
                  <!-- 🌟 格式化为带有空格和上标样式 -->
                  <td class="p-3">
                    <span class="inline-flex items-center px-2.5 py-1 rounded-lg bg-slate-100 text-slate-700 font-bold text-xs" v-html="formatOrdinalPeriodHtml(req.period)"></span>
                  </td>
                  <td class="p-3">
                    <span :class="statusClass(req.status)" class="inline-flex items-center px-2.5 py-1 rounded-full text-xs font-semibold ring-1 ring-inset">
                      {{ req.status === 'pending' ? 'PENDING' : 'ASSIGNED' }}
                    </span>
                  </td>
                  <td class="p-3 text-left">
                    <span v-if="req.status === 'assigned'" class="text-xs font-bold text-indigo-700 bg-indigo-50 px-3 py-1 rounded-full border border-indigo-100 inline-block">
                      {{ getSubstituteDisplay(req.id) }}
                    </span>
                    <span v-else class="text-xs text-slate-400 font-medium">-</span>
                  </td>
                  <td class="py-3 pr-6 pl-3 text-right">
                    <button 
                      v-if="req.status === 'pending'"
                      @click="openRecommendModal(req)"
                      class="inline-flex items-center justify-center px-4 py-2 text-xs font-bold text-indigo-700 bg-indigo-50 rounded-xl hover:bg-indigo-100 transition-all cursor-pointer whitespace-nowrap gap-1.5"
                    >
                      <Sparkles class="w-3.5 h-3.5 text-indigo-600" />
                      SCHEDULE
                    </button>
                  </td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>
      </div>

    </div>

    <!-- Modal -->
    <transition enter-active-class="transition duration-300 ease-out" enter-from-class="opacity-0 scale-95" enter-to-class="opacity-100 scale-100" leave-active-class="transition duration-200 ease-in" leave-from-class="opacity-100 scale-100" leave-to-class="opacity-0 scale-95">
      <div v-if="showModal" class="fixed inset-0 z-50 flex items-center justify-center p-4 sm:p-6">
        <div class="absolute inset-0 bg-slate-900/30 backdrop-blur-sm" @click="showModal = false"></div>
        <div class="relative bg-white rounded-3xl shadow-2xl w-full max-w-2xl overflow-hidden ring-1 ring-slate-900/10 max-h-[90vh] flex flex-col">
          
          <div class="px-8 py-6 border-b border-slate-100 flex justify-between items-center bg-white/50 backdrop-blur-md shrink-0">
            <div>
              <h2 class="text-xl font-bold text-slate-900">SUBSTITUTE ASSIGNMENT CENTER</h2>
              <p class="text-sm text-slate-500 mt-1">SUPPORTED BY SMART RECOMMENDATIONS, OR SELECT ANY SAME-SESSION TEACHER MANUALLY BELOW</p>
            </div>
            <button @click="showModal = false" class="text-slate-400 hover:text-slate-600 bg-slate-100 hover:bg-slate-200 rounded-full p-2 transition cursor-pointer">×</button>
          </div>
          
          <div class="p-8 bg-slate-50/50 space-y-6 overflow-y-auto">
            
            <div class="bg-white p-4 rounded-2xl border border-slate-200 shadow-sm flex items-center gap-3">
              <span class="text-xs font-bold text-slate-700 whitespace-nowrap">📍 LOCATION / REMARK:</span>
              <input 
                v-model="assignmentRemark" 
                type="text" 
                placeholder="E.G. LIBRARY (IF STUDENTS NEED TO BE BROUGHT TO THE LIBRARY OR CLASSES COMBINED)" 
                class="w-full px-3 py-2 bg-slate-50 border border-slate-200 rounded-xl text-xs font-medium text-slate-800 focus:outline-none focus:ring-2 focus:ring-indigo-500/50"
              />
            </div>

            <div class="bg-indigo-50/60 p-5 rounded-2xl border border-indigo-100 shadow-sm">
              <h3 class="text-xs font-bold uppercase tracking-wider text-indigo-900 mb-3 flex items-center gap-2">
                <span>🛠️ MANUAL ASSIGNMENT (WITHOUT SMART RECOMMENDATION)</span>
              </h3>
              <div class="flex flex-col sm:flex-row items-center gap-3">
                <select 
                  v-model="manualSelectedTeacherId" 
                  class="w-full px-3.5 py-2.5 bg-white border border-indigo-200 rounded-xl text-xs font-semibold text-slate-800 focus:outline-none focus:ring-2 focus:ring-indigo-500 cursor-pointer"
                >
                  <option value="" disabled>-- PLEASE SELECT A SAME-SESSION TEACHER MANUALLY --</option>
                  <option v-for="t in allSameSessionTeachers" :key="t.id" :value="t.id">
                    {{ t.name }} (SUBJECT: {{ t.subject || 'NONE' }})
                  </option>
                </select>
                <button 
                  @click="assignSubstitute(manualSelectedTeacherId)" 
                  :disabled="!manualSelectedTeacherId"
                  class="w-full sm:w-auto bg-indigo-600 hover:bg-indigo-700 disabled:opacity-50 text-white px-5 py-2.5 rounded-xl text-xs font-semibold shadow-sm transition-all shrink-0 cursor-pointer"
                >
                  CONFIRM MANUAL ASSIGNMENT
                </button>
              </div>
            </div>

            <hr class="border-slate-200" />

            <div>
              <h3 class="text-xs font-bold uppercase tracking-wider text-slate-500 mb-3 flex items-center gap-2">
                <Sparkles class="w-4 h-4 text-indigo-600" />
                SMART RECOMMENDATION CANDIDATES (TOP 6)
              </h3>
              
              <div v-if="loading" class="flex flex-col items-center justify-center py-6 space-y-3">
                <div class="w-6 h-6 border-4 border-indigo-500/30 border-t-indigo-600 rounded-full animate-spin"></div>
                <p class="text-xs text-slate-500 font-medium">SMART ALGORITHM IS CALCULATING...</p>
              </div>
              
              <div v-else-if="recommendations.length === 0" class="bg-white p-4 rounded-2xl border border-slate-200 text-xs text-slate-500 text-center">
                NO AUTOMATIC RECOMMENDATIONS AVAILABLE, PLEASE USE MANUAL ASSIGNMENT ABOVE.
              </div>

              <div v-else class="space-y-3">
                <div v-for="(teacher, index) in recommendations" :key="teacher.id" class="group flex flex-col sm:flex-row sm:justify-between sm:items-center p-4 bg-white border border-slate-200 rounded-2xl hover:border-indigo-300 hover:shadow-sm transition-all">
                  <div class="flex items-center gap-3 mb-3 sm:mb-0">
                    <div class="w-8 h-8 rounded-full bg-gradient-to-br from-indigo-100 to-violet-100 text-indigo-700 font-extrabold flex items-center justify-center text-xs">
                      #{{ index + 1 }}
                    </div>
                    <div>
                      <div class="font-bold text-slate-900 text-sm flex items-center gap-2">
                        {{ teacher.name }} 
                        <span class="text-[10px] font-medium px-2 py-0.5 rounded-md bg-slate-100 text-slate-600">{{ teacher.subject }}</span>
                      </div>
                      <div class="text-[11px] text-slate-500 mt-1 flex items-center gap-2 flex-wrap">
                        <span>ORIGINAL SCHEDULE: <span class="font-bold text-slate-700">{{ teacher.originalClasses }} CLASSES</span></span>
                        <span>·</span>
                        <span>TODAY'S SUBSTITUTIONS: <span class="font-bold text-orange-600">{{ teacher.todaySubCount }} CLASSES</span></span>
                        <span>·</span>
                        <span>THIS WEEK'S SUBSTITUTIONS: <span class="font-bold text-slate-700">{{ teacher.currentSubCount }}/{{ teacher.max_substitute_per_week }}</span></span>
                      </div>    
                    </div>
                  </div>
                  <button @click="assignSubstitute(teacher.id)" class="bg-slate-900 hover:bg-indigo-600 text-white px-4 py-2 rounded-xl text-xs font-semibold shadow-sm transition-all cursor-pointer">
                    SMART ASSIGN
                  </button>
                </div>
              </div>
            </div>

          </div>
        </div>
      </div>
    </transition>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import { supabase } from '../services/supabase'
import { recommendSubstitute } from '../utils/algorithm'
import { useToast } from '../utils/toast'
import { 
  LayoutDashboard, 
  Plus, 
  Clock3, 
  CircleCheck, 
  ChartColumnBig, 
  CalendarDays, 
  FolderOpen, 
  Sparkles 
} from 'lucide-vue-next'

const toast = useToast()
const leaveRequests = ref([])
const substituteAssignments = ref({})
const teachersMap = ref({})

const showModal = ref(false)
const loading = ref(false)
const recommendations = ref([])
const currentRequest = ref(null)
const assignmentRemark = ref('')

const viewMode = ref('today')
const targetDate = ref(new Date().toISOString().split('T')[0])
const foldedDates = ref({})

const sortKey = ref('period')
const sortOrder = ref('asc')

// 🌟 核心修改：在数字上标和 PERIOD 之间加入 &nbsp; 空格
const formatOrdinalPeriodHtml = (p) => {
  const num = Number(p)
  if (isNaN(num)) return `PERIOD ${p}`
  
  let suffix = 'th'
  if (num % 100 < 11 || num % 100 > 13) {
    switch (num % 10) {
      case 1: suffix = 'st'; break;
      case 2: suffix = 'nd'; break;
      case 3: suffix = 'rd'; break;
    }
  }
  return `${num}<sup>${suffix}</sup>&nbsp;PERIOD`
}

const handleSort = (key) => {
  if (sortKey.value === key) {
    sortOrder.value = sortOrder.value === 'asc' ? 'desc' : 'asc'
  } else {
    sortKey.value = key
    sortOrder.value = 'asc'
  }
}

const sortFunction = (a, b) => {
  let valA, valB

  if (sortKey.value === 'teacher') {
    valA = teachersMap.value[a.teacher_id]?.name || ''
    valB = teachersMap.value[b.teacher_id]?.name || ''
  } else if (sortKey.value === 'class') {
    valA = `${a.class_name} ${a.subject}`
    valB = `${b.class_name} ${b.subject}`
  } else if (sortKey.value === 'period') {
    valA = Number(a.period)
    valB = Number(b.period)
  } else if (sortKey.value === 'status') {
    valA = a.status
    valB = b.status
  } else if (sortKey.value === 'substitute') {
    valA = getSubstituteDisplay(a.id)
    valB = getSubstituteDisplay(b.id)
  }

  if (valA < valB) return sortOrder.value === 'asc' ? -1 : 1
  if (valA > valB) return sortOrder.value === 'asc' ? 1 : -1
  return 0
}

const allSameSessionTeachers = ref([])
const manualSelectedTeacherId = ref('')

const pendingCount = computed(() => leaveRequests.value.filter(r => r.status === 'pending').length)
const assignedCount = computed(() => leaveRequests.value.filter(r => r.status === 'assigned').length)

const displayedRequests = computed(() => {
  const filtered = leaveRequests.value.filter(r => r.leave_date === targetDate.value)
  return filtered.sort(sortFunction)
})

const groupedRequests = computed(() => {
  const groups = {}
  leaveRequests.value.forEach(req => {
    if (!groups[req.leave_date]) {
      groups[req.leave_date] = []
    }
    groups[req.leave_date].push(req)
  })

  Object.keys(groups).forEach(date => {
    groups[date].sort(sortFunction)
  })

  return groups
})

const toggleDateFold = (date) => {
  foldedDates.value[date] = !foldedDates.value[date]
}

const fetchRequests = async () => {
  const { data: tData } = await supabase.from('teachers').select('id, name')
  if (tData) {
    tData.forEach(t => {
      teachersMap.value[t.id] = t
    })
  }

  const { data: lData, error } = await supabase
    .from('leave_requests')
    .select('*')
    .order('leave_date', { ascending: false })
  
  if (!error) {
    leaveRequests.value = lData || []

    const leaveIds = leaveRequests.value.map(l => l.id)
    if (leaveIds.length > 0) {
      const { data: sData } = await supabase
        .from('substitute_assignments')
        .select('*')
        .in('leave_request_id', leaveIds)
      
      if (sData) {
        const map = {}
        sData.forEach(sub => {
          map[sub.leave_request_id] = sub
        })
        substituteAssignments.value = map
      }
    }
  }
}

const getSubstituteDisplay = (leaveRequestId) => {
  const sub = substituteAssignments.value[leaveRequestId]
  if (!sub || !sub.sub_teacher_id) return ''
  
  const subTeacher = teachersMap.value[sub.sub_teacher_id]
  const name = subTeacher ? subTeacher.name : 'UNKNOWN'

  if (sub.remark) {
    return `${name} (${sub.remark})`
  }
  return name
}

const statusClass = (status) => {
  if (status === 'pending') return 'bg-amber-50 text-amber-700 ring-amber-600/20'
  return 'bg-emerald-50 text-emerald-700 ring-emerald-600/20'
}

const openRecommendModal = async (req) => {
  currentRequest.value = req
  assignmentRemark.value = ''
  manualSelectedTeacherId.value = ''
  showModal.value = true
  loading.value = true

  try {
    recommendations.value = await recommendSubstitute(req)

    const { data: absentTeacher } = await supabase
      .from('teachers')
      .select('session')
      .eq('id', req.teacher_id)
      .single()
    
    const session = absentTeacher?.session || 'morning'

    const { data: teachersData } = await supabase
      .from('teachers')
      .select('*')
      .eq('is_active', true)
      .eq('session', session)
      .neq('id', req.teacher_id)

    allSameSessionTeachers.value = teachersData || []
  } catch (error) {
    toast.error("FAILED TO LOAD SCHEDULE DATA: " + error.message)
  } finally {
    loading.value = false
  }
}

const assignSubstitute = async (teacherId) => {
  if (!teacherId) return
  try {
    const { error: insertErr } = await supabase.from('substitute_assignments').insert({
      leave_request_id: currentRequest.value.id,
      sub_teacher_id: teacherId,
      remark: assignmentRemark.value ? assignmentRemark.value.trim().toUpperCase() : null
    })
    if (insertErr) throw insertErr

    const { error: updateErr } = await supabase.from('leave_requests')
      .update({ status: 'assigned' })
      .eq('id', currentRequest.value.id)
    if (updateErr) throw updateErr

    toast.success("SUBSTITUTE ASSIGNED SUCCESSFULLY!")
    showModal.value = false
    fetchRequests()
  } catch (error) {
    toast.error("ASSIGNMENT FAILED: " + error.message)
  }
}

onMounted(() => {
  fetchRequests()
})
</script>