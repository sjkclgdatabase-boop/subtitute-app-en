<template>
  <div class="p-4 sm:p-8 mx-auto min-h-screen space-y-8 min-w-[1024px] print:p-0 print:max-w-none print:min-w-0">
    
    <!-- Top Header: Unified card style and gradient title -->
    <div class="no-print bg-white rounded-3xl p-6 sm:p-8 shadow-sm ring-1 ring-slate-900/5 space-y-2">
      <h1 class="text-2xl sm:text-3xl font-extrabold tracking-tight text-transparent bg-clip-text bg-gradient-to-r from-slate-900 via-indigo-800 to-violet-800">
        ACADEMIC DATA ANALYSIS & MMI REPORT CENTER
      </h1>
      <p class="text-slate-500 text-xs sm:text-sm font-medium leading-relaxed">
        MULTI-DIMENSIONAL MONITORING OF TEACHING INTERRUPTIONS, SUBJECT IMPACTS, AND SUBSTITUTE WORKLOADS.
      </p>
    </div>

    <!-- 🌐 Global Date Range Filter Control Panel -->
    <div class="no-print bg-white p-6 rounded-3xl shadow-sm ring-1 ring-slate-900/5 flex flex-wrap items-center justify-between gap-4">
      <div class="flex items-center gap-3">
        <div class="w-10 h-10 rounded-2xl bg-indigo-50 text-indigo-600 flex items-center justify-center font-bold text-lg">
          📅
        </div>
        <div>
          <div class="text-xs font-bold text-slate-400 uppercase tracking-wider">DATE RANGE FILTER</div>
          <div class="text-xs font-extrabold text-slate-800">ALL REPORTS AND CHARTS ARE CALCULATED IN REAL-TIME ACCORDING TO THE TIME PERIOD</div>
        </div>
      </div>
      
      <div class="flex flex-wrap items-center gap-3">
        <div class="flex items-center gap-2 bg-slate-50 border border-slate-200 px-4 h-11 rounded-2xl">
          <span class="text-xs font-bold text-slate-500">FROM</span>
          <input 
            type="date" 
            v-model="startDate" 
            @change="loadAllData"
            class="bg-transparent text-xs font-bold text-slate-700 focus:outline-none cursor-pointer"
          />
        </div>
        <span class="text-slate-400 font-bold">-</span>
        <div class="flex items-center gap-2 bg-slate-50 border border-slate-200 px-4 h-11 rounded-2xl">
          <span class="text-xs font-bold text-slate-500">TO</span>
          <input 
            type="date" 
            v-model="endDate" 
            @change="loadAllData"
            class="bg-transparent text-xs font-bold text-slate-700 focus:outline-none cursor-pointer"
          />
        </div>
        <button 
          @click="resetDateFilter" 
          class="px-5 h-11 bg-slate-100 hover:bg-slate-200 text-slate-700 rounded-2xl text-xs font-bold transition cursor-pointer"
        >
          RESET
        </button>
      </div>
    </div>

    <!-- Top Bar: Analysis Perspective Switcher (Tabs) -->
    <div class="no-print flex flex-wrap gap-2 bg-white p-3 rounded-3xl shadow-sm ring-1 ring-slate-900/5">
      <button 
        @click="currentTab = 'overview'" 
        :class="currentTab === 'overview' ? 'bg-slate-900 text-white shadow-sm' : 'text-slate-600 hover:bg-slate-100'"
        class="px-4 py-2.5 rounded-2xl text-xs font-bold transition-all cursor-pointer whitespace-nowrap"
      >
        📊 OVERVIEW & SUBSTITUTE WORKLOAD
      </button>
      <button 
        @click="currentTab = 'reason'" 
        :class="currentTab === 'reason' ? 'bg-slate-900 text-white shadow-sm' : 'text-slate-600 hover:bg-slate-100'"
        class="px-4 py-2.5 rounded-2xl text-xs font-bold transition-all cursor-pointer whitespace-nowrap"
      >
        ⚠️ INTERRUPTION REASON ANALYSIS
      </button>
      <button 
        @click="currentTab = 'trend'" 
        :class="currentTab === 'trend' ? 'bg-slate-900 text-white shadow-sm' : 'text-slate-600 hover:bg-slate-100'"
        class="px-4 py-2.5 rounded-2xl text-xs font-bold transition-all cursor-pointer whitespace-nowrap"
      >
        📅 INTERRUPTION DATE PEAKS
      </button>
      <button 
        @click="currentTab = 'class'" 
        :class="currentTab === 'class' ? 'bg-slate-900 text-white shadow-sm' : 'text-slate-600 hover:bg-slate-100'"
        class="px-4 py-2.5 rounded-2xl text-xs font-bold transition-all cursor-pointer whitespace-nowrap"
      >
        🏫 AFFECTED CLASSES ANALYSIS
      </button>
      <button 
        @click="currentTab = 'subject'" 
        :class="currentTab === 'subject' ? 'bg-slate-900 text-white shadow-sm' : 'text-slate-600 hover:bg-slate-100'"
        class="px-4 py-2.5 rounded-2xl text-xs font-bold transition-all cursor-pointer whitespace-nowrap"
      >
        📚 AFFECTED SUBJECTS
      </button>
      <button 
        @click="currentTab = 'affectedTeacher'" 
        :class="currentTab === 'affectedTeacher' ? 'bg-slate-900 text-white shadow-sm' : 'text-slate-600 hover:bg-slate-100'"
        class="px-4 py-2.5 rounded-2xl text-xs font-bold transition-all cursor-pointer whitespace-nowrap"
      >
        📉 AFFECTED TEACHERS (TOP 5)
      </button>
      <button 
        @click="currentTab = 'teacher'" 
        :class="currentTab === 'teacher' ? 'bg-slate-900 text-white shadow-sm' : 'text-slate-600 hover:bg-slate-100'"
        class="px-4 py-2.5 rounded-2xl text-xs font-bold transition-all cursor-pointer whitespace-nowrap"
      >
        👨‍🏫 OVERALL TEACHER RECORDS
      </button>
    </div>

    <!-- Print-specific Report Header Title -->
    <div class="print-header hidden mb-6 text-center">
      <h2 class="text-2xl font-extrabold text-slate-900">ACADEMIC DATA ANALYSIS & MMI REPORT</h2>
      <p class="text-xs text-slate-600 mt-1">
        TIME PERIOD: [{{ startDate || 'NO LIMIT' }} TO {{ endDate || 'NO LIMIT' }}]
      </p>
    </div>

    <!-- ================= TAB 1: Comprehensive Overview ================= -->
    <div v-if="currentTab === 'overview'" class="space-y-8 animate-fadeIn">
      <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-6">
        <div class="bg-white p-6 rounded-3xl shadow-sm ring-1 ring-slate-900/5">
          <div class="text-xs font-bold text-slate-400 uppercase tracking-wider">TOTAL INTERRUPTION SLOTS</div>
          <div class="text-3xl font-black text-slate-900 mt-2">{{ totalInterruptionPeriods }} SLOTS</div>
        </div>
        <div class="bg-white p-6 rounded-3xl shadow-sm ring-1 ring-slate-900/5">
          <div class="text-xs font-bold text-slate-400 uppercase tracking-wider">TOTAL SUBSTITUTE CASES</div>
          <div class="text-3xl font-black text-indigo-600 mt-2">{{ totalSubstituteCount }} TIMES</div>
        </div>
        <div class="bg-white p-6 rounded-3xl shadow-sm ring-1 ring-slate-900/5">
          <div class="text-xs font-bold text-slate-400 uppercase tracking-wider">TOTAL INTERRUPTION RECORDS</div>
          <div class="text-3xl font-black text-slate-900 mt-2">{{ interruptionLogs.length }} RECORDS</div>
        </div>
      </div>

      <div class="bg-white p-8 rounded-3xl shadow-sm ring-1 ring-slate-900/5">
        <div class="flex justify-between items-center mb-6">
          <h2 class="text-base font-bold text-slate-900">⚖️ TOP 5 SUBSTITUTE WORKLOAD RANKING</h2>
          <button @click="exportSinglePdf" class="no-print px-4 py-2 bg-slate-100 hover:bg-slate-200 text-slate-700 rounded-xl text-xs font-bold transition flex items-center gap-1 cursor-pointer">
            📥 PRINT / SAVE AS PDF
          </button>
        </div>
        <div class="grid grid-cols-1 sm:grid-cols-5 gap-4">
          <div v-for="(t, idx) in sortedSubstituteStats.slice(0, 5)" :key="t.name" class="p-4 bg-slate-50 border border-slate-100 rounded-2xl flex flex-col justify-between">
            <div class="text-xs font-bold text-slate-400">RANK #{{ idx + 1 }}</div>
            <div class="my-2">
              <div class="text-sm font-extrabold text-slate-900">{{ t.name }}</div>
              <div class="text-[11px] text-slate-500">{{ t.subject || 'GENERAL SUBJECT' }}</div>
            </div>
            <div class="text-xs font-bold text-indigo-600 bg-white px-3 py-1.5 rounded-xl shadow-sm text-center border border-slate-200">
              {{ t.count }} TIMES
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- ================= TAB 2: Reason Analysis ================= -->
    <div v-if="currentTab === 'reason'" class="bg-white p-8 rounded-3xl shadow-sm ring-1 ring-slate-900/5 space-y-6 animate-fadeIn">
      <div class="flex justify-between items-center">
        <div>
          <h2 class="text-lg font-bold text-slate-900">⚠️ PDPC INTERRUPTION REASON STATISTICS</h2>
          <p class="text-xs text-slate-500 mt-1">STATISTICS OF TIME SLOTS AND PERCENTAGES TAKEN BY EACH ACTIVITY.</p>
        </div>
        <button @click="exportSinglePdf" class="no-print px-4 py-2 bg-slate-100 hover:bg-slate-200 text-slate-700 rounded-xl text-xs font-bold transition flex items-center gap-1 cursor-pointer">
          📥 PRINT / SAVE AS PDF
        </button>
      </div>
      <div v-if="reasonStats.length === 0" class="text-xs text-slate-400 py-12 text-center border border-dashed rounded-2xl">NO INTERRUPTION RECORDS IN THIS PERIOD</div>
      <div v-else class="space-y-4">
        <div v-for="item in reasonStats" :key="item.reason" class="space-y-1.5 p-4 bg-slate-50 rounded-2xl">
          <div class="flex justify-between text-xs font-bold text-slate-800">
            <span>{{ item.reason }}</span>
            <span class="text-indigo-600">{{ item.count }} SLOTS ({{ item.percentage }}%)</span>
          </div>
          <div class="w-full h-3 bg-slate-200 rounded-full overflow-hidden">
            <div class="h-full bg-indigo-600 rounded-full" :style="{ width: item.percentage + '%' }"></div>
          </div>
        </div>
      </div>
    </div>

    <!-- ================= TAB 3: Date Peaks ================= -->
    <div v-if="currentTab === 'trend'" class="bg-white p-8 rounded-3xl shadow-sm ring-1 ring-slate-900/5 space-y-6 animate-fadeIn">
      <div class="flex justify-between items-center">
        <div>
          <h2 class="text-lg font-bold text-slate-900">📅 INTERRUPTION STATISTICS BY DAY</h2>
          <p class="text-xs text-slate-500 mt-1">DISTRIBUTION OF INTERRUPTION FREQUENCY BY DAY OF THE WEEK.</p>
        </div>
        <button @click="exportSinglePdf" class="no-print px-4 py-2 bg-slate-100 hover:bg-slate-200 text-slate-700 rounded-xl text-xs font-bold transition flex items-center gap-1 cursor-pointer">
          📥 PRINT / SAVE AS PDF
        </button>
      </div>
      <div class="grid grid-cols-1 sm:grid-cols-5 gap-4">
        <div v-for="dayData in dayOfWeekStats" :key="dayData.day" class="p-5 bg-slate-50 border border-slate-100 rounded-2xl text-center space-y-2">
          <div class="text-xs font-bold text-slate-500 uppercase">{{ dayData.day }}</div>
          <div class="text-2xl font-black text-slate-900">{{ dayData.count }} <span class="text-xs font-normal text-slate-400">SLOTS</span></div>
          <div class="text-[11px] text-indigo-600 font-semibold bg-indigo-50 py-1 rounded-lg">AT {{ dayData.percentage }}%</div>
        </div>
      </div>
    </div>

    <!-- ================= TAB 4: Class Interruption Analysis ================= -->
    <div v-if="currentTab === 'class'" class="bg-white p-8 rounded-3xl shadow-sm ring-1 ring-slate-900/5 space-y-6 animate-fadeIn">
      <div class="flex justify-between items-center">
        <div>
          <h2 class="text-lg font-bold text-slate-900">🏫 CLASS INTERRUPTION STATISTICS</h2>
          <p class="text-xs text-slate-500 mt-1">CUMULATIVE NUMBER OF AFFECTED CLASS TIME SLOTS.</p>
        </div>
        <button @click="exportSinglePdf" class="no-print px-4 py-2 bg-slate-100 hover:bg-slate-200 text-slate-700 rounded-xl text-xs font-bold transition flex items-center gap-1 cursor-pointer">
          📥 PRINT / SAVE AS PDF
        </button>
      </div>

      <div class="overflow-x-auto">
        <table class="w-full text-left text-xs border-collapse print-table">
          <thead>
            <tr class="bg-slate-50 text-slate-500 uppercase tracking-wider select-none font-semibold">
              <th @click="sortClassTable('className')" class="p-4 border-b cursor-pointer hover:bg-slate-100 transition">
                CLASS NAME <span class="text-indigo-600">{{ classSortKey === 'className' ? (classSortAsc ? '▲' : '▼') : '↕' }}</span>
              </th>
              <th @click="sortClassTable('totalPeriods')" class="p-4 border-b cursor-pointer hover:bg-slate-100 transition">
                AFFECTED SLOTS <span class="text-indigo-600">{{ classSortKey === 'totalPeriods' ? (classSortAsc ? '▲' : '▼') : '↕' }}</span>
              </th>
              <th @click="sortClassTable('percentage')" class="p-4 border-b cursor-pointer hover:bg-slate-100 transition">
                PERCENTAGE <span class="text-indigo-600">{{ classSortKey === 'percentage' ? (classSortAsc ? '▲' : '▼') : '↕' }}</span>
              </th>
            </tr>
          </thead>
          <tbody class="divide-y divide-slate-100 font-medium text-slate-800">
            <tr v-for="c in sortedClassStats" :key="c.className" class="hover:bg-slate-50">
              <td class="p-4 font-bold text-slate-900">{{ c.className }}</td>
              <td class="p-4 font-bold text-indigo-600">{{ c.totalPeriods }} SLOTS</td>
              <td class="p-4 text-slate-600">{{ c.percentage }}%</td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>

    <!-- ================= TAB 5: Most Affected Subjects Ranking ================= -->
    <div v-if="currentTab === 'subject'" class="bg-white p-8 rounded-3xl shadow-sm ring-1 ring-slate-900/5 space-y-6 animate-fadeIn">
      <div class="flex justify-between items-center">
        <div>
          <h2 class="text-lg font-bold text-slate-900">📚 SUBJECT INTERRUPTION STATISTICS</h2>
          <p class="text-xs text-slate-500 mt-1">CUMULATIVE SUBJECT TIME SLOTS INTERRUPTED DUE TO LEAVE OR ACTIVITIES.</p>
        </div>
        <button @click="exportSinglePdf" class="no-print px-4 py-2 bg-slate-100 hover:bg-slate-200 text-slate-700 rounded-xl text-xs font-bold transition flex items-center gap-1 cursor-pointer">
          📥 PRINT / SAVE AS PDF
        </button>
      </div>

      <div class="overflow-x-auto">
        <table class="w-full text-left text-xs border-collapse print-table">
          <thead>
            <tr class="bg-slate-50 text-slate-500 uppercase tracking-wider select-none font-semibold">
              <th @click="sortSubjectTable('subjectName')" class="p-4 border-b cursor-pointer hover:bg-slate-100 transition">
                SUBJECT NAME <span class="text-indigo-600">{{ subjectSortKey === 'subjectName' ? (subjectSortAsc ? '▲' : '▼') : '↕' }}</span>
              </th>
              <th @click="sortSubjectTable('totalPeriods')" class="p-4 border-b cursor-pointer hover:bg-slate-100 transition">
                AFFECTED SLOTS <span class="text-indigo-600">{{ subjectSortKey === 'totalPeriods' ? (subjectSortAsc ? '▲' : '▼') : '↕' }}</span>
              </th>
            </tr>
          </thead>
          <tbody class="divide-y divide-slate-100 font-medium text-slate-800">
            <tr v-for="s in sortedSubjectStats" :key="s.subjectName" class="hover:bg-slate-50">
              <td class="p-4 font-bold text-slate-900">{{ s.subjectName }}</td>
              <td class="p-4 font-bold text-indigo-600">{{ s.totalPeriods }} SLOTS</td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>

    <!-- ================= TAB 6: Most Affected Teachers Ranking (Top 5) ================= -->
    <div v-if="currentTab === 'affectedTeacher'" class="bg-white p-8 rounded-3xl shadow-sm ring-1 ring-slate-900/5 space-y-6 animate-fadeIn">
      <div class="flex justify-between items-center">
        <div>
          <h2 class="text-lg font-bold text-slate-900">📉 TEACHER CLASS INTERRUPTION STATISTICS (TOP 5)</h2>
          <p class="text-xs text-slate-500 mt-1">LIST OF TOP 5 TEACHERS WHOSE CLASSES ARE MOST FREQUENTLY INTERRUPTED.</p>
        </div>
        <button @click="exportSinglePdf" class="no-print px-4 py-2 bg-slate-100 hover:bg-slate-200 text-slate-700 rounded-xl text-xs font-bold transition flex items-center gap-1 cursor-pointer">
          📥 PRINT / SAVE AS PDF
        </button>
      </div>

      <div class="overflow-x-auto">
        <table class="w-full text-left text-xs border-collapse print-table">
          <thead>
            <tr class="bg-slate-50 text-slate-500 uppercase tracking-wider select-none font-semibold">
              <th @click="sortAffectedTeacherTable('teacherName')" class="p-4 border-b cursor-pointer hover:bg-slate-100 transition">
                TEACHER NAME <span class="text-indigo-600">{{ affectedTeacherSortKey === 'teacherName' ? (affectedTeacherSortAsc ? '▲' : '▼') : '↕' }}</span>
              </th>
              <th @click="sortAffectedTeacherTable('totalPeriods')" class="p-4 border-b cursor-pointer hover:bg-slate-100 transition">
                AFFECTED SLOTS <span class="text-indigo-600">{{ affectedTeacherSortKey === 'totalPeriods' ? (affectedTeacherSortAsc ? '▲' : '▼') : '↕' }}</span>
              </th>
            </tr>
          </thead>
          <tbody class="divide-y divide-slate-100 font-medium text-slate-800">
            <tr v-for="(t, index) in sortedAffectedTeacherStats.slice(0, 5)" :key="t.teacherName" class="hover:bg-slate-50">
              <td class="p-4 font-bold text-slate-900">{{ t.teacherName }}</td>
              <td class="p-4 font-bold text-amber-600">{{ t.totalPeriods }} SLOTS</td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>

    <!-- ================= TAB 7: Overall Teacher Interruption Master Table ================= -->
    <div v-if="currentTab === 'teacher'" class="bg-white p-8 rounded-3xl shadow-sm ring-1 ring-slate-900/5 space-y-6 animate-fadeIn">
      <div class="flex justify-between items-center">
        <div>
          <h2 class="text-lg font-bold text-slate-900">👨‍🏫 OVERALL SUBSTITUTE WORKLOAD & TEACHER INTERRUPTIONS</h2>
          <p class="text-xs text-slate-500 mt-1">DISPLAY OF ALL REGISTERED TEACHERS ALONG WITH SUBSTITUTE LOADS AND CLASS INTERRUPTIONS.</p>
        </div>
        <button @click="exportSinglePdf" class="no-print px-4 py-2 bg-slate-100 hover:bg-slate-200 text-slate-700 rounded-xl text-xs font-bold transition flex items-center gap-1 cursor-pointer">
          📥 PRINT / SAVE AS PDF
        </button>
      </div>

      <div class="overflow-x-auto">
        <table class="w-full text-left text-xs border-collapse print-table">
          <thead>
            <tr class="bg-slate-50 text-slate-500 uppercase tracking-wider select-none font-semibold">
              <th @click="sortTeacherTable('name')" class="p-4 border-b cursor-pointer hover:bg-slate-100 transition">
                TEACHER NAME <span class="text-indigo-600">{{ teacherSortKey === 'name' ? (teacherSortAsc ? '▲' : '▼') : '↕' }}</span>
              </th>
              <th @click="sortTeacherTable('subject')" class="p-4 border-b cursor-pointer hover:bg-slate-100 transition">
                SUBJECT TAUGHT <span class="text-indigo-600">{{ teacherSortKey === 'subject' ? (teacherSortAsc ? '▲' : '▼') : '↕' }}</span>
              </th>
              <th @click="sortTeacherTable('count')" class="p-4 border-b cursor-pointer hover:bg-slate-100 transition">
                TOTAL SUBSTITUTIONS <span class="text-indigo-600">{{ teacherSortKey === 'count' ? (teacherSortAsc ? '▲' : '▼') : '↕' }}</span>
              </th>
              <th @click="sortTeacherTable('interruptedCount')" class="p-4 border-b cursor-pointer hover:bg-slate-100 transition">
                AFFECTED SLOTS <span class="text-indigo-600">{{ teacherSortKey === 'interruptedCount' ? (teacherSortAsc ? '▲' : '▼') : '↕' }}</span>
              </th>
            </tr>
          </thead>
          <tbody class="divide-y divide-slate-100 font-medium text-slate-800">
            <tr v-for="stat in sortedTeacherStats" :key="stat.name" class="hover:bg-slate-50">
              <td class="p-4 font-bold text-slate-900">{{ stat.name }}</td>
              <td class="p-4 text-slate-600">{{ stat.subject || '-' }}</td>
              <td class="p-4 font-bold text-indigo-600">{{ stat.count }} SLOTS</td>
              <td class="p-4 font-bold text-amber-600">{{ stat.interruptedCount }} SLOTS</td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>

  </div>
