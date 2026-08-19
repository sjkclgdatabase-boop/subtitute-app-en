<template>
  <div class="p-4 sm:p-8 mx-auto min-h-screen space-y-8 min-w-[1024px] print:p-0 print:max-w-none print:min-w-0">
    
    <!-- 顶部标题 -->
    <div class="no-print bg-white rounded-3xl p-6 sm:p-8 shadow-sm ring-1 ring-slate-900/5 space-y-2">
      <h1 class="text-2xl sm:text-3xl font-extrabold tracking-tight text-transparent bg-clip-text bg-gradient-to-r from-slate-900 via-indigo-800 to-violet-800 flex items-center gap-3">
        <ChartNoAxesCombined class="w-8 h-8 text-indigo-700 shrink-0" />
        Academic Data Analysis & MMI Report Center
      </h1>
      <p class="text-slate-500 text-xs sm:text-sm font-medium leading-relaxed">
        Multi-dimensional monitoring of teaching interruptions, subject impact, and substitute loads (integrated with precise leave requests and school-wide activity tracking).
      </p>
    </div>

    <!-- 🌐 全局日期范围筛选器 -->
    <div class="no-print bg-white p-6 rounded-3xl shadow-sm ring-1 ring-slate-900/5 flex flex-wrap items-center justify-between gap-4">
      <div class="flex items-center gap-3">
        <div class="w-10 h-10 rounded-2xl bg-indigo-50 text-indigo-600 flex items-center justify-center font-bold text-lg">
          <CalendarDays class="w-5 h-5" />
        </div>
        <div>
          <div class="text-xs font-bold text-slate-400 uppercase tracking-wider">TIME RANGE FILTER</div>
          <div class="text-xs font-extrabold text-slate-800">Reports and leaderboards calculated in real-time</div>
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
          class="px-5 h-11 bg-slate-100 hover:bg-slate-200 text-slate-700 rounded-2xl text-xs font-bold transition cursor-pointer flex items-center gap-2"
        >
          <RefreshCw class="w-4 h-4" />
          RESET
        </button>
      </div>
    </div>

    <!-- 顶栏：Tab -->
    <div class="no-print flex flex-wrap gap-2 bg-white p-3 rounded-3xl shadow-sm ring-1 ring-slate-900/5">
      <button 
        @click="currentTab = 'overview'" 
        :class="currentTab === 'overview' ? 'bg-slate-900 text-white shadow-sm' : 'text-slate-600 hover:bg-slate-100'"
        class="px-4 py-2.5 rounded-2xl text-xs font-bold transition-all cursor-pointer whitespace-nowrap flex items-center gap-2"
      >
        <LayoutDashboard class="w-4 h-4" />
        Overview & Load
      </button>
      <button 
        @click="currentTab = 'reason'" 
        :class="currentTab === 'reason' ? 'bg-slate-900 text-white shadow-sm' : 'text-slate-600 hover:bg-slate-100'"
        class="px-4 py-2.5 rounded-2xl text-xs font-bold transition-all cursor-pointer whitespace-nowrap flex items-center gap-2"
      >
        <TriangleAlert class="w-4 h-4" />
        Reason Analysis
      </button>
      <button 
        @click="currentTab = 'trend'" 
        :class="currentTab === 'trend' ? 'bg-slate-900 text-white shadow-sm' : 'text-slate-600 hover:bg-slate-100'"
        class="px-4 py-2.5 rounded-2xl text-xs font-bold transition-all cursor-pointer whitespace-nowrap flex items-center gap-2"
      >
        <CalendarDays class="w-4 h-4" />
        Peak Dates
      </button>
      <button 
        @click="currentTab = 'class'" 
        :class="currentTab === 'class' ? 'bg-slate-900 text-white shadow-sm' : 'text-slate-600 hover:bg-slate-100'"
        class="px-4 py-2.5 rounded-2xl text-xs font-bold transition-all cursor-pointer whitespace-nowrap flex items-center gap-2"
      >
        <School class="w-4 h-4" />
        Class Analysis
      </button>
      <button 
        @click="currentTab = 'subject'" 
        :class="currentTab === 'subject' ? 'bg-slate-900 text-white shadow-sm' : 'text-slate-600 hover:bg-slate-100'"
        class="px-4 py-2.5 rounded-2xl text-xs font-bold transition-all cursor-pointer whitespace-nowrap flex items-center gap-2"
      >
        <BookOpen class="w-4 h-4" />
        Subject Analysis
      </button>
      <button 
        @click="currentTab = 'teacher'" 
        :class="currentTab === 'teacher' ? 'bg-slate-900 text-white shadow-sm' : 'text-slate-600 hover:bg-slate-100'"
        class="px-4 py-2.5 rounded-2xl text-xs font-bold transition-all cursor-pointer whitespace-nowrap flex items-center gap-2"
      >
        <UsersRound class="w-4 h-4" />
        Teacher Overview
      </button>
    </div>

    <!-- 打印报表抬头 -->
    <div class="print-header hidden mb-6 text-center">
      <h2 class="text-2xl font-extrabold text-slate-900">MMI Evaluation & Academic Data Report</h2>
      <p class="text-xs text-slate-600 mt-1">
        Period: [{{ startDate || 'All Time' }} TO {{ endDate || 'All Time' }}]
      </p>
    </div>

    <!-- TAB 1: 综合概览 -->
    <div v-if="currentTab === 'overview'" class="space-y-8 animate-fadeIn">
      <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-6">
        <div class="bg-white p-6 rounded-3xl shadow-sm ring-1 ring-slate-900/5">
          <div class="text-xs font-bold text-slate-400 uppercase tracking-wider flex items-center gap-2">
            <Clock3 class="w-4 h-4 text-slate-400" />
            TOTAL INTERRUPTED PERIODS
          </div>
          <div class="text-3xl font-black text-slate-900 mt-2">{{ totalInterruptionPeriods }} Slots</div>
        </div>
        <div class="bg-white p-6 rounded-3xl shadow-sm ring-1 ring-slate-900/5">
          <div class="text-xs font-bold text-slate-400 uppercase tracking-wider flex items-center gap-2">
            <ArrowLeftRight class="w-4 h-4 text-indigo-600" />
            TOTAL SUBSTITUTE ASSIGNMENTS
          </div>
          <div class="text-3xl font-black text-indigo-600 mt-2">{{ totalSubstituteCount }} Times</div>
        </div>
        <div class="bg-white p-6 rounded-3xl shadow-sm ring-1 ring-slate-900/5">
          <div class="text-xs font-bold text-slate-400 uppercase tracking-wider flex items-center gap-2">
            <FileText class="w-4 h-4 text-slate-400" />
            TOTAL INTERRUPTION RECORDS
          </div>
          <div class="text-3xl font-black text-slate-900 mt-2">{{ interruptionLogs.length }} Records</div>
        </div>
      </div>

      <div class="bg-white p-8 rounded-3xl shadow-sm ring-1 ring-slate-900/5">
        <div class="flex justify-between items-center mb-6">
          <h2 class="text-base font-bold text-slate-900 flex items-center gap-2">
            <Scale class="w-5 h-5 text-indigo-600" />
            HIGH-LOAD TEACHERS RANKING (TOP 5)
          </h2>
          <button @click="exportSinglePdf" class="no-print px-4 py-2 bg-slate-100 hover:bg-slate-200 text-slate-700 rounded-xl text-xs font-bold transition flex items-center gap-1.5 cursor-pointer">
            <Printer class="w-4 h-4" />
            Print / Save as PDF
          </button>
        </div>
        <div class="grid grid-cols-1 sm:grid-cols-5 gap-4">
          <div v-for="(t, idx) in sortedSubstituteStats.slice(0, 5)" :key="t.name" class="p-4 bg-slate-50 border border-slate-100 rounded-2xl flex flex-col justify-between">
            <div class="text-xs font-bold text-slate-400">Rank #{{ idx + 1 }}</div>
            <div class="my-2">
              <div class="text-sm font-extrabold text-slate-900">{{ t.name }}</div>
              <div class="text-[11px] text-slate-500 font-medium">{{ t.subject || 'General Subject' }}</div>
            </div>
            <div class="text-xs font-bold text-indigo-600 bg-white px-3 py-1.5 rounded-xl shadow-sm text-center border border-slate-200">
              {{ t.count }} substituted
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- TAB 2: 项目分析 -->
    <div v-if="currentTab === 'reason'" class="bg-white p-8 rounded-3xl shadow-sm ring-1 ring-slate-900/5 space-y-6 animate-fadeIn">
      <div class="flex justify-between items-center">
        <div>
          <h2 class="text-lg font-bold text-slate-900 flex items-center gap-2">
            <TriangleAlert class="w-5 h-5 text-orange-600" />
            INTERRUPTION CATEGORY STATISTICS
          </h2>
          <p class="text-xs text-slate-500 mt-1 font-medium">Categorized by personal leave, official duty, internal tasks, and historical data.</p>
        </div>
        <button @click="exportSinglePdf" class="no-print px-4 py-2 bg-slate-100 hover:bg-slate-200 text-slate-700 rounded-xl text-xs font-bold transition flex items-center gap-1.5 cursor-pointer">
          <Printer class="w-4 h-4" />
          Print / Save as PDF
        </button>
      </div>
      
      <div v-if="groupedReasonStats.length === 0" class="text-xs text-slate-400 py-12 text-center border border-dashed rounded-2xl font-medium">No interruption records found in this period.</div>
      
      <div v-else class="grid grid-cols-1 lg:grid-cols-2 gap-6">
        <div v-for="group in groupedReasonStats" :key="group.id" class="bg-slate-50 p-6 rounded-2xl border border-slate-100 flex flex-col hover:shadow-sm transition-shadow">
          <div class="flex justify-between items-center mb-5 pb-4 border-b border-slate-200/80">
            <h3 class="text-sm font-extrabold text-slate-800 flex items-center gap-2">
              <component :is="group.iconComponent" class="w-4 h-4 text-slate-600" />
              {{ group.title }}
            </h3>
            <span class="text-[11px] font-bold px-3 py-1 rounded-full shadow-sm" :class="group.badgeClass">
              Total {{ group.total }} Slots
            </span>
          </div>
          
          <div class="space-y-4 flex-1">
            <div v-for="item in group.items" :key="item.reason" class="space-y-1.5">
              <div class="flex justify-between text-xs font-bold text-slate-700">
                <span class="truncate pr-4" :title="item.reason">{{ item.reason }}</span>
                <span class="whitespace-nowrap">{{ item.count }} Slots ({{ item.percentage }}%)</span>
              </div>
              <div class="w-full h-2 bg-slate-200 rounded-full overflow-hidden">
                <div class="h-full rounded-full transition-all duration-500" :class="group.barClass" :style="{ width: item.percentage + '%' }"></div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- TAB 3: 日期高峰 -->
    <div v-if="currentTab === 'trend'" class="bg-white p-8 rounded-3xl shadow-sm ring-1 ring-slate-900/5 space-y-6 animate-fadeIn">
      <div class="flex justify-between items-center">
        <div>
          <h2 class="text-lg font-bold text-slate-900 flex items-center gap-2">
            <CalendarDays class="w-5 h-5 text-indigo-600" />
            INTERRUPTION PEAK DATES
          </h2>
          <p class="text-xs text-slate-500 mt-1 font-medium">Frequency of teaching interruptions grouped by days of the week.</p>
        </div>
        <button @click="exportSinglePdf" class="no-print px-4 py-2 bg-slate-100 hover:bg-slate-200 text-slate-700 rounded-xl text-xs font-bold transition flex items-center gap-1.5 cursor-pointer">
          <Printer class="w-4 h-4" />
          Print / Save as PDF
        </button>
      </div>
      <div class="grid grid-cols-1 sm:grid-cols-5 gap-4">
        <div v-for="dayData in dayOfWeekStats" :key="dayData.day" class="p-5 bg-slate-50 border border-slate-100 rounded-2xl text-center space-y-2">
          <div class="text-xs font-bold text-slate-500 uppercase">{{ dayData.day }}</div>
          <div class="text-2xl font-black text-slate-900">{{ dayData.count }} <span class="text-xs font-normal text-slate-400">Slots</span></div>
          <div class="text-[11px] text-indigo-600 font-semibold bg-indigo-50 py-1 rounded-lg">Accounts for {{ dayData.percentage }}%</div>
        </div>
      </div>
    </div>

    <!-- TAB 4: 班级干扰分析 -->
    <div v-if="currentTab === 'class'" class="bg-white p-8 rounded-3xl shadow-sm ring-1 ring-slate-900/5 space-y-6 animate-fadeIn">
      <div class="flex flex-col sm:flex-row justify-between items-start sm:items-center gap-4">
        <div>
          <h2 class="text-lg font-bold text-slate-900 flex items-center gap-2">
            <School class="w-5 h-5 text-indigo-600" />
            CLASS ANALYSIS
          </h2>
          <p class="text-xs text-slate-500 mt-1 font-medium">Filter by grade and analyze the accumulated interrupted periods for each class.</p>
        </div>
        <button @click="exportSinglePdf" class="no-print px-4 py-2 bg-slate-100 hover:bg-slate-200 text-slate-700 rounded-xl text-xs font-bold transition flex items-center gap-1.5 cursor-pointer shrink-0">
          <Printer class="w-4 h-4" />
          Print / Save as PDF
        </button>
      </div>

      <!-- 年级筛选器 -->
      <div class="no-print bg-slate-50 p-4 rounded-2xl border border-slate-100 flex flex-wrap items-center gap-4">
        <div class="flex items-center gap-2">
          <span class="text-xs font-bold text-slate-600 flex items-center gap-1.5">
            <Filter class="w-4 h-4" />
            Grade Filter:
          </span>
          <select 
            v-model="selectedClassGradeFilter" 
            class="bg-white border border-slate-200 rounded-xl px-3 py-2 text-xs font-bold text-slate-800 outline-none cursor-pointer"
          >
            <option value="">All Grades</option>
            <option v-for="g in availableClassGrades" :key="g" :value="g">{{ g }}</option>
          </select>
        </div>

        <button 
          v-if="selectedClassGradeFilter" 
          @click="selectedClassGradeFilter = ''" 
          class="text-xs font-bold text-indigo-600 hover:text-indigo-800 underline cursor-pointer ml-auto"
        >
          Clear Filter
        </button>
      </div>

      <div class="overflow-x-auto">
        <table class="w-full text-left text-xs border-collapse print-table">
          <thead>
            <tr class="bg-slate-50 text-slate-500 uppercase tracking-wider select-none font-semibold">
              <th @click="sortClassTable('className')" class="p-4 border-b cursor-pointer hover:bg-slate-100 transition">
                Class Name <span class="text-indigo-600">{{ classSortKey === 'className' ? (classSortAsc ? '▲' : '▼') : '↕' }}</span>
              </th>
              <th @click="sortClassTable('totalPeriods')" class="p-4 border-b cursor-pointer hover:bg-slate-100 transition">
                Total Interrupted Periods <span class="text-indigo-600">{{ classSortKey === 'totalPeriods' ? (classSortAsc ? '▲' : '▼') : '↕' }}</span>
              </th>
              <th @click="sortClassTable('percentage')" class="p-4 border-b cursor-pointer hover:bg-slate-100 transition">
                Percentage <span class="text-indigo-600">{{ classSortKey === 'percentage' ? (classSortAsc ? '▲' : '▼') : '↕' }}</span>
              </th>
            </tr>
          </thead>
          <tbody class="divide-y divide-slate-100 font-medium text-slate-800">
            <tr v-if="filteredClassStats.length === 0">
              <td colspan="3" class="p-8 text-center text-slate-400 font-medium">No records found for the selected filter.</td>
            </tr>
            <tr v-for="c in filteredClassStats" :key="c.className" class="hover:bg-slate-50">
              <td class="p-4 font-bold text-slate-900">{{ c.className }}</td>
              <td class="p-4 font-bold text-indigo-600">{{ c.totalPeriods }} Slots</td>
              <td class="p-4 text-slate-600">{{ c.percentage }}%</td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>

    <!-- TAB 5: 被影响最多的科目与班级明细排行 -->
    <div v-if="currentTab === 'subject'" class="bg-white p-8 rounded-3xl shadow-sm ring-1 ring-slate-900/5 space-y-6 animate-fadeIn">
      <div class="flex flex-col sm:flex-row justify-between items-start sm:items-center gap-4">
        <div>
          <h2 class="text-lg font-bold text-slate-900 flex items-center gap-2">
            <BookOpen class="w-5 h-5 text-indigo-600" />
            DETAILED SUBJECT ANALYSIS
          </h2>
          <p class="text-xs text-slate-500 mt-1 font-medium">Filter by grade and class with multi-dimensional tracking (integrating precise leave slots and school activities).</p>
        </div>
        <button @click="exportSinglePdf" class="no-print px-4 py-2 bg-slate-100 hover:bg-slate-200 text-slate-700 rounded-xl text-xs font-bold transition flex items-center gap-1.5 cursor-pointer shrink-0">
          <Printer class="w-4 h-4" />
          Print / Save as PDF
        </button>
      </div>

      <div class="no-print bg-slate-50 p-4 rounded-2xl border border-slate-100 flex flex-wrap items-center gap-4">
        <div class="flex items-center gap-2">
          <span class="text-xs font-bold text-slate-600 flex items-center gap-1.5">
            <Filter class="w-4 h-4" />
            Grade Filter:
          </span>
          <select 
            v-model="selectedGradeFilter" 
            @change="selectedClassFilter = ''"
            class="bg-white border border-slate-200 rounded-xl px-3 py-2 text-xs font-bold text-slate-800 outline-none cursor-pointer"
          >
            <option value="">All Grades</option>
            <option v-for="g in availableGrades" :key="g" :value="g">{{ g }}</option>
          </select>
        </div>

        <div class="flex items-center gap-2">
          <span class="text-xs font-bold text-slate-600 flex items-center gap-1.5">
            <Filter class="w-4 h-4" />
            Class Filter:
          </span>
          <select 
            v-model="selectedClassFilter" 
            class="bg-white border border-slate-200 rounded-xl px-3 py-2 text-xs font-bold text-slate-800 outline-none cursor-pointer"
          >
            <option value="">All Classes</option>
            <option v-for="cls in availableClassesForFilter" :key="cls" :value="cls">{{ cls }}</option>
          </select>
        </div>

        <button 
          v-if="selectedGradeFilter || selectedClassFilter" 
          @click="selectedGradeFilter = ''; selectedClassFilter = ''" 
          class="text-xs font-bold text-indigo-600 hover:text-indigo-800 underline cursor-pointer ml-auto"
        >
          Clear Filter
        </button>
      </div>

      <div class="overflow-x-auto">
        <table class="w-full text-left text-xs border-collapse print-table">
          <thead>
            <tr class="bg-slate-50 text-slate-500 uppercase tracking-wider select-none font-semibold">
              <th @click="sortSubjectTable('grade')" class="p-4 border-b cursor-pointer hover:bg-slate-100 transition">
                Grade <span class="text-indigo-600">{{ subjectSortKey === 'grade' ? (subjectSortAsc ? '▲' : '▼') : '↕' }}</span>
              </th>
              <th @click="sortSubjectTable('className')" class="p-4 border-b cursor-pointer hover:bg-slate-100 transition">
                Class <span class="text-indigo-600">{{ subjectSortKey === 'className' ? (subjectSortAsc ? '▲' : '▼') : '↕' }}</span>
              </th>
              <th @click="sortSubjectTable('subjectName')" class="p-4 border-b cursor-pointer hover:bg-slate-100 transition">
                Affected Subject <span class="text-indigo-600">{{ subjectSortKey === 'subjectName' ? (subjectSortAsc ? '▲' : '▼') : '↕' }}</span>
              </th>
              <th @click="sortSubjectTable('totalPeriods')" class="p-4 border-b cursor-pointer hover:bg-slate-100 transition">
                Total Interrupted Periods <span class="text-indigo-600">{{ subjectSortKey === 'totalPeriods' ? (subjectSortAsc ? '▲' : '▼') : '↕' }}</span>
              </th>
            </tr>
          </thead>
          <tbody class="divide-y divide-slate-100 font-medium text-slate-800">
            <tr v-if="filteredSubjectStats.length === 0">
              <td colspan="4" class="p-8 text-center text-slate-400 font-medium">No records found for the selected filter.</td>
            </tr>
            <tr v-for="s in filteredSubjectStats" :key="s.id" class="hover:bg-slate-50">
              <td class="p-4 font-bold text-slate-600">{{ s.grade }}</td>
              <td class="p-4 font-bold text-slate-900">{{ s.className }}</td>
              <td class="p-4 font-bold text-indigo-600">{{ s.subjectName }}</td>
              <td class="p-4 font-bold text-amber-600">{{ s.totalPeriods }} Slots</td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>

    <!-- TAB 7: 老师干扰总表 -->
    <div v-if="currentTab === 'teacher'" class="bg-white p-8 rounded-3xl shadow-sm ring-1 ring-slate-900/5 space-y-6 animate-fadeIn">
      <div class="flex justify-between items-center">
        <div>
          <h2 class="text-lg font-bold text-slate-900 flex items-center gap-2">
            <UsersRound class="w-5 h-5 text-indigo-600" />
            TEACHER OVERVIEW
          </h2>
          <p class="text-xs text-slate-500 mt-1 font-medium">Displays statistics for all registered teachers, substitute loads, and interrupted class periods.</p>
        </div>
        <button @click="exportSinglePdf" class="no-print px-4 py-2 bg-slate-100 hover:bg-slate-200 text-slate-700 rounded-xl text-xs font-bold transition flex items-center gap-1.5 cursor-pointer">
          <Printer class="w-4 h-4" />
          Print / Save as PDF
        </button>
      </div>

      <div class="overflow-x-auto">
        <table class="w-full text-left text-xs border-collapse print-table">
          <thead>
            <tr class="bg-slate-50 text-slate-500 uppercase tracking-wider select-none font-semibold">
              <th @click="sortTeacherTable('name')" class="p-4 border-b cursor-pointer hover:bg-slate-100 transition">
                Teacher Name <span class="text-indigo-600">{{ teacherSortKey === 'name' ? (teacherSortAsc ? '▲' : '▼') : '↕' }}</span>
              </th>
              <th @click="sortTeacherTable('subject')" class="p-4 border-b cursor-pointer hover:bg-slate-100 transition">
                Teaching Subject <span class="text-indigo-600">{{ teacherSortKey === 'subject' ? (teacherSortAsc ? '▲' : '▼') : '↕' }}</span>
              </th>
              <th @click="sortTeacherTable('count')" class="p-4 border-b cursor-pointer hover:bg-slate-100 transition">
                Substitute Periods <span class="text-indigo-600">{{ teacherSortKey === 'count' ? (teacherSortAsc ? '▲' : '▼') : '↕' }}</span>
              </th>
              <th @click="sortTeacherTable('interruptedCount')" class="p-4 border-b cursor-pointer hover:bg-slate-100 transition">
                Interrupted Periods <span class="text-indigo-600">{{ teacherSortKey === 'interruptedCount' ? (teacherSortAsc ? '▲' : '▼') : '↕' }}</span>
              </th>
            </tr>
          </thead>
          <tbody class="divide-y divide-slate-100 font-medium text-slate-800">
            <tr v-for="stat in sortedTeacherStats" :key="stat.name" class="hover:bg-slate-50">
              <td class="p-4 font-bold text-slate-900">{{ stat.name }}</td>
              <td class="p-4 text-slate-600">{{ stat.subject || '-' }}</td>
              <td class="p-4 font-bold text-indigo-600">{{ stat.count }} Slots</td>
              <td class="p-4 font-bold text-amber-600">{{ stat.interruptedCount }} Slots</td>
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
import { 
  ChartNoAxesCombined, 
  CalendarDays, 
  RefreshCw, 
  LayoutDashboard, 
  TriangleAlert, 
  School, 
  BookOpen, 
  UsersRound, 
  Clock3, 
  ArrowLeftRight, 
  FileText, 
  Scale, 
  Printer, 
  Filter,
  CalendarCheck,
  BriefcaseBusiness,
  Building2,
  FolderOpen
} from 'lucide-vue-next'

