<template>
  <div class="p-4 sm:p-8 mx-auto min-h-screen space-y-8 min-w-max w-full print:p-0 print:w-auto print:block print:m-0">
    
    <!-- Top Header: Unified card style and gradient title -->
    <div class="no-print bg-white rounded-3xl p-6 sm:p-8 shadow-sm ring-1 ring-slate-900/5 space-y-6">
      
      <!-- Part 1: Main Title & Subtitle -->
      <div class="space-y-2 max-w-4xl">
        <h1 class="text-2xl sm:text-3xl font-extrabold tracking-tight text-transparent bg-clip-text bg-gradient-to-r from-slate-900 via-indigo-800 to-violet-800 flex items-center gap-3">
          <GraduationCap class="w-8 h-8 text-indigo-700 shrink-0" />
          SUBJECT TARGET & ACHIEVEMENT RATE ANALYSIS CENTER
        </h1>
        <p class="text-slate-500 text-xs sm:text-sm font-medium leading-relaxed">
          TRACK TEACHING TARGETS, MMI INTERRUPTIONS, AND ACHIEVEMENTS FROM MULTIPLE DIMENSIONS, SUPPORTING DATA REPORTS, VISUAL DASHBOARDS & PDF EXPORTS.
        </p>
      </div>

      <!-- Part 2: Bottom Row Alignment (Left Tab Switcher, Right Action Buttons) -->
      <div class="flex flex-col xl:flex-row justify-between items-start xl:items-center gap-4 pt-4 border-t border-slate-100">
        
        <!-- Left: View Switcher Tabs -->
        <div class="flex flex-wrap gap-3">
          <button 
            @click="activeTab = 'table'" 
            class="px-5 py-2.5 rounded-2xl text-xs font-bold transition-all flex items-center gap-2 cursor-pointer whitespace-nowrap"
            :class="activeTab === 'table' ? 'bg-slate-900 text-white shadow-md' : 'bg-slate-50 text-slate-600 hover:bg-slate-100'"
          >
            <BarChart3 class="w-4 h-4" /> DETAILED DATA REPORT
          </button>
          <button 
            @click="activeTab = 'chart'" 
            class="px-5 py-2.5 rounded-2xl text-xs font-bold transition-all flex items-center gap-2 cursor-pointer whitespace-nowrap"
            :class="activeTab === 'chart' ? 'bg-slate-900 text-white shadow-md' : 'bg-slate-50 text-slate-600 hover:bg-slate-100'"
          >
            <PieChart class="w-4 h-4" /> VISUAL CHART DASHBOARD
          </button>
        </div>

        <!-- Right: Action Buttons -->
        <div class="flex flex-wrap items-center gap-3">
          <button @click="exportPdfReport" class="no-print bg-emerald-600 hover:bg-emerald-700 text-white px-4 h-11 rounded-2xl text-xs font-bold transition shadow-sm flex items-center gap-2 cursor-pointer whitespace-nowrap">
            <Printer class="w-4 h-4" /> PRINT / PDF
          </button>
          <button @click="showManageModal = true" class="no-print bg-slate-900 hover:bg-slate-800 text-white px-4 h-11 rounded-2xl text-xs font-bold transition shadow-sm cursor-pointer whitespace-nowrap flex items-center gap-2">
            <Settings class="w-4 h-4" /> MANAGE TARGETS
          </button>
          <button @click="loadAnalyticsData" :disabled="loading" class="no-print bg-indigo-600 hover:bg-indigo-700 text-white px-5 h-11 rounded-2xl text-xs font-bold transition shadow-sm cursor-pointer flex items-center gap-2 whitespace-nowrap">
            <span v-if="loading" class="w-3.5 h-3.5 border-2 border-white/30 border-t-white rounded-full animate-spin"></span>
            <RotateCw v-else class="w-4 h-4" />
            <span>{{ loading ? 'CALCULATING...' : 'REFRESH' }}</span>
          </button>
        </div>

      </div>

    </div>

    <!-- 🔍 Multi-Dimensional Advanced Filter Panel -->
    <div class="no-print bg-white p-6 rounded-3xl shadow-sm ring-1 ring-slate-900/5 grid grid-cols-4 gap-4">
      <div class="relative w-full">
        <label class="block text-xs font-bold text-slate-500 mb-2 uppercase tracking-wider">FILTER BY GRADE</label>
        <select v-model="filterGrade" @change="onGradeChange" class="w-full bg-slate-50 border border-slate-200 px-4 h-11 rounded-2xl text-xs font-bold text-slate-700 focus:outline-none focus:ring-2 focus:ring-indigo-500 appearance-none pr-8 cursor-pointer truncate">
          <option value="all">ALL SCHOOL GRADES</option>
          <option v-for="g in [1,2,3,4,5,6]" :key="g" :value="g">GRADE {{ g }}</option>
        </select>
        <div class="absolute right-3 top-[42px] pointer-events-none text-slate-500">
          <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 9l-7 7-7-7"></path></svg>
        </div>
      </div>

      <div class="relative w-full">
        <label class="block text-xs font-bold text-slate-500 mb-2 uppercase tracking-wider">FILTER BY CLASS</label>
        <select v-model="filterClass" class="w-full bg-slate-50 border border-slate-200 px-4 h-11 rounded-2xl text-xs font-bold text-slate-700 focus:outline-none focus:ring-2 focus:ring-indigo-500 appearance-none pr-8 cursor-pointer truncate">
          <option value="all">ALL CLASSES FOR THIS GRADE</option>
          <option v-for="c in availableClasses" :key="c.id" :value="c.class_name">{{ c.class_name }}</option>
        </select>
        <div class="absolute right-3 top-[42px] pointer-events-none text-slate-500">
          <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 9l-7 7-7-7"></path></svg>
        </div>
      </div>

      <div class="relative w-full">
        <label class="block text-xs font-bold text-slate-500 mb-2 uppercase tracking-wider">FILTER BY SUBJECT</label>
        <select v-model="filterSubject" class="w-full bg-slate-50 border border-slate-200 px-4 h-11 rounded-2xl text-xs font-bold text-slate-700 focus:outline-none focus:ring-2 focus:ring-indigo-500 appearance-none pr-8 cursor-pointer truncate">
          <option value="all">ALL SUBJECTS</option>
          <option v-for="s in uniqueSubjects" :key="s" :value="s">{{ s }}</option>
        </select>
        <div class="absolute right-3 top-[42px] pointer-events-none text-slate-500">
          <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 9l-7 7-7-7"></path></svg>
        </div>
      </div>

      <div class="relative w-full">
        <label class="block text-xs font-bold text-indigo-600 mb-2 uppercase tracking-wider flex items-center gap-1.5">
          <Users class="w-4 h-4 text-indigo-600" /> FILTER BY SUBJECT TEACHER
        </label>
        <select v-model="filterTeacher" class="w-full bg-indigo-50/50 border border-indigo-200 px-4 h-11 rounded-2xl text-xs font-bold text-slate-800 focus:outline-none focus:ring-2 focus:ring-indigo-500 appearance-none pr-8 cursor-pointer truncate">
          <option value="all">ALL SCHOOL TEACHERS</option>
          <option v-for="tch in allTeachers" :key="tch.id" :value="tch.name">{{ tch.name }}</option>
        </select>
        <div class="absolute right-3 top-[42px] pointer-events-none text-slate-500">
          <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 9l-7 7-7-7"></path></svg>
        </div>
      </div>
    </div>

    <!-- 🎯 Core Statistical Metrics -->
    <div class="grid grid-cols-3 gap-6 print-summary">
      <div class="bg-white p-6 rounded-3xl shadow-sm ring-1 ring-slate-900/5">
        <div class="text-xs font-bold text-slate-500 uppercase tracking-wider">TOTAL FILTERED RESULTS</div>
        <div class="text-3xl font-black text-slate-900 mt-2">{{ analysisSummary.total }}</div>
      </div>
      <div class="bg-white p-6 rounded-3xl shadow-sm ring-1 ring-slate-900/5">
        <div class="text-xs font-bold text-emerald-600 uppercase tracking-wider">TARGET MET (PASSED)</div>
        <div class="text-3xl font-black text-emerald-700 mt-2">{{ analysisSummary.met }}</div>
      </div>
      <div class="bg-white p-6 rounded-3xl shadow-sm ring-1 ring-slate-900/5 flex items-center justify-between">
        <div>
          <div class="text-xs font-bold text-red-600 uppercase tracking-wider">NOT MET (WARNING)</div>
          <div class="text-3xl font-black text-red-700 mt-2">{{ analysisSummary.unmet }}</div>
        </div>
        <AlertTriangle class="w-8 h-8 text-red-500/80 shrink-0" />
      </div>
    </div>

    <!-- ================= TAB 1: Detailed Data Report View ================= -->
    <div v-if="activeTab === 'table'" class="bg-white rounded-3xl shadow-sm ring-1 ring-slate-900/5 overflow-hidden">
      <div class="p-6 border-b border-slate-100 flex justify-between items-center">
        <h3 class="text-xs font-bold uppercase tracking-wider text-slate-500">DETAILED ANALYSIS REPORT (CLASS, SUBJECT & TEACHER COMPARISON)</h3>
        <span class="text-[11px] text-slate-500 font-bold">EFFECTIVE SCHOOL CALENDAR RATIO: {{ (progressRatio * 100).toFixed(1) }}%</span>
      </div>

      <table class="w-full text-left border-collapse whitespace-nowrap print-table">
        <thead>
          <tr class="bg-slate-50 text-[11px] font-bold text-slate-700 uppercase tracking-wider border-b border-slate-200 select-none">
            <th @click="toggleSort('class_name')" class="py-4 pl-8 pr-4 cursor-pointer hover:bg-slate-100 transition select-none">
              GRADE / CLASS <span class="text-indigo-600">{{ getSortIcon('class_name') }}</span>
            </th>
            <th @click="toggleSort('subject_name')" class="p-4 cursor-pointer hover:bg-slate-100 transition select-none">
              SUBJECT NAME <span class="text-indigo-600">{{ getSortIcon('subject_name') }}</span>
            </th>
            <th @click="toggleSort('teacher_name')" class="p-4 cursor-pointer hover:bg-slate-100 transition select-none">
              SUBJECT TEACHER <span class="text-indigo-600">{{ getSortIcon('teacher_name') }}</span>
            </th>
            <th @click="toggleSort('target')" class="p-4 text-center cursor-pointer hover:bg-slate-100 transition select-none">
              ANNUAL PLAN TARGET <span class="text-indigo-600">{{ getSortIcon('target') }}</span>
            </th>
            <th @click="toggleSort('expected')" class="p-4 text-center cursor-pointer hover:bg-slate-100 transition select-none">
              THEORETICAL ACTUAL PROGRESS <span class="text-indigo-600">{{ getSortIcon('expected') }}</span>
            </th>
            <th @click="toggleSort('lostCount')" class="p-4 text-center cursor-pointer hover:bg-slate-100 transition select-none">
              TOTAL PDPC PERIODS AFFECTED <span class="text-indigo-600">{{ getSortIcon('lostCount') }}</span>
            </th>
            <th @click="toggleSort('actual')" class="p-4 text-center cursor-pointer hover:bg-slate-100 transition select-none">
              ACTUAL EXECUTION <span class="text-indigo-600">{{ getSortIcon('actual') }}</span>
            </th>
            <th @click="toggleSort('gap')" class="p-4 text-center cursor-pointer hover:bg-slate-100 transition select-none">
              GAP <span class="text-indigo-600">{{ getSortIcon('gap') }}</span>
            </th>
            <th @click="toggleSort('status')" class="py-4 pr-8 pl-4 text-center cursor-pointer hover:bg-slate-100 transition select-none">
              STATUS <span class="text-indigo-600">{{ getSortIcon('status') }}</span>
            </th>
          </tr>
        </thead>
        <tbody class="divide-y divide-slate-100 text-xs font-medium text-slate-800">
          <tr v-if="filteredAnalysisList.length === 0">
            <td colspan="9" class="py-12 text-center text-slate-400 font-medium">
              NO ANALYSIS RECORDS FOUND, PLEASE ADJUST FILTER CONDITIONS.
            </td>
          </tr>
          <tr v-for="(item, idx) in filteredAnalysisList" :key="idx" class="hover:bg-slate-50/50 transition">
            <td class="py-4 pl-8 pr-4 font-bold text-slate-900">
              GRADE {{ item.grade }} - {{ item.class_name }}
            </td>
            <td class="p-4 font-bold text-slate-900">{{ item.subject_name }}</td>
            <td class="p-4 font-bold text-indigo-900">{{ item.teacher_name || 'NOT ASSIGNED' }}</td>
            <td class="p-4 text-center font-bold">{{ item.target }} SLOTS</td>
            <td class="p-4 text-center font-bold text-indigo-700">{{ item.expected }} SLOTS</td>
            <td class="p-4 text-center text-amber-700 font-bold">-{{ item.lostCount }} SLOTS</td>
            <td class="p-4 text-center font-bold text-slate-900">{{ item.actual }} SLOTS</td>
            <td class="p-4 text-center font-bold" :class="item.gap >= 0 ? 'text-emerald-700' : 'text-red-700'">
              {{ item.gap >= 0 ? '+' + item.gap : item.gap }} SLOTS
            </td>
            <td class="py-4 pr-8 pl-4 text-center font-bold">
              <span 
                class="px-2.5 py-1 rounded-lg text-xs inline-block"
                :class="{
                  'bg-red-50 text-red-700': item.status === 'NOT MET',
                  'bg-emerald-50 text-emerald-700': item.status === 'EXCEEDED TARGET',
                  'bg-slate-100 text-slate-700': item.status === 'TARGET MET'
                }"
              >
                {{ item.status }}
              </span>
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <!-- ================= TAB 2: 📈 Visual Chart Analysis Dashboard ================= -->
    <div v-if="activeTab === 'chart'" class="space-y-6">
      <div class="grid grid-cols-2 gap-6">
        <div class="bg-white p-6 rounded-3xl shadow-sm ring-1 ring-slate-900/5 space-y-4">
          <div class="flex justify-between items-center">
            <h3 class="text-sm font-bold text-slate-800 flex items-center gap-2">
              <Target class="w-4 h-4 text-indigo-600" /> OVERALL SUBJECT ACHIEVEMENT HEALTH RATE
            </h3>
            <span class="text-xs text-indigo-600 font-bold bg-indigo-50 px-2.5 py-1 rounded-lg">REAL-TIME CALCULATION</span>
          </div>
          
          <div class="py-4 flex flex-col items-center justify-center space-y-4">
            <div class="relative w-36 h-36 rounded-full flex items-center justify-center border-8 border-slate-100 shadow-inner"
                 :style="{ borderColor: completionRate >= 80 ? '#10b981' : '#f59e0b' }">
              <div class="text-center">
                <span class="text-3xl font-black text-slate-900">{{ completionRate }}%</span>
                <div class="text-[10px] text-slate-400 font-bold uppercase mt-1">ACHIEVEMENT RATE</div>
              </div>
            </div>
            <p class="text-xs text-slate-500 font-medium">UNDER CURRENT CONDITIONS, <strong class="text-slate-800">{{ analysisSummary.met }}</strong> SUBJECTS HAVE MET THE TARGET, <strong class="text-red-600">{{ analysisSummary.unmet }}</strong> HAVE NOT.</p>
          </div>
        </div>

        <div class="bg-white p-6 rounded-3xl shadow-sm ring-1 ring-slate-900/5 space-y-4">
          <div class="flex justify-between items-center">
            <h3 class="text-sm font-bold text-slate-800 flex items-center gap-2">
              <BarChart3 class="w-4 h-4 text-indigo-600" /> ACHIEVEMENT DISTRIBUTION BY GRADE
            </h3>
            <span class="text-xs text-slate-400 font-semibold">MET VS NOT MET</span>
          </div>

          <div class="space-y-3 pt-2">
            <div v-for="g in [1,2,3,4,5,6]" :key="g" class="space-y-1">
              <div class="flex justify-between text-xs font-bold">
                <span class="text-slate-700">GRADE {{ g }}</span>
                <span class="text-slate-500">MET: {{ getGradeStats(g).met }} / NOT MET: {{ getGradeStats(g).unmet }}</span>
              </div>
              <div class="w-full h-3 bg-slate-100 rounded-full overflow-hidden flex">
                <div class="bg-emerald-500 h-full transition-all duration-500" :style="{ width: getGradeStats(g).total > 0 ? (getGradeStats(g).met / getGradeStats(g).total) * 100 + '%' : '0%' }"></div>
                <div class="bg-red-400 h-full transition-all duration-500" :style="{ width: getGradeStats(g).total > 0 ? (getGradeStats(g).unmet / getGradeStats(g).total) * 100 + '%' : '0%' }"></div>
              </div>
            </div>
          </div>
        </div>
      </div>

      <div class="bg-white p-6 rounded-3xl shadow-sm ring-1 ring-slate-900/5 space-y-6">
        <div class="flex justify-between items-center">
          <div>
            <h3 class="text-sm font-bold text-slate-800 flex items-center gap-2">
              <AlertTriangle class="w-4 h-4 text-amber-500" /> RANKING OF SUBJECTS WITH MOST SLOT LOSSES DUE TO MMI INTERRUPTIONS
            </h3>
            <p class="text-xs text-slate-400 mt-0.5 font-medium">SHOWING TEACHING SLOT LOSSES BY SUBJECT DUE TO LEAVE OR OFFICIAL BUSINESS</p>
          </div>
          <span class="text-xs bg-amber-50 text-amber-700 px-3 py-1 rounded-full font-bold">INTERRUPTION WARNING</span>
        </div>

        <div class="space-y-4">
          <div v-if="subjectLossRanking.length === 0" class="text-xs text-slate-400 text-center py-6">
            NO INTERRUPTION LOSS RECORDS FOUND UNDER CURRENT CONDITIONS.
          </div>
          <div v-for="(item, index) in subjectLossRanking.slice(0, 5)" :key="index" class="space-y-1.5">
            <div class="flex justify-between items-center text-xs font-bold">
              <span class="text-slate-800 flex items-center gap-2">
                <span class="w-5 h-5 rounded-full bg-slate-100 text-slate-600 flex items-center justify-center text-[10px] font-black">{{ index + 1 }}</span>
                {{ item.subject }}
              </span>
              <span class="text-amber-600 font-black">-{{ item.lost }} TOTAL SLOT LOSSES</span>
            </div>
            <div class="w-full h-2.5 bg-slate-100 rounded-full overflow-hidden">
              <div class="bg-amber-500 h-full rounded-full transition-all duration-500" 
                   :style="{ width: Math.min(100, (item.lost / maxSubjectLoss) * 100) + '%' }"></div>
            </div>
          </div>
        </div>
      </div>

    </div>

    <!-- Modal: Manage Targets -->
    <div v-if="showManageModal" class="fixed inset-0 z-50 flex items-center justify-center bg-slate-900/50 backdrop-blur-sm p-4">
      <div class="bg-white rounded-3xl max-w-3xl w-full p-6 shadow-2xl space-y-6 max-h-[90vh] overflow-y-auto">
        <div class="flex justify-between items-center border-b border-slate-100 pb-4">
          <h3 class="text-base font-bold text-slate-800 flex items-center gap-2">
            <Settings class="w-4 h-4 text-indigo-600" /> ONLINE SUBJECT TARGET MAINTENANCE
          </h3>
          <button @click="showManageModal = false" class="text-slate-400 hover:text-slate-700 font-bold text-sm cursor-pointer flex items-center gap-1">
            <X class="w-4 h-4" /> CLOSE
          </button>
        </div>

        <div class="bg-slate-50 p-4 rounded-2xl border border-slate-200 grid grid-cols-4 gap-3 items-end">
          <div class="relative w-full">
            <label class="block text-[11px] font-bold text-slate-600 mb-1">GRADE</label>
            <select v-model="newTarget.grade" class="w-full bg-white border border-slate-200 px-3 py-2 rounded-xl text-xs font-semibold appearance-none pr-8 cursor-pointer">
              <option v-for="g in [1,2,3,4,5,6]" :key="g" :value="g">GRADE {{ g }}</option>
            </select>
            <div class="absolute right-3 top-[28px] pointer-events-none text-slate-500">
              <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 9l-7 7-7-7"></path></svg>
            </div>
          </div>
          <div>
            <label class="block text-[11px] font-bold text-slate-600 mb-1">SUBJECT NAME</label>
            <input type="text" v-model="newTarget.subject_name" placeholder="E.G. MATHEMATICS" class="w-full bg-white border border-slate-200 px-3 py-2 rounded-xl text-xs font-semibold" />
          </div>
          <div>
            <label class="block text-[11px] font-bold text-slate-600 mb-1">TOTAL PLANNED SLOTS</label>
            <input type="number" v-model.number="newTarget.planned_periods" placeholder="E.G. 215" class="w-full bg-white border border-slate-200 px-3 py-2 rounded-xl text-xs font-semibold" />
          </div>
          <button @click="addTarget" class="bg-indigo-600 hover:bg-indigo-700 text-white py-2 rounded-xl text-xs font-bold shadow-sm transition cursor-pointer flex items-center justify-center gap-1.5">
            <Plus class="w-3.5 h-3.5" /> ADD NEW TARGET
          </button>
        </div>

        <div class="space-y-3">
          <h4 class="text-xs font-bold text-slate-500 uppercase tracking-wider">LIST OF ESTABLISHED SUBJECT TARGETS</h4>
          <div class="divide-y divide-slate-100 border border-slate-200 rounded-2xl overflow-hidden max-h-60 overflow-y-auto">
            <div v-for="t in allTargets" :key="t.id" class="p-3 flex items-center justify-between bg-white hover:bg-slate-50 transition">
              <div class="flex items-center gap-4 text-xs font-semibold text-slate-700">
                <span class="px-2.5 py-1 bg-indigo-50 text-indigo-700 rounded-lg font-bold">GRADE {{ t.grade }}</span>
                <span class="text-slate-900 font-bold">{{ t.subject_name }}</span>
                <span class="text-slate-500">TARGET: <strong>{{ t.planned_periods }}</strong> SLOTS</span>
              </div>
              <button @click="deleteTarget(t.id)" class="text-slate-400 hover:text-red-600 text-xs font-bold px-2 py-1 transition cursor-pointer flex items-center gap-1">
                <Trash2 class="w-3.5 h-3.5" /> DELETE
              </button>
            </div>
          </div>
        </div>

        <div class="flex justify-end pt-2">
          <button @click="loadAnalyticsData(); showManageModal = false" class="bg-slate-900 hover:bg-slate-800 text-white px-5 py-2 rounded-xl text-xs font-bold shadow-sm cursor-pointer">
            DONE & REFRESH REPORT
          </button>
        </div>
      </div>
    </div>

  </div>