</template>

<script setup>
import { ref, computed, onMounted, onActivated } from 'vue'
import { supabase } from '../services/supabase'

const currentTab = ref('overview')

// 🌐 Global Date Range State
const startDate = ref('')
const endDate = ref('')

const stats = ref([])
const interruptionLogs = ref([])
const reasonStats = ref([])
const dayOfWeekStats = ref([])
const classStats = ref([])
const subjectStats = ref([])
const affectedTeacherStats = ref([])

// General Safe Sorting Method
const smartSort = (valA, valB, asc) => {
  const a = valA ?? '';
  const b = valB ?? '';
  
  if (typeof a === 'number' && typeof b === 'number') {
    return asc ? a - b : b - a
  }
  const strA = String(a).toLowerCase();
  const strB = String(b).toLowerCase();
  return asc ? strA.localeCompare(strB) : strB.localeCompare(strA)
}

// Sorting States and Methods
const classSortKey = ref('totalPeriods')
const classSortAsc = ref(false)
const sortClassTable = (key) => {
  if (classSortKey.value === key) classSortAsc.value = !classSortAsc.value
  else { classSortKey.value = key; classSortAsc.value = true }
}
const sortedClassStats = computed(() => [...classStats.value].sort((a, b) => smartSort(a[classSortKey.value], b[classSortKey.value], classSortAsc.value)))