const currentTab = ref('overview')

const startDate = ref('')
const endDate = ref('')

const stats = ref([])
const interruptionLogs = ref([])
const reasonStats = ref([])
const dayOfWeekStats = ref([])
const classStats = ref([])
const subjectStats = ref([])

const selectedClassGradeFilter = ref('')
const selectedGradeFilter = ref('')
const selectedClassFilter = ref('')

const gradeOrderMap = {
  'Year 1': 1,
  'Year 2': 2,
  'Year 3': 3,
  'Year 4': 4,
  'Year 5': 5,
  'Year 6': 6
};

const sortGrgradesHelper = (setObj) => {
  return Array.from(setObj).sort((a, b) => {
    const wA = gradeOrderMap[a] !== undefined ? gradeOrderMap[a] : 99;
    const wB = gradeOrderMap[b] !== undefined ? gradeOrderMap[b] : 99;
    return wA - wB;
  });
};

const getGradeFromClass = (cName) => {
  if (!cName) return 'Others';
  const match = cName.match(/^([0-9]+)/);
  if (match) {
    const gradeNum = match[1];
    const gradeMap = {
      '1': 'Year 1',
      '2': 'Year 2',
      '3': 'Year 3',
      '4': 'Year 4',
      '5': 'Year 5',
      '6': 'Year 6'
    };
    return gradeMap[gradeNum] || `Year ${gradeNum}`;
  }
  return 'Whole School / Others';
};