</template>

<script setup>
import { ref, computed, onMounted, onActivated } from 'vue'
import { supabase } from '../services/supabase'
import { useToast } from '../utils/toast'
import { 
  GraduationCap, 
  BarChart3, 
  PieChart, 
  Printer, 
  Settings, 
  RotateCw, 
  Users, 
  AlertTriangle, 
  Target, 
  X, 
  Plus, 
  Trash2 
} from 'lucide-vue-next'

const toast = useToast()
const loading = ref(false)
const activeTab = ref('table')

const filterGrade = ref('all')
const filterClass = ref('all')
const filterSubject = ref('all')
const filterTeacher = ref('all')

const sortKey = ref('grade')
const sortOrder = ref('asc')

const analysisList = ref([])
const allClasses = ref([])
const allTargets = ref([])
const allSchoolWeeks = ref([])
const allTeachers = ref([])

const showManageModal = ref(false)
const newTarget = ref({ grade: 1, subject_name: '', planned_periods: 215, kpm_min_hours: 180 })

const progressRatio = computed(() => {
  if (!allSchoolWeeks.value || allSchoolWeeks.value.length === 0) return 0.5
  const totalSchoolWeeks = allSchoolWeeks.value.filter(w => w.is_school_week).length
  if (totalSchoolWeeks === 0) return 0.5
  const today = new Date().toISOString().split('T')[0]
  const passedSchoolWeeks = allSchoolWeeks.value.filter(w => {
    return w.is_school_week && w.end_date <= today
  }).length
  const ratio = passedSchoolWeeks / totalSchoolWeeks
  return Math.min(1, Math.max(0, ratio))
})

