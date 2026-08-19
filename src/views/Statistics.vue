<template>
  <div class="p-4 sm:p-8 mx-auto min-h-screen space-y-8 min-w-[1024px] print:p-0 print:max-w-none print:min-w-0">
    
    <!-- Top Header: Unified card style and gradient title -->
    <div class="no-print bg-white rounded-3xl p-6 sm:p-8 shadow-sm ring-1 ring-slate-900/5 space-y-2">
      <h1 class="text-2xl sm:text-3xl font-extrabold tracking-tight text-transparent bg-clip-text bg-gradient-to-r from-slate-900 via-indigo-800 to-violet-800 flex items-center gap-3">
        <ChartNoAxesCombined class="w-8 h-8 text-indigo-700 shrink-0" />
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
          <CalendarDays class="w-5 h-5" />
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
          class="px-5 h-11 bg-slate-100 hover:bg-slate-200 text-slate-700 rounded-2xl text-xs font-bold transition flex items-center gap-2 cursor-pointer"
        >
          <RefreshCw class="w-4 h-4" />
          RESET
        </button>
      </div>
    </div>

    <!-- Top Bar: Analysis Perspective Switcher (Tabs) -->
    <div class="no-print flex flex-wrap gap-2 bg-white p-3 rounded-3xl shadow-sm ring-1 ring-slate-900/5">
      <button 
        @click="currentTab = 'overview'" 
        :class="currentTab === 'overview' ? 'bg-slate-900 text-white shadow-sm' : 'text-slate-600 hover:bg-slate-100'"
        class="px-4 py-2.5 rounded-2xl text-xs font-bold transition-all cursor-pointer whitespace-nowrap flex items-center gap-2"
      >
        <LayoutDashboard class="w-4 h-4" />
        OVERVIEW & SUBSTITUTE WORKLOAD
      </button>
      <button 
        @click="currentTab = 'reason'" 
        :class="currentTab === 'reason' ? 'bg-slate-900 text-white shadow-sm' : 'text-slate-600 hover:bg-slate-100'"
        class="px-4 py-2.5 rounded-2xl text-xs font-bold transition-all cursor-pointer whitespace-nowrap flex items-center gap-2"
      >
        <TriangleAlert class="w-4 h-4" />
        INTERRUPTION REASON ANALYSIS
      </button>
      <button 
        @click="currentTab = 'trend'" 
        :class="currentTab === 'trend' ? 'bg-slate-900 text-white shadow-sm' : 'text-slate-600 hover:bg-slate-100'"
        class="px-4 py-2.5 rounded-2xl text-xs font-bold transition-all cursor-pointer whitespace-nowrap flex items-center gap-2"
      >
        <CalendarDays class="w-4 h-4" />
        INTERRUPTION DATE PEAKS
      </button>
      <button 
        @click="currentTab = 'class'" 
        :class="currentTab === 'class' ? 'bg-slate-900 text-white shadow-sm' : 'text-slate-600 hover:bg-slate-100'"
        class="px-4 py-2.5 rounded-2xl text-xs font-bold transition-all cursor-pointer whitespace-nowrap flex items-center gap-2"
      >
        <School class="w-4 h-4" />
        AFFECTED CLASSES ANALYSIS
      </button>
      <button 
        @click="currentTab = 'subject'" 
        :class="currentTab === 'subject' ? 'bg-slate-900 text-white shadow-sm' : 'text-slate-600 hover:bg-slate-100'"
        class="px-4 py-2.5 rounded-2xl text-xs font-bold transition-all cursor-pointer whitespace-nowrap flex items-center gap-2"
      >
        <BookOpen class="w-4 h-4" />
        AFFECTED SUBJECTS & CLASSES
      </button>
      <button 
        @click="currentTab = 'teacher'" 
        :class="currentTab === 'teacher' ? 'bg-slate-900 text-white shadow-sm' : 'text-slate-600 hover:bg-slate-100'"
        class="px-4 py-2.5 rounded-2xl text-xs font-bold transition-all cursor-pointer whitespace-nowrap flex items-center gap-2"
      >
        <UsersRound class="w-4 h-4" />
        OVERALL TEACHER RECORDS
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
          <div class="text-xs font-bold text-slate-400 uppercase tracking-wider flex items-center gap-2">
            <Clock3 class="w-4 h-4 text-slate-400" />
            TOTAL INTERRUPTION SLOTS
          </div>
          <div class="text-3xl font-black text-slate-900 mt-2">{{ totalInterruptionPeriods }} SLOTS</div>
        </div>
        <div class="bg-white p-6 rounded-3xl shadow-sm ring-1 ring-slate-900/5">
          <div class="text-xs font-bold text-slate-400 uppercase tracking-wider flex items-center gap-2">
            <ArrowLeftRight class="w-4 h-4 text-indigo-600" />
            TOTAL SUBSTITUTE CASES
          </div>
          <div class="text-3xl font-black text-indigo-600 mt-2">{{ totalSubstituteCount }} TIMES</div>
        </div>
        <div class="bg-white p-6 rounded-3xl shadow-sm ring-1 ring-slate-900/5">
          <div class="text-xs font-bold text-slate-400 uppercase tracking-wider flex items-center gap-2">
            <FileText class="w-4 h-4 text-slate-400" />
            TOTAL INTERRUPTION RECORDS
          </div>
          <div class="text-3xl font-black text-slate-900 mt-2">{{ interruptionLogs.length }} RECORDS</div>
        </div>
      </div>

      <div class="bg-white p-8 rounded-3xl shadow-sm ring-1 ring-slate-900/5">
        <div class="flex justify-between items-center mb-6">
          <h2 class="text-base font-bold text-slate-900 flex items-center gap-2">
            <Scale class="w-5 h-5 text-indigo-600" />
            TOP 5 SUBSTITUTE WORKLOAD RANKING
          </h2>
          <button @click="exportSinglePdf" class="no-print px-4 py-2 bg-slate-100 hover:bg-slate-200 text-slate-700 rounded-xl text-xs font-bold transition flex items-center gap-1.5 cursor-pointer">
            <Printer class="w-4 h-4" />
            PRINT / SAVE AS PDF
          </button>
        </div>
        <div class="grid grid-cols-1 sm:grid-cols-5 gap-4">
          <div v-for="(t, idx) in sortedSubstituteStats.slice(0, 5)" :key="t.name" class="p-4 bg-slate-50 border border-slate-100 rounded-2xl flex flex-col justify-between">
            <div class="text-xs font-bold text-slate-400">RANK #{{ idx + 1 }}</div>
            <div class="my-2">
              <div class="text-sm font-extrabold text-slate-900">{{ t.name }}</div>
              <div class="text-[11px] text-slate-500 font-medium">{{ t.subject || 'GENERAL SUBJECT' }}</div>
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
          <h2 class="text-lg font-bold text-slate-900 flex items-center gap-2">
            <TriangleAlert class="w-5 h-5 text-orange-600" />
            TEACHING INTERRUPTION REASON STATISTICS
          </h2>
          <p class="text-xs text-slate-500 mt-1 font-medium">STATISTICS OF TIME SLOTS AND PERCENTAGES TAKEN BY EACH ACTIVITY.</p>
        </div>
        <button @click="exportSinglePdf" class="no-print px-4 py-2 bg-slate-100 hover:bg-slate-200 text-slate-700 rounded-xl text-xs font-bold transition flex items-center gap-1.5 cursor-pointer">
          <Printer class="w-4 h-4" />
          PRINT / SAVE AS PDF
        </button>
      </div>
      
      <div v-if="groupedReasonStats.length === 0" class="text-xs text-slate-400 py-12 text-center border border-dashed rounded-2xl font-medium">NO INTERRUPTION RECORDS IN THIS PERIOD</div>
      
      <div v-else class="grid grid-cols-1 lg:grid-cols-2 gap-6">
        <div v-for="group in groupedReasonStats" :key="group.id" class="bg-slate-50 p-6 rounded-2xl border border-slate-100 flex flex-col hover:shadow-sm transition-shadow">
          <div class="flex justify-between items-center mb-5 pb-4 border-b border-slate-200/80">
            <h3 class="text-sm font-extrabold text-slate-800 flex items-center gap-2">
              <component :is="group.iconComponent" class="w-4 h-4 text-slate-600" />
              {{ group.title }}
            </h3>
            <span class="text-[11px] font-bold px-3 py-1 rounded-full shadow-sm" :class="group.badgeClass">
              TOTAL {{ group.total }} SLOTS
            </span>
          </div>
          
          <div class="space-y-4 flex-1">
            <div v-for="item in group.items" :key="item.reason" class="space-y-1.5">
              <div class="flex justify-between text-xs font-bold text-slate-700">
                <span class="truncate pr-4" :title="item.reason">{{ item.reason }}</span>
                <span class="whitespace-nowrap">{{ item.count }} SLOTS ({{ item.percentage }}%)</span>
              </div>
              <div class="w-full h-2 bg-slate-200 rounded-full overflow-hidden">
                <div class="h-full rounded-full transition-all duration-500" :class="group.barClass" :style="{ width: item.percentage + '%' }"></div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- ================= TAB 3: Date Peaks ================= -->
    <div v-if="currentTab === 'trend'" class="bg-white p-8 rounded-3xl shadow-sm ring-1 ring-slate-900/5 space-y-6 animate-fadeIn">
      <div class="flex justify-between items-center">
        <div>
          <h2 class="text-lg font-bold text-slate-900 flex items-center gap-2">
            <CalendarDays class="w-5 h-5 text-indigo-600" />
            INTERRUPTION STATISTICS BY DAY
          </h2>
          <p class="text-xs text-slate-500 mt-1 font-medium">DISTRIBUTION OF INTERRUPTION FREQUENCY BY DAY OF THE WEEK.</p>
        </div>
        <button @click="exportSinglePdf" class="no-print px-4 py-2 bg-slate-100 hover:bg-slate-200 text-slate-700 rounded-xl text-xs font-bold transition flex items-center gap-1.5 cursor-pointer">
          <Printer class="w-4 h-4" />
          PRINT / SAVE AS PDF
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

    <!-- ================= TAB 4: Class Interruption Analysis (With Grade Filter) ================= -->
    <div v-if="currentTab === 'class'" class="bg-white p-8 rounded-3xl shadow-sm ring-1 ring-slate-900/5 space-y-6 animate-fadeIn">
      <div class="flex flex-col sm:flex-row justify-between items-start sm:items-center gap-4">
        <div>
          <h2 class="text-lg font-bold text-slate-900 flex items-center gap-2">
            <School class="w-5 h-5 text-indigo-600" />
            CLASS INTERRUPTION STATISTICS
          </h2>
          <p class="text-xs text-slate-500 mt-1 font-medium">FILTER BY YEAR/GRADE AND VIEW CUMULATIVE AFFECTED CLASS PERIODS.</p>
        </div>
        <button @click="exportSinglePdf" class="no-print px-4 py-2 bg-slate-100 hover:bg-slate-200 text-slate-700 rounded-xl text-xs font-bold transition flex items-center gap-1.5 cursor-pointer shrink-0">
          <Printer class="w-4 h-4" />
          PRINT / SAVE AS PDF
        </button>
      </div>

      <!-- Grade Filter Panel -->
      <div class="no-print bg-slate-50 p-4 rounded-2xl border border-slate-100 flex flex-wrap items-center gap-4">
        <div class="flex items-center gap-2">
          <span class="text-xs font-bold text-slate-600 flex items-center gap-1.5">
            <Filter class="w-4 h-4" />
            GRADE FILTER:
          </span>
          <select 
            v-model="selectedClassGradeFilter" 
            class="bg-white border border-slate-200 rounded-xl px-3 py-2 text-xs font-bold text-slate-800 outline-none cursor-pointer"
          >
            <option value="">ALL GRADES</option>
            <option v-for="g in availableClassGrades" :key="g" :value="g">{{ g }}</option>
          </select>
        </div>

        <button 
          v-if="selectedClassGradeFilter" 
          @click="selectedClassGradeFilter = ''" 
          class="text-xs font-bold text-indigo-600 hover:text-indigo-800 underline cursor-pointer ml-auto"
        >
          CLEAR FILTER
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
            <tr v-if="filteredClassStats.length === 0">
              <td colspan="3" class="p-8 text-center text-slate-400 font-medium">NO MATCHING CLASS RECORDS FOUND FOR THIS FILTER</td>
            </tr>
            <tr v-for="c in filteredClassStats" :key="c.className" class="hover:bg-slate-50">
              <td class="p-4 font-bold text-slate-900">{{ c.className }}</td>
              <td class="p-4 font-bold text-indigo-600">{{ c.totalPeriods }} SLOTS</td>
              <td class="p-4 text-slate-600">{{ c.percentage }}%</td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>

    <!-- ================= TAB 5: Affected Subjects & Classes Ranking (With Filters) ================= -->
    <div v-if="currentTab === 'subject'" class="bg-white p-8 rounded-3xl shadow-sm ring-1 ring-slate-900/5 space-y-6 animate-fadeIn">
      <div class="flex flex-col sm:flex-row justify-between items-start sm:items-center gap-4">
        <div>
          <h2 class="text-lg font-bold text-slate-900 flex items-center gap-2">
            <BookOpen class="w-5 h-5 text-indigo-600" />
            SUBJECT & CLASS INTERRUPTION DETAILS
          </h2>
          <p class="text-xs text-slate-500 mt-1 font-medium">MULTI-DIMENSIONAL FILTERING FOR SUBJECT INTERRUPTION DETAILS BY GRADE AND CLASS.</p>
        </div>
        <button @click="exportSinglePdf" class="no-print px-4 py-2 bg-slate-100 hover:bg-slate-200 text-slate-700 rounded-xl text-xs font-bold transition flex items-center gap-1.5 cursor-pointer shrink-0">
          <Printer class="w-4 h-4" />
          PRINT / SAVE AS PDF
        </button>
      </div>

      <!-- Grade & Class Linked Filter Panel -->
      <div class="no-print bg-slate-50 p-4 rounded-2xl border border-slate-100 flex flex-wrap items-center gap-4">
        <div class="flex items-center gap-2">
          <span class="text-xs font-bold text-slate-600 flex items-center gap-1.5">
            <Filter class="w-4 h-4" />
            GRADE FILTER:
          </span>
          <select 
            v-model="selectedGradeFilter" 
            @change="selectedClassFilter = ''"
            class="bg-white border border-slate-200 rounded-xl px-3 py-2 text-xs font-bold text-slate-800 outline-none cursor-pointer"
          >
            <option value="">ALL GRADES</option>
            <option v-for="g in availableGrades" :key="g" :value="g">{{ g }}</option>
          </select>
        </div>

        <div class="flex items-center gap-2">
          <span class="text-xs font-bold text-slate-600 flex items-center gap-1.5">
            <Filter class="w-4 h-4" />
            CLASS FILTER:
          </span>
          <select 
            v-model="selectedClassFilter" 
            class="bg-white border border-slate-200 rounded-xl px-3 py-2 text-xs font-bold text-slate-800 outline-none cursor-pointer"
          >
            <option value="">ALL CLASSES</option>
            <option v-for="cls in availableClassesForFilter" :key="cls" :value="cls">{{ cls }}</option>
          </select>
        </div>

        <button 
          v-if="selectedGradeFilter || selectedClassFilter" 
          @click="selectedGradeFilter = ''; selectedClassFilter = ''" 
          class="text-xs font-bold text-indigo-600 hover:text-indigo-800 underline cursor-pointer ml-auto"
        >
          CLEAR FILTER
        </button>
      </div>

      <div class="overflow-x-auto">
        <table class="w-full text-left text-xs border-collapse print-table">
          <thead>
            <tr class="bg-slate-50 text-slate-500 uppercase tracking-wider select-none font-semibold">
              <th @click="sortSubjectTable('grade')" class="p-4 border-b cursor-pointer hover:bg-slate-100 transition">
                GRADE <span class="text-indigo-600">{{ subjectSortKey === 'grade' ? (subjectSortAsc ? '▲' : '▼') : '↕' }}</span>
              </th>
              <th @click="sortSubjectTable('className')" class="p-4 border-b cursor-pointer hover:bg-slate-100 transition">
                CLASS <span class="text-indigo-600">{{ subjectSortKey === 'className' ? (subjectSortAsc ? '▲' : '▼') : '↕' }}</span>
              </th>
              <th @click="sortSubjectTable('subjectName')" class="p-4 border-b cursor-pointer hover:bg-slate-100 transition">
                AFFECTED SUBJECT <span class="text-indigo-600">{{ subjectSortKey === 'subjectName' ? (subjectSortAsc ? '▲' : '▼') : '↕' }}</span>
              </th>
              <th @click="sortSubjectTable('totalPeriods')" class="p-4 border-b cursor-pointer hover:bg-slate-100 transition">
                AFFECTED SLOTS <span class="text-indigo-600">{{ subjectSortKey === 'totalPeriods' ? (subjectSortAsc ? '▲' : '▼') : '↕' }}</span>
              </th>
            </tr>
          </thead>
          <tbody class="divide-y divide-slate-100 font-medium text-slate-800">
            <tr v-if="filteredSubjectStats.length === 0">
              <td colspan="4" class="p-8 text-center text-slate-400 font-medium">NO MATCHING SUBJECT RECORDS FOUND FOR THIS FILTER</td>
            </tr>
            <tr v-for="s in filteredSubjectStats" :key="s.id" class="hover:bg-slate-50">
              <td class="p-4 font-bold text-slate-600">{{ s.grade }}</td>
              <td class="p-4 font-bold text-slate-900">{{ s.className }}</td>
              <td class="p-4 font-bold text-indigo-600">{{ s.subjectName }}</td>
              <td class="p-4 font-bold text-amber-600">{{ s.totalPeriods }} SLOTS</td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>

    <!-- ================= TAB 7: Overall Teacher Interruption Master Table ================= -->
    <div v-if="currentTab === 'teacher'" class="bg-white p-8 rounded-3xl shadow-sm ring-1 ring-slate-900/5 space-y-6 animate-fadeIn">
      <div class="flex justify-between items-center">
        <div>
          <h2 class="text-lg font-bold text-slate-900 flex items-center gap-2">
            <UsersRound class="w-5 h-5 text-indigo-600" />
            OVERALL SUBSTITUTE WORKLOAD & TEACHER INTERRUPTIONS
          </h2>
          <p class="text-xs text-slate-500 mt-1 font-medium">DISPLAY OF ALL REGISTERED TEACHERS ALONG WITH SUBSTITUTE LOADS AND CLASS INTERRUPTIONS.</p>
        </div>
        <button @click="exportSinglePdf" class="no-print px-4 py-2 bg-slate-100 hover:bg-slate-200 text-slate-700 rounded-xl text-xs font-bold transition flex items-center gap-1.5 cursor-pointer">
          <Printer class="w-4 h-4" />
          PRINT / SAVE AS PDF
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
              <td class="p-4 font-bold text-indigo-600">{{ stat.count }} TIMES</td>
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