const subjectSortKey = ref('totalPeriods')
const subjectSortAsc = ref(false)
const sortSubjectTable = (key) => {
  if (subjectSortKey.value === key) subjectSortAsc.value = !subjectSortAsc.value
  else { subjectSortKey.value = key; subjectSortAsc.value = true }
}
const sortedSubjectStats = computed(() => [...subjectStats.value].sort((a, b) => smartSort(a[subjectSortKey.value], b[subjectSortKey.value], subjectSortAsc.value)))

const affectedTeacherSortKey = ref('totalPeriods')
const affectedTeacherSortAsc = ref(false)
const sortAffectedTeacherTable = (key) => {
  if (affectedTeacherSortKey.value === key) affectedTeacherSortAsc.value = !affectedTeacherSortAsc.value
  else { affectedTeacherSortKey.value = key; affectedTeacherSortAsc.value = true }
}
const sortedAffectedTeacherStats = computed(() => [...affectedTeacherStats.value].sort((a, b) => smartSort(a[affectedTeacherSortKey.value], b[affectedTeacherSortKey.value], affectedTeacherSortAsc.value)))

const teacherSortKey = ref('count')
const teacherSortAsc = ref(false)
const sortTeacherTable = (key) => {
  if (teacherSortKey.value === key) teacherSortAsc.value = !teacherSortAsc.value
  else { teacherSortKey.value = key; teacherSortAsc.value = true }
}
const sortedTeacherStats = computed(() => [...stats.value].sort((a, b) => smartSort(a[teacherSortKey.value], b[teacherSortKey.value], teacherSortAsc.value)))