const cleanString = (str) => {
  if (!str) return ''
  return String(str).trim().toUpperCase().replace(/[^A-Z0-9\u4e00-\u9fa5]/g, '')
}

const standardizeSubjectName = (name) => {
  const clean = cleanString(name)
  if (!clean) return ''
  
  if (['BI', 'ENGLISH', 'BAHASAINGGERIS', 'ENG', '英文'].includes(clean)) return 'BAHASA INGGERIS'
  if (['BM', 'MELAYU', 'BAHASAMELAYU', 'MALAY', '国文', '马来文'].includes(clean)) return 'BAHASA MELAYU'
  if (['BC', 'CINA', 'BAHASACINA', 'CHINESE', '华文', '华语'].includes(clean)) return 'BAHASA CINA'
  
  if (['MATEMATIK', 'MATH', 'MT', 'MM', '数学'].includes(clean)) return 'MATEMATIK'
  if (['SN', 'SAINS', 'SCIENCE', 'SC', '科学'].includes(clean)) return 'SAINS'
  
  if (['PJ', 'PENDIDIKANJASMANI', 'JASMANI', 'PE', '体育'].includes(clean)) return 'PENDIDIKAN JASMANI'
  if (['PM', 'PENDIDIKANMORAL', 'MORAL', '道德'].includes(clean)) return 'PENDIDIKAN MORAL'
  if (['PI', 'PENDIDIKANISLAM', 'ISLAM', '宗教'].includes(clean)) return 'PENDIDIKAN ISLAM'
  if (['PSV', 'PENDIDIKANSENIVISUAL', 'SENI', 'VISUAL', 'ART', '美术'].includes(clean)) return 'PENDIDIKAN SENI VISUAL'
  if (['MZ', 'PMUZIK', 'PENDIDIKANMUZIK', 'MUZIK', 'MUSIC', '音乐'].includes(clean)) return 'PENDIDIKAN MUZIK'
  if (['PK', 'PENDIDIKANKESIHATAN', 'KESIHATAN', 'HEALTH', '健教', '健康教育'].includes(clean)) return 'PENDIDIKAN KESIHATAN'
  if (['SEJARAH', 'SEJ', 'HIST', '历史'].includes(clean)) return 'SEJARAH'
  if (['RBT', 'REKABENTUKDANTEKNOLOGI', 'REKABENTUK', '设计与工艺'].includes(clean)) return 'REKA BENTUK DAN TEKNOLOGI'
  
  return clean
}