// 🌐 Global Date Range State
const startDate = ref('')
const endDate = ref('')

const stats = ref([])
const interruptionLogs = ref([])
const reasonStats = ref([])
const dayOfWeekStats = ref([])
const classStats = ref([])
const subjectStats = ref([])

// 🌟 Filter State Variables
const selectedClassGradeFilter = ref('') // Class view grade filter
const selectedGradeFilter = ref('')      // Subject view grade filter
const selectedClassFilter = ref('')      // Subject view class filter

// 🌟 Fixed Grade Weight Sorting Dictionary (Ensures Year 1 to Year 6 are always in strict ascending order)
const gradeOrderMap = {
  'YEAR 1': 1,
  'YEAR 2': 2,
  'YEAR 3': 3,
  'YEAR 4': 4,
  'YEAR 5': 5,
  'YEAR 6': 6
};

const sortGrgradesHelper = (setObj) => {
  return Array.from(setObj).sort((a, b) => {
    const wA = gradeOrderMap[a] !== undefined ? gradeOrderMap[a] : 99;
    const wB = gradeOrderMap[b] !== undefined ? gradeOrderMap[b] : 99;
    return wA - wB;
  });
};

// 🌟 Helper Function: Derive English Grade from Class Name
const getGradeFromClass = (cName) => {
  if (!cName) return 'OTHER GRADES';
  const match = cName.match(/^([0-9]+)/);
  if (match) {
    const gradeNum = match[1];
    const gradeMap = {
      '1': 'YEAR 1',
      '2': 'YEAR 2',
      '3': 'YEAR 3',
      '4': 'YEAR 4',
      '5': 'YEAR 5',
      '6': 'YEAR 6'
    };
    return gradeMap[gradeNum] || `YEAR ${gradeNum}`;
  }
  return 'ALL / OTHER';
};