// Other Regular Statistics
const totalSubstituteCount = computed(() => stats.value.reduce((acc, cur) => acc + (cur.count || 0), 0))
const totalInterruptionPeriods = computed(() => interruptionLogs.value.reduce((acc, cur) => acc + ((cur.end_period || 0) - (cur.start_period || 0) + 1), 0))
const sortedSubstituteStats = computed(() => [...stats.value].sort((a, b) => (b.count || 0) - (a.count || 0)))

// Reset Date Selection
const resetDateFilter = () => {
  startDate.value = ''
  endDate.value = ''
  loadAllData()
}

// 🔄 Data Loading Function
const loadAllData = async () => {
  const { data: teachers } = await supabase.from('teachers').select('*')
  
  // 1. Fetch substitute records (to exclude swaps)
  let assignQuery = supabase
    .from('substitute_assignments')
    .select('sub_teacher_id, assignment_type, leave_request_id, leave_requests!inner(leave_date)')

  if (startDate.value) assignQuery = assignQuery.gte('leave_requests.leave_date', startDate.value)
  if (endDate.value) assignQuery = assignQuery.lte('leave_requests.leave_date', endDate.value)

  const { data: assignments } = await assignQuery

  // Collect leave_request_ids of all swaps
  const swapLeaveIds = new Set()
  assignments?.forEach(a => {
    if (a.assignment_type === 'swap' && a.leave_request_id) {
      swapLeaveIds.add(a.leave_request_id)
    }
  })

  // 2. Fetch MMI records
  let mmiQuery = supabase.from('mmi_interruptions').select('*')
  if (startDate.value) mmiQuery = mmiQuery.gte('interruption_date', startDate.value)
  if (endDate.value) mmiQuery = mmiQuery.lte('interruption_date', endDate.value)
  const { data: mmiData } = await mmiQuery

  if (mmiData) interruptionLogs.value = mmiData

  // 3. Fetch precise leave records within this timeframe (leave_requests)
  let leaveQuery = supabase.from('leave_requests').select('*')
  if (startDate.value) leaveQuery = leaveQuery.gte('leave_date', startDate.value)
  if (endDate.value) leaveQuery = leaveQuery.lte('leave_date', endDate.value)
  const { data: leaveData } = await leaveQuery

  // ================== Begin Statistics ==================
  const teacherMap = {}
  const teacherNameSet = new Set()
  
  teachers?.forEach(t => {
    teacherMap[t.id] = { name: t.name, subject: t.subject, count: 0, interruptedCount: 0 }
    teacherNameSet.add(t.name.trim().toUpperCase())
  })

  // Teacher substitute count statistics (filtering swaps)
  assignments?.forEach(a => {
    if (a.assignment_type !== 'swap' && a.sub_teacher_id && teacherMap[a.sub_teacher_id]) {
      teacherMap[a.sub_teacher_id].count++
    }
  })

  // Teacher interruption count statistics (based on MMI)
  const teacherInterruptionMap = {}
  mmiData?.forEach(l => {
    let rawTarget = (l.target_display || '').trim()
    let tName = ''
    if (rawTarget.includes('GURU:')) tName = rawTarget.replace('GURU:', '').trim()
    else if (rawTarget.includes('教师:')) tName = rawTarget.replace('教师:', '').trim()
    else if (teacherNameSet.has(rawTarget.toUpperCase())) tName = rawTarget

    if (tName) {
      const pCount = (l.end_period || 0) - (l.start_period || 0) + 1
      teacherInterruptionMap[tName.toUpperCase()] = (teacherInterruptionMap[tName.toUpperCase()] || 0) + pCount
    }
  })

  stats.value = Object.values(teacherMap).map(t => ({
    ...t,
    interruptedCount: teacherInterruptionMap[t.name.trim().toUpperCase()] || 0
  }))

  // MMI item proportion analysis and peak dates
  if (mmiData) {
    const totalPAll = mmiData.reduce((acc, cur) => acc + ((cur.end_period || 0) - (cur.start_period || 0) + 1), 0)
    const reasons = {}
    mmiData.forEach(l => { const pCount = (l.end_period || 0) - (l.start_period || 0) + 1; reasons[l.reason] = (reasons[l.reason] || 0) + pCount })
    reasonStats.value = Object.entries(reasons).map(([reason, count]) => ({ reason, count, percentage: totalPAll > 0 ? ((count / totalPAll) * 100).toFixed(1) : 0 })).sort((a, b) => b.count - a.count)

    const dayNames = { 1: 'MONDAY', 2: 'TUESDAY', 3: 'WEDNESDAY', 4: 'THURSDAY', 5: 'FRIDAY', 6: 'SATURDAY', 7: 'SUNDAY' }
    const daysCount = {}
    mmiData.forEach(l => { const dIndex = new Date(l.interruption_date).getDay() || 7; const dName = dayNames[dIndex] || 'OTHER'; const pCount = (l.end_period || 0) - (l.start_period || 0) + 1; daysCount[dName] = (daysCount[dName] || 0) + pCount })
    dayOfWeekStats.value = ['MONDAY', 'TUESDAY', 'WEDNESDAY', 'THURSDAY', 'FRIDAY'].map(day => ({ day, count: daysCount[day] || 0, percentage: totalPAll > 0 ? (((daysCount[day] || 0) / totalPAll) * 100).toFixed(1) : 0 }))
  }

  // ================== Class and Subject Interruption Calculation ==================
  const classMap = {}
  const subjectMap = {}
  let totalClassPeriods = 0

  // Step A: Incorporate pure class activities (from MMI table)
  mmiData?.forEach(l => { 
    let rawTarget = (l.target_display || '').trim(); 
    if (rawTarget.includes('GURU') || rawTarget.includes('教师') || teacherNameSet.has(rawTarget.toUpperCase())) return; 
    
    const pCount = (l.end_period || 0) - (l.start_period || 0) + 1; 
    
    if (rawTarget.startsWith('KELAS:') || rawTarget.startsWith('班级:')) {
      const cNames = rawTarget.replace(/(?:KELAS|班级)[:：]/, '').split(',');
      cNames.forEach(c => {
        const cleanC = c.trim();
        if (cleanC) {
          classMap[cleanC] = (classMap[cleanC] || 0) + pCount;
          totalClassPeriods += pCount;
        }
      });
    } else {
      const cName = rawTarget || 'ALL CLASSES'; 
      classMap[cName] = (classMap[cName] || 0) + pCount; 
      totalClassPeriods += pCount;
    }
  })

  // Step B: Incorporate class impacts from teacher leaves (from leave_requests table)
  leaveData?.forEach(req => {
    if (swapLeaveIds.has(req.id)) return;

    const cNames = req.class_name ? req.class_name.split('/') : ['UNKNOWN CLASS'];
    cNames.forEach(c => {
      const cleanName = c.trim();
      if (cleanName) {
        classMap[cleanName] = (classMap[cleanName] || 0) + 1; 
        totalClassPeriods += 1;
      }
    })

    const sub = req.subject ? req.subject.trim() : 'UNKNOWN SUBJECT';
    if (sub && sub !== 'UNKNOWN SUBJECT') {
      subjectMap[sub] = (subjectMap[sub] || 0) + 1;
    }
  })

  classStats.value = Object.entries(classMap)
    .map(([className, totalPeriods]) => ({ 
      className, 
      totalPeriods, 
      percentage: totalClassPeriods > 0 ? ((totalPeriods / totalClassPeriods) * 100).toFixed(1) : 0 
    }))
    .sort((a, b) => b.totalPeriods - a.totalPeriods)

  subjectStats.value = Object.entries(subjectMap)
    .map(([subjectName, totalPeriods]) => ({ subjectName, totalPeriods }))
    .sort((a, b) => b.totalPeriods - a.totalPeriods)
  
  affectedTeacherStats.value = stats.value.filter(t => t.interruptedCount > 0).map(t => ({ teacherName: t.name, totalPeriods: t.interruptedCount })).sort((a, b) => b.totalPeriods - a.totalPeriods)
}

onMounted(loadAllData)

onActivated(() => {
  loadAllData()
})

// 🖨️ Export PDF / Print Action
const exportSinglePdf = () => {
  window.print()
}
</script>

<style scoped>
.print-header {
  display: none;
}

@media print {
  @page {
    size: portrait; /* Force portrait layout */
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