const isSubjectMatch = (subjA, subjB) => {
  if (!subjA || !subjB) return false
  const stdA = standardizeSubjectName(subjA)
  const stdB = standardizeSubjectName(subjB)
  if (stdA && stdB && stdA === stdB) return true
  
  const cA = cleanString(subjA)
  const cB = cleanString(subjB)
  return cA === cB || cA.includes(cB) || cB.includes(cA)
}

const toggleSort = (key) => {
  if (sortKey.value === key) {
    sortOrder.value = sortOrder.value === 'asc' ? 'desc' : 'asc'
  } else {
    sortKey.value = key
    sortOrder.value = 'asc'
  }
}

const getSortIcon = (key) => {
  if (sortKey.value !== key) return '↕'
  return sortOrder.value === 'asc' ? '▲' : '▼'
}

const onGradeChange = () => {
  filterClass.value = 'all'
}

const availableClasses = computed(() => {
  if (filterGrade.value === 'all') return allClasses.value
  return allClasses.value.filter(c => Number(c.grade) === Number(filterGrade.value))
})

const uniqueSubjects = computed(() => {
  const subs = new Set(allTargets.value.map(t => standardizeSubjectName(t.subject_name)))
  return Array.from(subs)
})

const fetchAllRows = async (tableName) => {
  let allData = []
  let from = 0
  const limit = 1000 
  
  while (true) {
    const { data, error } = await supabase
      .from(tableName)
      .select('*')
      .range(from, from + limit - 1)
      
    if (error) throw error
    if (data) allData.push(...data)
    
    if (!data || data.length < limit) break
    from += limit
  }
  return allData
}