// 🌟 Dynamic Available Grades for Class View (Sorted)
const availableClassGrades = computed(() => {
  const grades = new Set()
  classStats.value.forEach(c => {
    const g = getGradeFromClass(c.className)
    if (g && g !== 'ALL / OTHER') grades.add(g)
  })
  return sortGrgradesHelper(grades)
})

// 🌟 Class View: Filtered class list based on grade selection
const filteredClassStats = computed(() => {
  let list = classStats.value

  if (selectedClassGradeFilter.value) {
    list = list.filter(c => getGradeFromClass(c.className) === selectedClassGradeFilter.value)
  }

  return [...list].sort((a, b) => smartSort(a[classSortKey.value], b[classSortKey.value], classSortAsc.value))
})

// 🌟 Subject View: Dynamic available grades list (Sorted)
const availableGrades = computed(() => {
  const grades = new Set()
  subjectStats.value.forEach(s => {
    if (s.grade && s.grade !== 'ALL / OTHER') grades.add(s.grade)
  })
  return sortGrgradesHelper(grades)
})

// 🌟 Subject View: Dynamic classes available for current grade selection
const availableClassesForFilter = computed(() => {
  const classes = new Set()
  subjectStats.value.forEach(s => {
    if (!selectedGradeFilter.value || s.grade === selectedGradeFilter.value) {
      if (s.className && s.className !== 'UNKNOWN CLASS') classes.add(s.className)
    }
  })
  return Array.from(classes).sort()
})