const availableClassGrades = computed(() => {
  const grades = new Set()
  classStats.value.forEach(c => {
    const g = getGradeFromClass(c.className)
    if (g && g !== 'Whole School / Others') grades.add(g)
  })
  return sortGrgradesHelper(grades)
})

const filteredClassStats = computed(() => {
  let list = classStats.value
  if (selectedClassGradeFilter.value) {
    list = list.filter(c => getGradeFromClass(c.className) === selectedClassGradeFilter.value)
  }
  return [...list].sort((a, b) => smartSort(a[classSortKey.value], b[classSortKey.value], classSortAsc.value))
})

const availableGrades = computed(() => {
  const grades = new Set()
  subjectStats.value.forEach(s => {
    if (s.grade && s.grade !== 'Whole School / Others') grades.add(s.grade)
  })
  return sortGrgradesHelper(grades)
})

const availableClassesForFilter = computed(() => {
  const classes = new Set()
  subjectStats.value.forEach(s => {
    if (!selectedGradeFilter.value || s.grade === selectedGradeFilter.value) {
      if (s.className && s.className !== 'UNKNOWN') classes.add(s.className)
    }
  })
  return Array.from(classes).sort()
})

const filteredSubjectStats = computed(() => {
  let list = subjectStats.value
  if (selectedGradeFilter.value) {
    list = list.filter(s => s.grade === selectedGradeFilter.value)
  }
  if (selectedClassFilter.value) {
    list = list.filter(s => s.className === selectedClassFilter.value)
  }
  return [...list].sort((a, b) => smartSort(a[subjectSortKey.value], b[subjectSortKey.value], subjectSortAsc.value))
})