const loadAnalyticsData = async () => {
  loading.value = true
  try {
    const { data: targets } = await supabase.from('subject_targets').select('*').order('grade', { ascending: true })
    allTargets.value = targets || []

    const { data: classes } = await supabase.from('classes').select('*')
    allClasses.value = classes || []

    const { data: teachers } = await supabase.from('teachers').select('*')
    allTeachers.value = teachers || []

    const { data: weeks } = await supabase.from('school_weeks').select('*').order('week_number', { ascending: true })
    allSchoolWeeks.value = weeks || []

    const timetables = await fetchAllRows('timetable')
    const interruptions = await fetchAllRows('mmi_interruptions')
    const leaveRequests = await fetchAllRows('leave_requests')

    const teacherMap = {}
    if (teachers) {
      teachers.forEach(tch => {
        if (tch.id) teacherMap[String(tch.id)] = tch
        if (tch.name) teacherMap[cleanString(tch.name)] = tch
      })
    }

    const enrichedTimetables = (timetables || []).map(item => {
      const tIdKey = item.teacher_id ? String(item.teacher_id) : ''
      const tNameKey = item.teacher_name ? cleanString(item.teacher_name) : ''
      const teacherObj = teacherMap[tIdKey] || teacherMap[tNameKey] || {}
      return {
        ...item,
        teacher_info: teacherObj,
        resolved_teacher_name: teacherObj.name || item.teacher_name || item.teacher || ''
      }
    })

    const today = new Date().toISOString().split('T')[0]
    const currentWeek = (weeks || []).filter(w => w.is_school_week && w.end_date <= today).length || 27
    const validWeeksCount = (weeks || []).filter(w => w.is_school_week).length
    const totalSchoolWeeks = validWeeksCount > 0 ? validWeeksCount : 42
    let results = []
    
    for (const cls of (classes || [])) {
      const gradeTargets = (targets || []).filter(t => Number(t.grade) === Number(cls.grade))

      for (const t of gradeTargets) {
        const totalTarget = t.planned_periods || 215 
        const standardizedTargetSubject = standardizeSubjectName(t.subject_name)

        const matchedEntries = enrichedTimetables.filter(item => {
          const itemClass = cleanString(item.class_name)
          const clsName = cleanString(cls.class_name)
          const isClassMatched = itemClass === clsName || itemClass.includes(clsName) || clsName.includes(itemClass)
          if (!isClassMatched) return false

          const rawSubj = item.subject || item.subject_name || item.teacher_info?.subject_name || ''
          return isSubjectMatch(rawSubj, standardizedTargetSubject)
        })

        const assignedTeacherNames = [...new Set(matchedEntries.map(e => e.resolved_teacher_name).filter(Boolean))]
        const assignedTeacherIds = [...new Set(matchedEntries.map(e => e.teacher_id || e.teacher_info?.id).filter(Boolean))]
        const assignedTeacherName = assignedTeacherNames.length > 0 ? assignedTeacherNames.join(' / ') : 'NOT ASSIGNED'

        const uniquePeriods = new Set(matchedEntries.map(e => `${e.weekday}-${e.period}`))
        const weeklyPeriods = uniquePeriods.size

        const lostSlotSet = new Set()

        if (leaveRequests && leaveRequests.length > 0) {
          leaveRequests.forEach(req => {
            const reqClass = cleanString(req.class_name)
            const clsName = cleanString(cls.class_name)
            const isClassMatched = reqClass === clsName || reqClass.includes(clsName) || clsName.includes(reqClass)
            const isSubjMatched = isSubjectMatch(req.subject, standardizedTargetSubject)

            const reqTeacherNameClean = cleanString(req.teacher_name)
            const isTeacherMatched = 
              assignedTeacherIds.some(id => req.teacher_id && String(req.teacher_id) === String(id)) ||
              assignedTeacherNames.some(name => reqTeacherNameClean && cleanString(name) === reqTeacherNameClean)

            if (isTeacherMatched && isClassMatched && isSubjMatched) {
              if (req.leave_date) {
                const leaveDateObj = new Date(req.leave_date)
                const leaveWeekday = leaveDateObj.getDay()
                
                enrichedTimetables.forEach(item => {
                  const itemClass = cleanString(item.class_name)
                  const itemSubj = item.subject || item.subject_name || item.teacher_info?.subject_name || ''
                  const itemWeekday = Number(item.weekday)
                  
                  const matchCls = itemClass === clsName || itemClass.includes(clsName) || clsName.includes(itemClass)
                  const matchSubj = isSubjectMatch(itemSubj, standardizedTargetSubject)
                  const matchWd = itemWeekday === leaveWeekday || itemWeekday === (leaveWeekday === 0 ? 7 : leaveWeekday)
                  
                  if (matchCls && matchSubj && matchWd) {
                    lostSlotSet.add(`${req.leave_date}-P${item.period}`)
                  }
                })
              } else {
                lostSlotSet.add(`NODATE-${Math.random()}`)
              }
            }
          })
        }

        if (interruptions && interruptions.length > 0) {
          interruptions.forEach(int => {
            if (int.type === 'class') {
              const startP = Number(int.start_period) || 1
              const endP = Number(int.end_period) || 1
              const intScope = int.scope ? int.scope.trim() : ''
              const targetDisp = int.target_display ? int.target_display.trim() : ''
              const intGrade = Number(int.grade)
              const intClass = cleanString(int.class_name)
              const clsName = cleanString(cls.class_name)

              const intDate = new Date(int.interruption_date)
              const intWeekday = intDate.getDay()

              const isClassAffected = 
                intScope === 'all' || 
                targetDisp.includes('SEMUA') || targetDisp.includes('ALL') ||
                (intScope === 'grade' && intGrade === Number(cls.grade)) ||
                targetDisp.includes(`GRADE ${cls.grade}`) || targetDisp.includes(`TAHUN ${cls.grade}`) ||
                (intScope === 'class' && (intClass.includes(clsName) || clsName.includes(intClass || ''))) ||
                targetDisp.includes(clsName)

              if (isClassAffected) {
                enrichedTimetables.forEach(item => {
                  const itemClass = cleanString(item.class_name)
                  const itemPeriod = Number(item.period)
                  const rawSubj = item.subject || item.subject_name || item.teacher_info?.subject_name || ''
                  const itemWeekday = Number(item.weekday)

                  const matchClass = itemClass === clsName || itemClass.includes(clsName) || clsName.includes(itemClass)
                  const matchPeriod = itemPeriod >= startP && itemPeriod <= endP
                  const matchSubject = isSubjectMatch(rawSubj, standardizedTargetSubject)
                  const matchWeekday = itemWeekday === intWeekday || itemWeekday === (intWeekday === 0 ? 7 : intWeekday)

                  if (matchClass && matchPeriod && matchSubject && matchWeekday) {
                    lostSlotSet.add(`${int.interruption_date}-P${item.period}`)
                  }
                })
              }
            }
          })
        }

        let lostCount = lostSlotSet.size

        const plannedAccumulated = currentWeek * weeklyPeriods
        const actual = Math.max(0, Number((plannedAccumulated - lostCount).toFixed(1)))
        const moeTarget = t.kpm_min_hours || 160
        const theoryProgress = Math.round(moeTarget * (currentWeek / totalSchoolWeeks))
        const gap = Number((actual - theoryProgress).toFixed(1))
        
        let status = 'TARGET MET'
        if (gap < 0) {
          status = 'NOT MET'
        } else if (gap > 0) {
          status = 'EXCEEDED TARGET'
        } else {
          status = 'TARGET MET'
        }

        results.push({
          grade: cls.grade,
          class_name: cls.class_name,
          subject_name: t.subject_name,
          teacher_name: assignedTeacherName,
          target: totalTarget,
          expected: theoryProgress,
          lostCount: lostCount,
          actual: actual,
          gap: gap,
          status: status
        })
      }
    }

    analysisList.value = results
    toast.success("ANALYTICS DATA REFRESHED SUCCESSFULLY!")
  } catch (err) {
    console.error("ANALYTICS ERROR:", err)
    toast.error("FAILED TO LOAD DATA: " + err.message)
  } finally {
    loading.value = false
  }
}