// 🌟 Subject View: Real-time filtering based on grade and class selectors
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

// 🌟 Four Category Card Engine & Truncation Protection (Updated with CalendarCheck, BriefcaseBusiness, Building2, FolderOpen)
const groupedReasonStats = computed(() => {
  if (!reasonStats.value.length) return [];

  const groups = {
    personal: { id: 'personal', title: 'PERSONAL LEAVE', iconComponent: CalendarCheck, items: [], total: 0, badgeClass: 'bg-orange-100 text-orange-700', barClass: 'bg-orange-500' },
    official: { id: 'official', title: 'OFFICIAL / OUT-OF-SCHOOL DUTY', iconComponent: BriefcaseBusiness, items: [], total: 0, badgeClass: 'bg-blue-100 text-blue-700', barClass: 'bg-blue-500' },
    internal: { id: 'internal', title: 'SCHOOL INTERNAL TASKS', iconComponent: Building2, items: [], total: 0, badgeClass: 'bg-emerald-100 text-emerald-700', barClass: 'bg-emerald-500' },
    others:   { id: 'others', title: 'ARCHIVE / OTHER MISC', iconComponent: FolderOpen, items: [], total: 0, badgeClass: 'bg-slate-200 text-slate-700', barClass: 'bg-slate-400' }
  };

  const totalPAll = reasonStats.value.reduce((acc, cur) => acc + cur.count, 0);

  reasonStats.value.forEach(item => {
    let cleanReason = item.reason.replace(/\[.*?\]\s*/, '');
    let targetGroup = 'others';

    if (item.reason.includes('[个人请假]') || item.reason.includes('CUTI PERIBADI') || item.reason.includes('PERSONAL')) targetGroup = 'personal';
    else if (item.reason.includes('[离校公干]') || item.reason.includes('TUGAS LUAR') || item.reason.includes('OFFICIAL')) targetGroup = 'official';
    else if (item.reason.includes('[校内任务]') || item.reason.includes('TUGAS DALAMAN') || item.reason.includes('INTERNAL')) targetGroup = 'internal';

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
        reason: 'OTHER MISC (LAIN-LAIN)',
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

// Other Regular Statistics
const totalSubstituteCount = computed(() => stats.value.reduce((acc, cur) => acc + (cur.count || 0), 0))
const totalInterruptionPeriods = computed(() => interruptionLogs.value.reduce((acc, cur) => acc + ((cur.end_period || 0) - (cur.start_period || 0) + 1), 0))
const sortedSubstituteStats = computed(() => [...stats.value].sort((a, b) => (b.count || 0) - (a.count || 0)))

// Reset Date Selection
const resetDateFilter = () => {
  startDate.value = ''
  endDate.value = ''
  selectedClassGradeFilter.value = ''
  selectedGradeFilter.value = ''
  selectedClassFilter.value = ''
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

  // 3. Fetch precise leave records within this timeframe
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

  assignments?.forEach(a => {
    if (a.assignment_type !== 'swap' && a.sub_teacher_id && teacherMap[a.sub_teacher_id]) {
      teacherMap[a.sub_teacher_id].count++
    }
  })

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

  if (mmiData) {
    const totalPAll = mmiData.reduce((acc, cur) => acc + ((cur.end_period || 0) - (cur.start_period || 0) + 1), 0)
    const reasons = {}
    mmiData.forEach(l => { 
      const pCount = (l.end_period || 0) - (l.start_period || 0) + 1; 
      let rawReason = (l.reason || 'NO RECORD').trim().toUpperCase();
      rawReason = rawReason.replace(/^(教师请假:\s*|CUTI GURU:\s*)/i, ''); 
      reasons[rawReason] = (reasons[rawReason] || 0) + pCount;
    })
    
    reasonStats.value = Object.entries(reasons)
      .map(([reason, count]) => ({ reason, count, percentage: totalPAll > 0 ? ((count / totalPAll) * 100).toFixed(1) : 0 }))
      .sort((a, b) => b.count - a.count)

    const dayNames = { 1: 'MONDAY', 2: 'TUESDAY', 3: 'WEDNESDAY', 4: 'THURSDAY', 5: 'FRIDAY', 6: 'SATURDAY', 7: 'SUNDAY' }
    const daysCount = {}
    mmiData.forEach(l => { const dIndex = new Date(l.interruption_date).getDay() || 7; const dName = dayNames[dIndex] || 'OTHER'; const pCount = (l.end_period || 0) - (l.start_period || 0) + 1; daysCount[dName] = (daysCount[dName] || 0) + pCount })
    dayOfWeekStats.value = ['MONDAY', 'TUESDAY', 'WEDNESDAY', 'THURSDAY', 'FRIDAY'].map(day => ({ day, count: daysCount[day] || 0, percentage: totalPAll > 0 ? (((daysCount[day] || 0) / totalPAll) * 100).toFixed(1) : 0 }))
  }

  const classMap = {}
  const subjectDetailMap = {} 
  let totalClassPeriods = 0

  const cleanClassName = (rawStr) => {
    if (!rawStr) return '';
    let cleaned = rawStr.replace(/^(班级|班級|KELAS|CLASS)\s*[:：]\s*/i, '').trim();
    cleaned = cleaned.replace(/^(班级|班級|KELAS|CLASS)\s*[:：]\s*/i, '').trim();
    if (!cleaned || /VIRTUAL_CLASS/i.test(cleaned)) return '';
    return cleaned.toUpperCase();
  };

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
    if (/^GURU:/i.test(rawTarget) || rawTarget.includes('教师') || teacherNameSet.has(rawTarget.toUpperCase()) || /VIRTUAL_CLASS/i.test(rawTarget)) return; 
    const pCount = (l.end_period || 0) - (l.start_period || 0) + 1; 
    processClassNames(rawTarget, pCount);
  })

  leaveData?.forEach(req => {
    if (swapLeaveIds.has(req.id)) return;

    processClassNames(req.class_name, 1);

    const sub = req.subject ? req.subject.trim().toUpperCase() : 'UNKNOWN SUBJECT';
    if (sub && sub !== 'UNKNOWN SUBJECT' && !sub.includes('VIRTUAL_SUB')) {
      const cNames = req.class_name ? req.class_name.split(/,|、|\//) : ['UNKNOWN CLASS'];
      cNames.forEach(c => {
        const cleanC = cleanClassName(c) || 'UNKNOWN CLASS';
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