const groupedReasonStats = computed(() => {
  if (!reasonStats.value.length) return [];

  const groups = {
    personal: { id: 'personal', title: 'Personal Leave', iconComponent: CalendarCheck, items: [], total: 0, badgeClass: 'bg-orange-100 text-orange-700', barClass: 'bg-orange-500' },
    official: { id: 'official', title: 'Official Duty', iconComponent: BriefcaseBusiness, items: [], total: 0, badgeClass: 'bg-blue-100 text-blue-700', barClass: 'bg-blue-500' },
    internal: { id: 'internal', title: 'Internal Task', iconComponent: Building2, items: [], total: 0, badgeClass: 'bg-emerald-100 text-emerald-700', barClass: 'bg-emerald-500' },
    others:   { id: 'others', title: 'History / Others', iconComponent: FolderOpen, items: [], total: 0, badgeClass: 'bg-slate-200 text-slate-700', barClass: 'bg-slate-400' }
  };

  const totalPAll = reasonStats.value.reduce((acc, cur) => acc + cur.count, 0);

  reasonStats.value.forEach(item => {
    let cleanReason = item.reason.replace(/\[.*?\]\s*/, '');
    let targetGroup = 'others';

    // 🌟 兼容中英马三语历史标签
    if (item.reason.includes('[PERSONAL LEAVE]') || item.reason.includes('[个人请假]') || item.reason.includes('[CUTI PERIBADI]')) targetGroup = 'personal';
    else if (item.reason.includes('[OFFICIAL DUTY]') || item.reason.includes('[离校公干]') || item.reason.includes('[TUGAS RASMI]')) targetGroup = 'official';
    else if (item.reason.includes('[INTERNAL TASK]') || item.reason.includes('[校内任务]') || item.reason.includes('[TUGAS DALAMAN]')) targetGroup = 'internal';

    groups[targetGroup].items.push({
      reason: cleanReason,
      count: item.count,
      percentage: totalPAll > 0 ? ((item.count / totalPAll) * 100).toFixed(1) : 0
    });
    groups[targetGroup].total += item.count;
  });

  Object.values(groups).forEach(g => {
    if (g.items.length > 8) {
      const top8 = g.items.slice(0, 8);
      const remaining = g.items.slice(8);
      const remainingCount = remaining.reduce((sum, r) => sum + r.count, 0);
      
      top8.push({
        reason: 'Others (Merged)',
        count: remainingCount,
        percentage: totalPAll > 0 ? ((remainingCount / totalPAll) * 100).toFixed(1) : 0
      });
      g.items = top8;
    }
  });

  return Object.values(groups)
    .filter(g => g.total > 0)
    .sort((a, b) => b.total - a.total);
});

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