const filteredAnalysisList = computed(() => {
  let list = analysisList.value.filter(item => {
    const matchGrade = filterGrade.value === 'all' || Number(item.grade) === Number(filterGrade.value)
    const matchClass = filterClass.value === 'all' || item.class_name === filterClass.value
    const matchSubject = filterSubject.value === 'all' || standardizeSubjectName(filterSubject.value) === standardizeSubjectName(item.subject_name)
    const matchTeacher = filterTeacher.value === 'all' || (item.teacher_name && cleanString(item.teacher_name) === cleanString(filterTeacher.value))
    return matchGrade && matchClass && matchSubject && matchTeacher
  })

  return list.sort((a, b) => {
    let valA = a[sortKey.value]
    let valB = b[sortKey.value]

    if (valA === undefined || valA === null) valA = ''
    if (valB === undefined || valB === null) valB = ''

    if (typeof valA === 'number' && typeof valB === 'number') {
      return sortOrder.value === 'asc' ? valA - valB : valB - valA
    } else {
      const strA = String(valA)
      const strB = String(valB)
      return sortOrder.value === 'asc' 
        ? strA.localeCompare(strB, 'en') 
        : strB.localeCompare(strA, 'en')
    }
  })
})

const analysisSummary = computed(() => {
  const total = filteredAnalysisList.value.length
  const met = filteredAnalysisList.value.filter(i => i.status === 'TARGET MET' || i.status === 'EXCEEDED TARGET').length
  const unmet = filteredAnalysisList.value.filter(i => i.status === 'NOT MET').length
  return { total, met, unmet }
})

const completionRate = computed(() => {
  if (analysisSummary.value.total === 0) return 0
  return Math.round((analysisSummary.value.met / analysisSummary.value.total) * 100)
})

const getGradeStats = (g) => {
  const list = filteredAnalysisList.value.filter(i => Number(i.grade) === Number(g))
  const met = list.filter(i => i.status === 'TARGET MET' || i.status === 'EXCEEDED TARGET').length
  const unmet = list.filter(i => i.status === 'NOT MET').length
  return { met, unmet, total: list.length }
}

const subjectLossRanking = computed(() => {
  const map = {}
  filteredAnalysisList.value.forEach(i => {
    if (!map[i.subject_name]) map[i.subject_name] = 0
    map[i.subject_name] += i.lostCount
  })
  return Object.keys(map).map(subj => ({
    subject: subj,
    lost: map[subj]
  })).sort((a, b) => b.lost - a.lost)
})

const maxSubjectLoss = computed(() => {
  if (subjectLossRanking.value.length === 0) return 1
  return Math.max(...subjectLossRanking.value.map(i => i.lost), 1)
})

const exportPdfReport = () => {
  window.print()
}

const addTarget = async () => {
  if (!newTarget.value.subject_name.trim()) return toast.error("PLEASE ENTER A SUBJECT NAME!")
  try {
    const { error } = await supabase.from('subject_targets').insert([{
      grade: newTarget.value.grade,
      subject_name: newTarget.value.subject_name.trim().toUpperCase(),
      planned_periods: newTarget.value.planned_periods,
      kpm_min_hours: newTarget.value.kpm_min_hours
    }])
    if (error) throw error
    toast.success("TARGET ADDED SUCCESSFULLY!")
    newTarget.value.subject_name = ''
    loadAnalyticsData()
  } catch (err) {
    toast.error("FAILED TO ADD: " + err.message)
  }
}