const classSortKey = ref('totalPeriods')
const classSortAsc = ref(false)
const sortClassTable = (key) => {
  if (classSortKey.value === key) classSortAsc.value = !classSortAsc.value
  else { classSortKey.value = key; classSortAsc.value = true }
}

const subjectSortKey = ref('totalPeriods')
const subjectSortAsc = ref(false)
const sortSubjectTable = (key) => {
  if (subjectSortKey.value === key) subjectSortAsc.value = !subjectSortAsc.value
  else { subjectSortKey.value = key; subjectSortAsc.value = true }
}

const teacherSortKey = ref('count')
const teacherSortAsc = ref(false)
const sortTeacherTable = (key) => {
  if (teacherSortKey.value === key) teacherSortAsc.value = !teacherSortAsc.value
  else { teacherSortKey.value = key; teacherSortAsc.value = true }
}
const sortedTeacherStats = computed(() => [...stats.value].sort((a, b) => smartSort(a[teacherSortKey.value], b[teacherSortKey.value], teacherSortAsc.value)))

const totalSubstituteCount = computed(() => stats.value.reduce((acc, cur) => acc + (cur.count || 0), 0))
const totalInterruptionPeriods = computed(() => interruptionLogs.value.reduce((acc, cur) => acc + ((cur.end_period || 0) - (cur.start_period || 0) + 1), 0))
const sortedSubstituteStats = computed(() => [...stats.value].sort((a, b) => (b.count || 0) - (a.count || 0)))

const resetDateFilter = () => {
  startDate.value = ''
  endDate.value = ''
  selectedClassGradeFilter.value = ''
  selectedGradeFilter.value = ''
  selectedClassFilter.value = ''
  loadAllData()
}

const cleanClassName = (rawStr) => {
  if (!rawStr) return '';
  let cleaned = rawStr.replace(/^(班级|班級|KELAS|CLASS)\s*[:：]\s*/i, '').trim();
  cleaned = cleaned.replace(/^(班级|班級|KELAS|CLASS)\s*[:：]\s*/i, '').trim();
  if (!cleaned || /VIRTUAL_CLASS/i.test(cleaned)) return '';
  return cleaned.toUpperCase();
};

const loadAllData = async () => {
  const { data: teachers } = await supabase.from('teachers').select('*')
  
  let assignQuery = supabase
    .from('substitute_assignments')
    .select('sub_teacher_id, assignment_type, leave_request_id, leave_requests!inner(leave_date)')

  if (startDate.value) assignQuery = assignQuery.gte('leave_requests.leave_date', startDate.value)
  if (endDate.value) assignQuery = assignQuery.lte('leave_requests.leave_date', endDate.value)

  const { data: assignments } = await assignQuery

  const swapLeaveIds = new Set()
  assignments?.forEach(a => {
    if (a.assignment_type === 'swap' && a.leave_request_id) {
      swapLeaveIds.add(a.leave_request_id)
    }
  })

  let mmiQuery = supabase.from('mmi_interruptions').select('*')
  if (startDate.value) mmiQuery = mmiQuery.gte('interruption_date', startDate.value)
  if (endDate.value) mmiQuery = mmiQuery.lte('interruption_date', endDate.value)
  const { data: mmiData } = await mmiQuery

  if (mmiData) interruptionLogs.value = mmiData

  let leaveQuery = supabase.from('leave_requests').select('*')
  if (startDate.value) leaveQuery = leaveQuery.gte('leave_date', startDate.value)
  if (endDate.value) leaveQuery = leaveQuery.lte('leave_date', endDate.value)
  const { data: leaveData } = await leaveQuery

  const { data: timetables } = await supabase.from('timetable').select('*')

  const teacherMap = {}
  const teacherNameSet = new Set()
  
  teachers?.forEach(t => {
    teacherMap[t.id] = { name: t.name, subject: t.subject, count: 0, interruptedCount: 0 }
    teacherNameSet.add(t.name.trim().toUpperCase())
  })

  assignments?.forEach(a => {
    if (a.assignment_type !== 'swap' && a.sub_teacher_id && teacherMap[a.sub_teacher_id]) {
      teacherMap[a.sub_teacher_id].count++
    }
  })

  const teacherInterruptionMap = {}
  mmiData?.forEach(l => {
    let rawTarget = (l.target_display || '').trim()
    let tName = ''
    if (rawTarget.includes('教师:')) tName = rawTarget.replace('教师:', '').trim()
    else if (rawTarget.includes('GURU:')) tName = rawTarget.replace('GURU:', '').trim()
    else if (rawTarget.includes('TEACHER:')) tName = rawTarget.replace('TEACHER:', '').trim()
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

  if (mmiData) {
    const totalPAll = mmiData.reduce((acc, cur) => acc + ((cur.end_period || 0) - (cur.start_period || 0) + 1), 0)
    const reasons = {}
    mmiData.forEach(l => { 
      const pCount = (l.end_period || 0) - (l.start_period || 0) + 1; 
      let rawReason = (l.reason || 'No Data').trim().toUpperCase();
      rawReason = rawReason.replace(/^(教师请假:\s*|CUTI GURU:\s*|TEACHER LEAVE:\s*)/i, ''); 
      reasons[rawReason] = (reasons[rawReason] || 0) + pCount;
    })
    
    reasonStats.value = Object.entries(reasons)
      .map(([reason, count]) => ({ reason, count, percentage: totalPAll > 0 ? ((count / totalPAll) * 100).toFixed(1) : 0 }))
      .sort((a, b) => b.count - a.count)

    const dayNames = { 1: 'Monday', 2: 'Tuesday', 3: 'Wednesday', 4: 'Thursday', 5: 'Friday', 6: 'Saturday', 7: 'Sunday' }
    const daysCount = {}
    mmiData.forEach(l => { 
      const dIndex = new Date(l.interruption_date).getDay() || 7; 
      const dName = dayNames[dIndex] || 'Others'; 
      const pCount = (l.end_period || 0) - (l.start_period || 0) + 1; 
      daysCount[dName] = (daysCount[dName] || 0) + pCount 
    })
    dayOfWeekStats.value = ['Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday'].map(day => ({ 
      day, 
      count: daysCount[day] || 0, 
      percentage: totalPAll > 0 ? (((daysCount[day] || 0) / totalPAll) * 100).toFixed(1) : 0 
    }))
  }

  const classMap = {}
  const subjectDetailMap = {} 
  let totalClassPeriods = 0

  const processClassNames = (rawStr, pCount) => {
    if (!rawStr) return;
    let cleaned = rawStr.replace(/^(班级|班級|KELAS|CLASS)\s*[:：]\s*/i, '').trim();
    if (!cleaned || /VIRTUAL_CLASS/i.test(cleaned)) return;

    const separators = /,|、|\//;
    if (separators.test(cleaned)) {
      const parts = cleaned.split(separators);
      parts.forEach(p => {
        let subClean = cleanClassName(p);
        if (subClean && subClean !== 'VIRTUAL_CLASS') {
          classMap[subClean] = (classMap[subClean] || 0) + pCount;
          totalClassPeriods += pCount;
        }
      });
    } else {
      const cName = cleanClassName(cleaned);
      if (cName && cName !== 'VIRTUAL_CLASS') {
        classMap[cName] = (classMap[cName] || 0) + pCount;
        totalClassPeriods += pCount;
      }
    }
  };

  mmiData?.forEach(l => { 
    let rawTarget = (l.target_display || '').trim(); 
    if (/^GURU:/i.test(rawTarget) || /^TEACHER:/i.test(rawTarget) || rawTarget.includes('教师') || teacherNameSet.has(rawTarget.toUpperCase()) || /VIRTUAL_CLASS/i.test(rawTarget)) return; 
    const pCount = (l.end_period || 0) - (l.start_period || 0) + 1; 
    processClassNames(rawTarget, pCount);
  })

  // A. 处理教师请假勾选节次带来的科目受干扰损失
  leaveData?.forEach(req => {
    if (swapLeaveIds.has(req.id)) return;

    processClassNames(req.class_name, 1);

    const sub = req.subject ? req.subject.trim().toUpperCase() : 'UNKNOWN';
    if (sub && sub !== 'UNKNOWN' && !sub.includes('VIRTUAL_SUB')) {
      const cNames = req.class_name ? req.class_name.split(/,|、|\//) : ['UNKNOWN'];
      cNames.forEach(c => {
        const cleanC = cleanClassName(c) || 'UNKNOWN';
        const grade = getGradeFromClass(cleanC);
        
        const compositeKey = `${grade}_${cleanC}_${sub}`;
        if (!subjectDetailMap[compositeKey]) {
          subjectDetailMap[compositeKey] = {
            id: compositeKey,
            grade: grade,
            className: cleanC,
            subjectName: sub,
            totalPeriods: 0
          };
        }
        subjectDetailMap[compositeKey].totalPeriods += 1;
      });
    }
  })

  // B. 处理全校性/跨班级活动冲击损失匹配
  mmiData?.forEach(int => {
    if (int.type === 'class' && timetables && timetables.length > 0) {
      const startP = Number(int.start_period) || 1;
      const endP = Number(int.end_period) || 1;
      const targetDisp = (int.target_display || '').trim();

      const intDate = new Date(int.interruption_date);
      const wd = intDate.getDay();
      const weekdayNum = wd === 0 ? 7 : wd;

      timetables.forEach(t => {
        if (Number(t.weekday) !== weekdayNum) return;
        const p = Number(t.period);
        if (p < startP || p > endP) return;

        const cName = cleanClassName(t.class_name);
        if (!cName || cName === 'VIRTUAL_CLASS') return;

        let isAffected = false;
        if (targetDisp.includes('全校') || targetDisp.includes('SELURUH SEKOLAH') || targetDisp.includes('ALL CLASSES')) {
          isAffected = true;
        } else if (targetDisp.includes('全年级') || targetDisp.includes('Tahun') || targetDisp.includes('TAHUN') || targetDisp.includes('Year') || targetDisp.includes('YEAR')) {
          const match = targetDisp.match(/Tahun\s*(\d)/i) || targetDisp.match(/(\d)\s*年级/) || targetDisp.match(/Year\s*(\d)/i);
          const gradeNum = match ? match[1] : null;
          const cGrade = getGradeFromClass(cName);
          if (gradeNum && (cName.startsWith(gradeNum) || cGrade.includes(gradeNum))) {
            isAffected = true;
          } else if (!gradeNum) {
            isAffected = true;
          }
        } else {
          const targetList = targetDisp.split(/,|、|\//).map(s => cleanClassName(s));
          isAffected = targetList.some(tc => tc && (cName === tc || cName.includes(tc) || tc.includes(cName)));
        }

        if (isAffected) {
          const sub = t.subject ? t.subject.trim().toUpperCase() : 'UNKNOWN';
          if (sub && sub !== 'UNKNOWN' && !sub.includes('VIRTUAL_SUB')) {
            const grade = getGradeFromClass(cName);
            const compositeKey = `${grade}_${cName}_${sub}`;
            if (!subjectDetailMap[compositeKey]) {
              subjectDetailMap[compositeKey] = {
                id: compositeKey,
                grade: grade,
                className: cName,
                subjectName: sub,
                totalPeriods: 0
              };
            }
            subjectDetailMap[compositeKey].totalPeriods += 1;
          }
        }
      });
    }
  });

  classStats.value = Object.entries(classMap)
    .map(([className, totalPeriods]) => ({ 
      className, 
      totalPeriods, 
      percentage: totalClassPeriods > 0 ? ((totalPeriods / totalClassPeriods) * 100).toFixed(1) : 0 
    }))
    .sort((a, b) => b.totalPeriods - a.totalPeriods)

  subjectStats.value = Object.values(subjectDetailMap)
    .sort((a, b) => b.totalPeriods - a.totalPeriods)
}

onMounted(loadAllData)

onActivated(() => {
  loadAllData()
})

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