const deleteTarget = async (id) => {
  try {
    const { error } = await supabase.from('subject_targets').delete().eq('id', id)
    if (error) throw error
    toast.success("TARGET DELETED")
    loadAnalyticsData()
  } catch (err) {
    toast.error("FAILED TO DELETE: " + err.message)
  }
}

onMounted(() => {
  loadAnalyticsData()
})

onActivated(() => {
  loadAnalyticsData()
})
</script>

<style scoped>
.print-header {
  display: none;
}

@media print {
  @page {
    size: portrait;
    margin: 10mm;
  }
  .no-print {
    display: none !important;
  }
  .print-header {
    display: block !important;
    margin-bottom: 20px;
  }
  body, * {
    font-family: "Microsoft YaHei", "PingFang SC", "Heiti SC", "WenQuanYi Micro Hei", Arial, sans-serif !important;
    background: white !important;
    color: black !important;
    -webkit-print-color-adjust: exact;
    print-color-adjust: exact;
  }
  .print-summary {
    display: grid !important;
    grid-template-columns: repeat(3, 1fr) !important;
    gap: 15px !important;
    margin-bottom: 20px;
  }
  .print-summary > div {
    border: 1px solid #94a3b8 !important;
    padding: 15px !important;
    border-radius: 12px !important;
  }
  .print-table {
    width: 100% !important;
    border-collapse: collapse !important;
    font-size: 11px !important;
  }
  .print-table th, .print-table td {
    border: 1px solid #cbd5e1 !important;
    padding: 8px 10px !important;
  }
  .print-table th {
    background-color: #f1f5f9 !important;
    color: #0f172a !important;
  }
}
</style>