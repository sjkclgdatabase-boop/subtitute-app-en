<template>
  <!-- 🌟 1. 统一的桌面端最小宽度约束 -->
  <div class="p-4 sm:p-8 mx-auto min-h-screen space-y-8 min-w-[1024px]">
    
    <!-- Header Section: Unified card style, typography, and gradient title -->
    <div class="bg-white rounded-3xl p-6 sm:p-8 shadow-sm ring-1 ring-slate-900/5 space-y-2">
      <h1 class="text-xl sm:text-2xl md:text-3xl font-extrabold tracking-tight text-transparent bg-clip-text bg-gradient-to-r from-slate-900 via-indigo-800 to-violet-800 flex items-center gap-3">
        <GraduationCap class="w-8 h-8 text-indigo-700 shrink-0" />
        MMI INTERRUPTION RECORD CENTER
      </h1>
      <p class="text-slate-500 text-xs sm:text-sm font-medium leading-relaxed">
        PROTECTING INSTRUCTIONAL TIME · RECORDS BY CLASS AND TEACHER TO GUARANTEE TEACHING HOURS.
      </p>
    </div>

    <!-- Mode Switcher Card: Unified style guidelines -->
    <div class="bg-white rounded-3xl p-6 sm:p-8 shadow-sm ring-1 ring-slate-900/5 space-y-4">
      <div class="flex items-center gap-2">
        <span class="w-2.5 h-2.5 rounded-full bg-indigo-600"></span>
        <span class="text-lg font-bold text-slate-900">SELECT INTERRUPTION RECORD MODE</span>
      </div>

      <div class="inline-flex bg-slate-100 p-1.5 rounded-2xl shadow-inner">
        <button 
          @click="activeTab = 'class'" 
          :class="activeTab === 'class' ? 'bg-slate-900 text-white shadow-md' : 'text-slate-600 hover:text-slate-900'"
          class="px-6 py-2.5 rounded-xl text-xs font-bold transition-all flex items-center justify-center gap-2 cursor-pointer"
        >
          <School class="w-4 h-4" /> CLASS RECORDS
        </button>
        <button 
          @click="activeTab = 'teacher'" 
          :class="activeTab === 'teacher' ? 'bg-slate-900 text-white shadow-md' : 'text-slate-600 hover:text-slate-900'"
          class="px-6 py-2.5 rounded-xl text-xs font-bold transition-all flex items-center justify-center gap-2 cursor-pointer"
        >
          <Users class="w-4 h-4" /> TEACHER RECORDS
        </button>
      </div>
    </div>

    <!-- Dimension 1: Record Interruption by Class -->
    <div v-if="activeTab === 'class'" class="bg-white rounded-3xl p-6 sm:p-8 shadow-sm ring-1 ring-slate-900/5 mb-8">
      <h2 class="text-lg font-bold text-slate-900 mb-6 flex items-center gap-2">
        <span class="w-2.5 h-2.5 rounded-full bg-indigo-600"></span>
        INTERRUPTION REGISTRATION
      </h2>

      <div class="grid grid-cols-1 md:grid-cols-2 gap-6 mb-6">
        <div>
          <label class="block text-xs font-bold text-slate-700 mb-2 flex items-center gap-1.5">
            <CalendarDays class="w-4 h-4 text-indigo-600" /> INTERRUPTION DATE:
          </label>
          <input 
            type="date" 
            v-model="classForm.date" 
            class="w-full bg-slate-50 border border-slate-200 px-4 py-3 rounded-2xl text-xs font-semibold text-slate-800 focus:outline-none focus:ring-2 focus:ring-indigo-500 cursor-pointer"
          />
        </div>

        <!-- 🌟 2. English Category Selection -->
        <div>
          <label class="block text-xs font-bold text-slate-700 mb-2 uppercase tracking-wider flex items-center gap-1.5">
            <AlertTriangle class="w-4 h-4 text-amber-500" /> INTERRUPTION CATEGORY (AUTO TAGGED):
          </label>
          
          <div class="flex flex-wrap gap-2 mb-3">
            <label v-for="cat in ['[ACADEMIC]', '[EVENT]', '[SEMINAR]', '[HOLIDAY]', '[UNCATEGORIZED]']" :key="cat" class="cursor-pointer">
              <input type="radio" v-model="classForm.category" :value="cat" class="hidden" />
              <span :class="classForm.category === cat ? 'bg-indigo-600 text-white shadow-sm' : 'bg-slate-100 text-slate-600 hover:bg-slate-200'" class="px-3 py-1.5 rounded-lg text-xs font-bold transition-all inline-block">
                {{ cat }}
              </span>
            </label>
          </div>

          <input 
            type="text" 
            v-model="classForm.eventName" 
            placeholder="PLEASE SPECIFY EVENT NAME (E.G. ANTI-DRUG CAMPAIGN, YEAR-END EXAM)..." 
            class="w-full bg-white border border-slate-200 px-4 h-11 rounded-2xl text-xs font-bold text-slate-900 uppercase focus:outline-none focus:ring-2 focus:ring-indigo-500 shadow-sm"
          />
        </div>
      </div>

      <div class="mb-6 bg-slate-50 p-6 rounded-2xl border border-slate-200 space-y-4">
        <label class="block text-xs font-bold text-slate-700 flex items-center gap-1.5">
          <Target class="w-4 h-4 text-indigo-600" /> Affected Scope:
        </label>
        
        <div class="flex flex-wrap gap-4">
          <label class="inline-flex items-center gap-2 cursor-pointer">
            <input type="radio" v-model="classForm.scopeType" value="specific" class="text-indigo-600 focus:ring-indigo-500" />
            <span class="text-xs font-bold text-slate-800">1. SPECIFIC CLASSES</span>
          </label>
          <label class="inline-flex items-center gap-2 cursor-pointer">
            <input type="radio" v-model="classForm.scopeType" value="grade" class="text-indigo-600 focus:ring-indigo-500" />
            <span class="text-xs font-bold text-slate-800">2. SPECIFIC GRADE</span>
          </label>
          <label class="inline-flex items-center gap-2 cursor-pointer">
            <input type="radio" v-model="classForm.scopeType" value="all" class="text-indigo-600 focus:ring-indigo-500" />
            <span class="text-xs font-bold text-slate-800">3. ALL CLASSES IN SCHOOL</span>
          </label>
        </div>

        <div v-if="classForm.scopeType === 'specific'" class="space-y-3 pt-2">
          <div class="flex justify-between items-center pb-2 border-b border-slate-200/80 text-xs">
            <span class="font-bold text-slate-500">Please select affected classes:</span>
            <div class="space-x-3">
              <button type="button" @click="selectAllClasses" class="text-indigo-600 hover:text-indigo-800 font-bold cursor-pointer">
                Select All
              </button>
              <span class="text-slate-300">|</span>
              <button type="button" @click="clearAllClasses" class="text-slate-500 hover:text-slate-700 font-bold cursor-pointer">
                Clear
              </button>
            </div>
          </div>

          <div v-for="(classes, grade) in groupedClasses" :key="grade" class="flex flex-col sm:flex-row sm:items-center gap-3 bg-white p-3.5 rounded-2xl border border-slate-200/80 shadow-xs">
            <div class="w-28 shrink-0 flex items-center justify-between sm:justify-start gap-2">
              <span class="text-xs font-black text-slate-700 uppercase tracking-wider">GRADE {{ grade }}</span>
              <span class="text-[10px] bg-slate-100 text-slate-600 px-2 py-0.5 rounded-full font-extrabold">
                {{ classes.filter(c => classForm.selectedClasses.includes(c)).length }}/{{ classes.length }}
              </span>
            </div>

            <div class="flex flex-wrap gap-2 flex-1">
              <label 
                v-for="c in classes" 
                :key="c" 
                :class="classForm.selectedClasses.includes(c) ? 'bg-indigo-600 text-white border-indigo-600 shadow-xs scale-105' : 'bg-slate-50 text-slate-700 border-slate-200 hover:border-indigo-300 hover:bg-indigo-50/30'"
                class="w-14 h-9 border rounded-xl text-xs font-extrabold flex items-center justify-center cursor-pointer transition-all select-none"
              >
                <input type="checkbox" :value="c" v-model="classForm.selectedClasses" class="hidden" />
                {{ c }}
              </label>
            </div>
          </div>
        </div>

        <div v-if="classForm.scopeType === 'grade'" class="flex flex-wrap gap-2 pt-2">
          <button 
            v-for="g in [1, 2, 3, 4, 5, 6]" 
            :key="g"
            type="button"
            @click="classForm.selectedGrade = g"
            :class="classForm.selectedGrade === g ? 'bg-indigo-600 text-white shadow-xs' : 'bg-white text-slate-700 border-slate-200'"
            class="px-4 py-2 border rounded-xl text-xs font-bold transition-all cursor-pointer"
          >
            GRADE {{ g }}
          </button>
        </div>

        <div v-if="classForm.scopeType === 'all'" class="text-xs text-indigo-700 font-bold pt-2">
          ✅ INVOLVING ALL CLASSES FROM GRADE 1 TO 6
        </div>
      </div>

      <div class="grid grid-cols-1 md:grid-cols-2 gap-6 mb-6">
        <div class="relative w-full">
          <label class="block text-xs font-bold text-slate-700 mb-2 flex items-center gap-1.5">
            <Clock class="w-4 h-4 text-indigo-600" /> START TIME SLOT:
          </label>
          <select v-model="classForm.startPeriod" class="w-full bg-slate-50 border border-slate-200 px-4 py-3 rounded-2xl text-xs font-bold text-slate-800 appearance-none pr-8 cursor-pointer">
            <option v-for="p in 11" :key="p" :value="p">PERIOD {{ p }}</option>
          </select>
          <div class="absolute right-4 top-[38px] pointer-events-none text-slate-500">
            <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 9l-7 7-7-7"></path></svg>
          </div>
        </div>
        <div class="relative w-full">
          <label class="block text-xs font-bold text-slate-700 mb-2 flex items-center gap-1.5">
            <Clock class="w-4 h-4 text-indigo-600" /> END TIME SLOT:
          </label>
          <select v-model="classForm.endPeriod" class="w-full bg-slate-50 border border-slate-200 px-4 py-3 rounded-2xl text-xs font-bold text-slate-800 appearance-none pr-8 cursor-pointer">
            <option v-for="p in 11" :key="p" :value="p">PERIOD {{ p }}</option>
          </select>
          <div class="absolute right-4 top-[38px] pointer-events-none text-slate-500">
            <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 9l-7 7-7-7"></path></svg>
          </div>
        </div>
      </div>

      <div class="mb-6">
        <label class="block text-xs font-bold text-slate-700 mb-2 flex items-center gap-1.5">
          <FileText class="w-4 h-4 text-indigo-600" /> DESCRIPTION & INTERVENTION
        </label>
        <input 
          v-model="classForm.remarks" 
          type="text" 
          placeholder="E.G. DENGUE FEVER TALK IN THE MAIN HALL." 
          class="w-full bg-slate-50 border border-slate-200 px-4 py-3 rounded-2xl text-xs font-semibold text-slate-800 uppercase focus:outline-none focus:ring-2 focus:ring-indigo-500"
        />
      </div>

      <button 
        @click="submitClassInterruption" 
        class="bg-slate-900 hover:bg-slate-800 text-white px-6 h-11 rounded-2xl text-xs font-bold shadow-md transition-all cursor-pointer flex items-center gap-2"
      >
        <Save class="w-4 h-4" /> SAVE CLASS INTERRUPTION RECORD
      </button>
    </div>

    <!-- Dimension 2: Record Interruption by Teacher -->
    <div v-if="activeTab === 'teacher'" class="bg-white rounded-3xl p-6 sm:p-8 shadow-sm ring-1 ring-slate-900/5 mb-8">
      <h2 class="text-lg font-bold text-slate-900 mb-6 flex items-center gap-2">
        <span class="w-2.5 h-2.5 rounded-full bg-violet-600"></span>
        TEACHER INTERRUPTION REGISTRATION
      </h2>

      <div class="grid grid-cols-1 md:grid-cols-2 gap-6 mb-6">
        <div>
          <label class="block text-xs font-bold text-slate-700 mb-2 flex items-center gap-1.5">
            <CalendarDays class="w-4 h-4 text-violet-600" /> INTERRUPTION DATE:
          </label>
          <input 
            type="date" 
            v-model="teacherForm.date" 
            @change="loadTeacherSubjects"
            class="w-full bg-slate-50 border border-slate-200 px-4 py-3 rounded-2xl text-xs font-semibold text-slate-800 focus:outline-none focus:ring-2 focus:ring-indigo-500 cursor-pointer"
          />
        </div>

        <div class="relative w-full">
          <label class="block text-xs font-bold text-slate-700 mb-2 flex items-center gap-1.5">
            <Users class="w-4 h-4 text-violet-600" /> SELECT AFFECTED / LEAVING TEACHER
          </label>
          <select 
            v-model="teacherForm.teacherId" 
            @change="loadTeacherSubjects"
            class="w-full bg-slate-50 border border-slate-200 px-4 py-3 rounded-2xl text-xs font-semibold text-slate-800 focus:outline-none focus:ring-2 focus:ring-indigo-500 appearance-none pr-8 truncate cursor-pointer"
          >
            <option value="" disabled>-- PLEASE SELECT A TEACHER --</option>
            <option v-for="t in teachersList" :key="t.id" :value="t.id">{{ t.name }}</option>
          </select>
          <div class="absolute right-4 top-[38px] pointer-events-none text-slate-500">
            <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 9l-7 7-7-7"></path></svg>
          </div>
        </div>
      </div>

      <!-- 🌟 3. English Teacher Category Selection -->
      <div class="mb-6">
        <label class="block text-xs font-bold text-slate-700 mb-2 flex items-center gap-1.5">
          <AlertTriangle class="w-4 h-4 text-amber-500" /> ABSENCE CATEGORY (AUTO TAGGED):
        </label>
        
        <div class="flex flex-wrap gap-2 mb-3">
          <label v-for="cat in ['[PERSONAL LEAVE]', '[OFFICIAL DUTY]', '[INTERNAL TASK]', '[UNCATEGORIZED]']" :key="cat" class="cursor-pointer">
            <input type="radio" v-model="teacherForm.category" :value="cat" class="hidden" />
            <span :class="teacherForm.category === cat ? 'bg-violet-600 text-white shadow-sm' : 'bg-slate-100 text-slate-600 hover:bg-slate-200'" class="px-3 py-1.5 rounded-lg text-xs font-bold transition-all inline-block">
              {{ cat }}
            </span>
          </label>
        </div>

        <input 
          v-model="teacherForm.eventName" 
          type="text" 
          placeholder="PLEASE SPECIFY DETAILS (E.G. ACCOMPANYING TEAM, CURRICULUM MEETING, SICK LEAVE)..." 
          class="w-full bg-white border border-slate-200 px-4 h-11 rounded-2xl text-xs font-semibold text-slate-800 uppercase shadow-sm focus:outline-none focus:ring-2 focus:ring-violet-500"
        />
      </div>

      <div class="bg-indigo-50/50 p-6 rounded-2xl border border-indigo-100 mb-6">
        <h3 class="text-xs font-bold uppercase tracking-wider text-indigo-900 mb-3 flex items-center justify-between">
          <span>📚 Automatic Data Adjustment: Original Timetable and Affected Classes</span>
          <span v-if="loadingSubjects" class="text-xs font-normal text-indigo-600 animate-pulse">LOADING TIMETABLE</span>
        </h3>

        <div v-if="exportedSubjects.length === 0" class="text-xs text-slate-400 py-4 text-center">
          PLEASE SELECT DATE AND TEACHER TO LOAD AFFECTED CLASSES AND SUBJECTS
        </div>

        <div v-else class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 gap-3">
          <div 
            v-for="sub in exportedSubjects" 
            :key="sub.period"
            class="p-3 bg-white border border-indigo-100 rounded-xl shadow-sm flex items-center justify-between"
          >
            <div>
              <div class="flex items-center gap-2">
                <span class="text-xs font-extrabold text-slate-900">PERIOD {{ sub.period }}</span>
                <span v-if="sub.is_combined" class="px-1.5 py-0.5 bg-violet-100 text-violet-700 rounded text-[9px] font-bold">COMBINED CLASS</span>
              </div>
              <span class="text-xs text-slate-500">{{ sub.class_name }} · {{ sub.subject }}</span>
            </div>
            <span class="text-[10px] px-2 py-0.5 rounded bg-indigo-100 text-indigo-700 font-bold">AFFECTED</span>
          </div>
        </div>
      </div>

      <button 
        @click="submitTeacherInterruption" 
        :disabled="exportedSubjects.length === 0"
        class="bg-slate-900 hover:bg-slate-800 disabled:opacity-50 text-white px-6 h-11 rounded-2xl text-xs font-bold shadow-md transition-all cursor-pointer flex items-center gap-2"
      >
        <Save class="w-4 h-4" /> CONFIRM AND SAVE TEACHER INTERRUPTION RECORD
      </button>
    </div>

    <!-- Interruption Log History Table Area -->
    <div class="bg-white rounded-3xl p-6 sm:p-8 shadow-sm ring-1 ring-slate-900/5">
      
      <div class="flex flex-col sm:flex-row justify-between items-start sm:items-center mb-5 gap-4">
        <h2 class="text-lg font-bold text-slate-900 flex items-center gap-2">
          <History class="w-5 h-5 text-indigo-600" />
          <span>MMI INTERRUPTION HISTORY SCHEDULE</span>
          <span class="text-xs bg-slate-100 px-2.5 py-1 rounded-full text-slate-600 font-semibold">
            TOTAL {{ filteredLogs.length }} RECORDS
          </span>
        </h2>

        <button 
          @click="exportLogsToExcel" 
          class="bg-indigo-600 hover:bg-indigo-700 text-white px-5 h-11 rounded-2xl text-xs font-bold shadow-sm transition-all flex items-center gap-2 cursor-pointer shrink-0"
        >
          <FileSpreadsheet class="w-4 h-4" /> EXPORT EXCEL SCHEDULE
        </button>
      </div>

      <!-- Filters -->
      <div class="flex flex-wrap items-center gap-3 mb-6">
        <div class="relative min-w-[140px]">
          <select v-model="typeFilter" class="w-full bg-slate-50 border border-slate-200 px-4 h-11 rounded-2xl text-xs font-bold text-slate-700 focus:outline-none appearance-none pr-8 cursor-pointer">
            <option value="all">ALL TYPES</option>
            <option value="class">CLASS INTERRUPTION</option>
            <option value="teacher">TEACHER INTERRUPTION</option>
          </select>
          <div class="absolute right-3 top-1/2 -translate-y-1/2 pointer-events-none text-slate-500">
            <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 9l-7 7-7-7"></path></svg>
          </div>
        </div>

        <div class="relative min-w-[200px]">
          <select v-model="dateRangeFilter" class="w-full bg-slate-50 border border-slate-200 px-4 h-11 rounded-2xl text-xs font-bold text-slate-700 focus:outline-none appearance-none pr-8 cursor-pointer">
            <option value="all">ALL TIME PERIODS</option>
            <option value="week">📅 THIS WEEK (LAST 7 DAYS)</option>
            <option value="month">📅 THIS MONTH (CURRENT MONTH)</option>
          </select>
          <div class="absolute right-3 top-1/2 -translate-y-1/2 pointer-events-none text-slate-500">
            <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 9l-7 7-7-7"></path></svg>
          </div>
        </div>

        <div class="relative min-w-[200px]">
          <select v-model="selectedMonth" class="w-full bg-slate-50 border border-slate-200 px-4 h-11 rounded-2xl text-xs font-bold text-slate-700 focus:outline-none appearance-none pr-8 cursor-pointer">
            <option value="all">🗓️ ALL MONTHS (YEAR-ROUND)</option>
            <option v-for="m in 12" :key="m" :value="String(m)">MONTH {{ m }}</option>
          </select>
          <div class="absolute right-3 top-1/2 -translate-y-1/2 pointer-events-none text-slate-500">
            <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 9l-7 7-7-7"></path></svg>
          </div>
        </div>

        <div class="flex-1 min-w-[200px]">
          <input 
            type="text" 
            v-model="searchQuery" 
            placeholder="SEARCH TEACHER / CLASS / REASON..." 
            class="w-full bg-slate-50 border border-slate-200 px-4 h-11 rounded-2xl text-xs font-semibold text-slate-800 focus:outline-none focus:ring-2 focus:ring-indigo-500"
          />
        </div>
      </div>

      <!-- Table -->
      <div class="overflow-x-auto">
        <table class="w-full text-left border-collapse text-xs whitespace-nowrap">
          <thead>
            <tr class="bg-slate-50 text-slate-500 uppercase tracking-wider font-semibold border-b border-slate-200 select-none">
              <th @click="handleSort('interruption_date')" class="py-4 px-4 cursor-pointer hover:bg-slate-100 transition text-left">
                <div class="flex items-center gap-1">
                  <span>DATE</span>
                  <span class="text-[10px] text-indigo-600 font-bold">
                    {{ sortField === 'interruption_date' ? (sortOrder === 'asc' ? '▲' : '▼') : '↕' }}
                  </span>
                </div>
              </th>

              <th @click="handleSort('type')" class="py-4 px-4 cursor-pointer hover:bg-slate-100 transition text-center">
                <div class="flex items-center justify-center gap-1">
                  <span>TYPE</span>
                  <span class="text-[10px] text-indigo-600 font-bold">
                    {{ sortField === 'type' ? (sortOrder === 'asc' ? '▲' : '▼') : '↕' }}
                  </span>
                </div>
              </th>

              <th @click="handleSort('target_display')" class="py-4 px-4 cursor-pointer hover:bg-slate-100 transition text-left">
                <div class="flex items-center gap-1">
                  <span>TARGET / SCOPE</span>
                  <span class="text-[10px] text-indigo-600 font-bold">
                    {{ sortField === 'target_display' ? (sortOrder === 'asc' ? '▲' : '▼') : '↕' }}
                  </span>
                </div>
              </th>

              <th @click="handleSort('start_period')" class="py-4 px-4 cursor-pointer hover:bg-slate-100 transition text-center">
                <div class="flex items-center justify-center gap-1">
                  <span>TIME SLOT</span>
                  <span class="text-[10px] text-indigo-600 font-bold">
                    {{ sortField === 'start_period' ? (sortOrder === 'asc' ? '▲' : '▼') : '↕' }}
                  </span>
                </div>
              </th>

              <th @click="handleSort('reason')" class="py-4 px-4 cursor-pointer hover:bg-slate-100 transition text-center">
                <div class="flex items-center justify-center gap-1">
                  <span>REASON</span>
                  <span class="text-[10px] text-indigo-600 font-bold">
                    {{ sortField === 'reason' ? (sortOrder === 'asc' ? '▲' : '▼') : '↕' }}
                  </span>
                </div>
              </th>

              <th class="py-4 px-4 text-center">ACTIONS</th>
            </tr>
          </thead>
          <tbody class="divide-y divide-slate-100 font-medium text-slate-800">
            <tr v-if="filteredLogs.length === 0">
              <td colspan="6" class="py-12 text-center text-slate-400 font-medium">
                NO MMI INTERRUPTION RECORDS DETECTED
              </td>
            </tr>
            <tr v-for="log in filteredLogs" :key="log.id" class="hover:bg-slate-50/50 transition">
              
              <td class="py-4 px-4 font-bold text-slate-900 text-left">{{ log.interruption_date }}</td>
              
              <td class="py-4 px-4 text-center">
                <span :class="log.type === 'class' ? 'bg-indigo-50 text-indigo-700' : 'bg-violet-50 text-violet-700'" class="px-2.5 py-1 rounded-full text-xs font-bold inline-block">
                  {{ log.type === 'class' ? 'CLASS' : 'TEACHER' }}
                </span>
              </td>

              <td class="py-4 px-4 font-semibold text-slate-800 text-left whitespace-normal min-w-[200px]" :title="formatTargetDisplay(log.target_display)">
                {{ formatTargetDisplay(log.target_display) }}
              </td>

              <td class="py-4 px-4 text-center">
                <span class="bg-slate-100 px-2.5 py-1 rounded-lg text-xs text-slate-700 font-bold inline-block">
                  PERIOD {{ log.start_period }} - {{ log.end_period }}
                </span>
              </td>

              <td class="py-4 px-4 text-center">
                <span v-if="!log.reason || log.reason === '-'">-</span>
                <button 
                  v-else 
                  @click="openDetailModal(log)" 
                  class="text-indigo-600 hover:text-indigo-800 font-bold bg-indigo-50 hover:bg-indigo-100 px-4 py-2 rounded-xl transition inline-flex items-center justify-center gap-1.5 text-xs cursor-pointer"
                >
                  <span>VIEW DETAILS</span> 
                  <Eye class="w-3.5 h-3.5" />
                </button>
              </td>

              <td class="py-4 px-4 text-center">
                <button @click="deleteLog(log)" class="text-xs text-red-600 hover:text-red-800 font-bold px-4 py-2 bg-red-50 hover:bg-red-100 rounded-xl cursor-pointer transition inline-flex items-center gap-1.5">
                  <Trash2 class="w-3.5 h-3.5" /> DELETE
                </button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>

    <!-- Remark Details Modal -->
    <div v-if="showDetailModal" class="fixed inset-0 bg-slate-900/50 backdrop-blur-sm flex items-center justify-center z-50 p-4">
      <div class="bg-white rounded-3xl p-6 max-w-lg w-full shadow-2xl ring-1 ring-slate-900/10 animate-in fade-in zoom-in duration-200">
        <div class="flex justify-between items-center mb-4 border-b border-slate-100 pb-3">
          <h3 class="text-base font-bold text-slate-900 flex items-center gap-2">
            <span>📝 MMI INTERRUPTION DETAILS</span>
          </h3>
          <button @click="showDetailModal = false" class="text-slate-400 hover:text-slate-600 font-bold text-sm bg-slate-100 w-8 h-8 rounded-full flex items-center justify-center cursor-pointer">
            <X class="w-4 h-4" />
          </button>
        </div>

        <div class="space-y-3 mb-6 text-xs text-slate-700">
          <div class="bg-slate-50 p-3 rounded-xl border border-slate-100">
            <span class="font-bold text-slate-400 block mb-1">📅 DATE & TARGET:</span>
            <span class="font-semibold text-slate-900">{{ currentDetailLog?.interruption_date }} | {{ currentDetailLog?.target_display }}</span>
          </div>
          <div class="bg-slate-50 p-3 rounded-xl border border-slate-100">
            <span class="font-bold text-slate-400 block mb-1">⚠️ REASON FOR INTERRUPTION:</span>
            <span class="font-semibold text-slate-900">{{ currentDetailLog?.reason }}</span>
          </div>
          
          <div class="bg-indigo-50/50 p-4 rounded-xl border border-indigo-100">
            <span class="font-bold text-indigo-900 block mb-1">📄 NOTES & TIMETABLE CONTENT:</span>
            
            <p v-if="currentDetailLog?.type === 'teacher'" class="text-slate-800 leading-relaxed font-medium whitespace-pre-wrap">
              {{ currentDetailLog?.remarks?.replace('自动同步自请假录入 ', '') }}
            </p>
            
            <div v-if="currentDetailLog?.type === 'class'" class="text-slate-800 leading-relaxed font-medium whitespace-pre-wrap">
              <span v-if="currentDetailLog?.remarks">{{ currentDetailLog.remarks }}<br/><br/></span>
              
              <span v-if="loadingDetail" class="animate-pulse inline-flex items-center gap-1">
                <svg class="animate-spin h-3 w-3 text-slate-800" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24"><circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle><path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path></svg>
                MATCHING TIMETABLE...
              </span>
              <span v-else>{{ currentDetailAffectedClasses || 'NO TIMETABLE RECORD OR AFFECTED CLASSES' }}</span>
            </div>
          </div>
        </div>

        <div class="flex justify-end">
          <button @click="showDetailModal = false" class="bg-slate-900 hover:bg-slate-800 text-white px-5 py-2 rounded-xl text-xs font-bold transition cursor-pointer">
            CLOSE
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
  School, 
  Users, 
  CalendarDays, 
  Clock, 
  Save, 
  FileSpreadsheet, 
  Eye, 
  Trash2, 
  X,
  Target,
  FileText,
  History,
  AlertTriangle
} from 'lucide-vue-next'

const toast = useToast()
const activeTab = ref('class')

const getLocalToday = () => {
  const now = new Date()
  const year = now.getFullYear()
  const month = String(now.getMonth() + 1).padStart(2, '0')
  const day = String(now.getDate()).padStart(2, '0')
  return `${year}-${month}-${day}`
}

const classForm = ref({
  date: getLocalToday(),
  category: '[SEMINAR]', 
  eventName: '',         
  scopeType: 'specific',
  selectedClasses: [],
  selectedGrade: 1,
  startPeriod: 1,
  endPeriod: 4,
  remarks: ''
})

const groupedClasses = ref({})

const selectAllClasses = () => {
  const all = []
  Object.values(groupedClasses.value).forEach(arr => all.push(...arr))
  classForm.value.selectedClasses = all
}

const clearAllClasses = () => {
  classForm.value.selectedClasses = []
}

const fetchClasses = async () => {
  const { data } = await supabase
    .from('classes')
    .select('class_name, grade')
    .order('grade', { ascending: true })
    .order('class_name', { ascending: true })
  
  if (data) {
    const groups = {}
    data.forEach(c => {
      const g = c.grade || c.class_name[0]
      if (!groups[g]) groups[g] = []
      groups[g].push(c.class_name)
    })
    groupedClasses.value = groups
  }
}

const teacherForm = ref({
  date: getLocalToday(),
  teacherId: '',
  category: '[OFFICIAL DUTY]', 
  eventName: ''              
})

const teachersList = ref([])
const exportedSubjects = ref([])
const loadingSubjects = ref(false)

const interruptionLogs = ref([])

const searchQuery = ref('')
const typeFilter = ref('all')         
const dateRangeFilter = ref('all')    
const selectedMonth = ref('all')      

const sortField = ref('interruption_date')
const sortOrder = ref('desc')             

const handleSort = (field) => {
  if (sortField.value === field) {
    sortOrder.value = sortOrder.value === 'asc' ? 'desc' : 'asc'
  } else {
    sortField.value = field
    sortOrder.value = 'desc'
  }
}

const showDetailModal = ref(false)
const currentDetailLog = ref(null)
const currentDetailAffectedClasses = ref('')
const loadingDetail = ref(false)

const openDetailModal = async (log) => {
  currentDetailLog.value = log
  showDetailModal.value = true
  currentDetailAffectedClasses.value = ''

  if (log.type === 'class') {
    loadingDetail.value = true
    try {
      const logDate = new Date(log.interruption_date)
      const weekdayNum = logDate.getDay() 
      const queryWeekday = weekdayNum === 0 ? 7 : weekdayNum

      const { data: timetables, error } = await supabase
        .from('timetable')
        .select('*')
        .eq('weekday', queryWeekday)
      if (error) throw error

      if (timetables && timetables.length > 0) {
        const startP = Number(log.start_period)
        const endP = Number(log.end_period)
        const targetDisp = log.target_display || ''

        const matched = timetables.filter(t => {
          const itemWeekday = Number(t.weekday)
          const matchWd = itemWeekday === weekdayNum || itemWeekday === (weekdayNum === 0 ? 7 : weekdayNum)
          if (!matchWd) return false

          const p = Number(t.period)
          if (p < startP || p > endP) return false

          // 1. 全校影响 (三语兼容)
          if (targetDisp.includes('SEMUA') || targetDisp.includes('ALL') || targetDisp.includes('全校') || targetDisp.includes('WHOLE SCHOOL')) return true
          
          // 2. 年级影响 (三语兼容)
          if (targetDisp.includes('TAHUN') || targetDisp.includes('GRADE') || targetDisp.includes('全年级') || targetDisp.includes('Tahun') || targetDisp.includes('YEAR') || targetDisp.includes('年级')) {
            const match = targetDisp.match(/(?:TAHUN|GRADE|Tahun|YEAR|Year)\s*(\d)/i) || targetDisp.match(/(\d)\s*年级/)
            const grade = match ? match[1] : null
            return grade && String(t.class_name).startsWith(grade)
          }
          
          // 3. 班级影响 (三语兼容)
          const cleanTarget = targetDisp.replace(/^(?:KELAS|CLASS|班级)[:：]\s*/i, '').trim()
          const classList = cleanTarget.split(',').map(c => c.trim())
          
          return classList.some(c => 
            t.class_name === c || 
            t.class_name.toLowerCase() === c.toLowerCase() || 
            t.class_name.includes(c) || 
            c.includes(t.class_name)
          )
        })

        if (matched.length > 0) {
          matched.sort((a, b) => Number(a.period) - Number(b.period))
          const periods = [...new Set(matched.map(m => m.period))].sort((a, b) => a - b).join(', ')
          const classes = matched.map(m => `${m.class_name}(${m.subject || m.subject_name})`).join(', ')
          
          currentDetailAffectedClasses.value = `(INVOLVING SLOTS: PERIOD ${periods} | SUBJECTS: ${classes})`
        } else {
          currentDetailAffectedClasses.value = 'NO TIMETABLE RECORD OR AFFECTED CLASSES'
        }
      }
    } catch (err) {
      console.error("FAILED TO LOAD DETAILS:", err)
      currentDetailAffectedClasses.value = 'FAILED TO LOAD AFFECTED CLASSES'
    } finally {
      loadingDetail.value = false
    }
  }
}

const filteredLogs = computed(() => {
  const result = interruptionLogs.value.filter(log => {
    const query = searchQuery.value.toLowerCase().trim()
    const matchesSearch = !query || 
      log.target_display?.toLowerCase().includes(query) ||
      log.reason?.toLowerCase().includes(query) ||
      log.remarks?.toLowerCase().includes(query) ||
      log.interruption_date?.includes(query)

    const matchesType = typeFilter.value === 'all' || log.type === typeFilter.value

    let matchesDateRange = true
    if (dateRangeFilter.value !== 'all' && log.interruption_date) {
      const logDate = new Date(log.interruption_date)
      const now = new Date()

      if (dateRangeFilter.value === 'week') {
        const weekAgo = new Date()
        weekAgo.setDate(now.getDate() - 7)
        matchesDateRange = logDate >= weekAgo && logDate <= now
      } else if (dateRangeFilter.value === 'month') {
        matchesDateRange = logDate.getMonth() === now.getMonth() && logDate.getFullYear() === now.getFullYear()
      }
    }

    let matchesMonth = true
    if (selectedMonth.value !== 'all' && log.interruption_date) {
      const logDate = new Date(log.interruption_date)
      const logMonth = logDate.getMonth() + 1
      const currentYear = new Date().getFullYear()
      matchesMonth = (logDate.getFullYear() === currentYear) && (logMonth === Number(selectedMonth.value))
    }

    return matchesSearch && matchesType && matchesDateRange && matchesMonth
  })

  return result.sort((a, b) => {
    let valA = a[sortField.value] || ''
    let valB = b[sortField.value] || ''

    if (sortField.value === 'start_period') {
      valA = Number(valA)
      valB = Number(valB)
    }

    if (valA < valB) return sortOrder.value === 'asc' ? -1 : 1
    if (valA > valB) return sortOrder.value === 'asc' ? 1 : -1
    return 0
  })
})

const loadTeachers = async () => {
  const { data } = await supabase.from('teachers').select('id, name')
  if (data) teachersList.value = data
}

const loadTeacherSubjects = async () => {
  if (!teacherForm.value.teacherId || !teacherForm.value.date) {
    exportedSubjects.value = []
    return
  }
  loadingSubjects.value = true
  try {
    const { data } = await supabase
      .from('leave_requests')
      .select('*')
      .eq('teacher_id', teacherForm.value.teacherId)
      .eq('leave_date', teacherForm.value.date)
      .order('period', { ascending: true })

    const periodMap = new Map()
    ;(data || []).forEach(s => {
      if (!periodMap.has(s.period)) {
        periodMap.set(s.period, {
          ...s,
          is_combined: false
        })
      } else {
        const existing = periodMap.get(s.period)
        if (!existing.class_name.includes(s.class_name)) {
          existing.class_name = `${existing.class_name}/${s.class_name}`
          existing.is_combined = true 
        }
      }
    })

    exportedSubjects.value = Array.from(periodMap.values())
  } catch (err) {
    toast.error("FAILED TO EXTRACT SUBJECTS: " + err.message)
  } finally {
    loadingSubjects.value = false
  }
}

const submitClassInterruption = async () => {
  if (!classForm.value.eventName.trim()) {
    return toast.error("PLEASE SPECIFY THE EVENT NAME!")
  }

  let targetDisplay = ''
  if (classForm.value.scopeType === 'specific') {
    if (classForm.value.selectedClasses.length === 0) return toast.error("PLEASE SELECT AT LEAST ONE CLASS!")
    targetDisplay = classForm.value.selectedClasses.join(', ')
  } else if (classForm.value.scopeType === 'grade') {
    targetDisplay = `GRADE ${classForm.value.selectedGrade}`
  } else {
    targetDisplay = 'ALL CLASSES IN SCHOOL'
  }

  let finalCategory = classForm.value.category
  if (finalCategory === '[UNCATEGORIZED]') finalCategory = ''
  
  const finalReason = `${finalCategory} ${classForm.value.eventName.trim()}`.trim().toUpperCase()

  try {
    const { error } = await supabase.from('mmi_interruptions').insert({
      interruption_date: classForm.value.date,
      type: 'class',
      target_display: targetDisplay,
      start_period: classForm.value.startPeriod,
      end_period: classForm.value.endPeriod,
      reason: finalReason,
      remarks: classForm.value.remarks ? classForm.value.remarks.toUpperCase() : ''
    })

    if (error) throw error

    toast.success("CLASS INTERRUPTION RECORD SAVED SUCCESSFULLY!")
    fetchLogs()
    classForm.value.eventName = ''
  } catch (err) {
    toast.error("FAILED TO SAVE: " + err.message)
  }
}

const submitTeacherInterruption = async () => {
  if (exportedSubjects.value.length === 0) return

  if (!teacherForm.value.eventName.trim()) {
    return toast.error("PLEASE SPECIFY ABSENCE DETAILS!")
  }

  const teacher = teachersList.value.find(t => t.id === teacherForm.value.teacherId)
  const periods = exportedSubjects.value.map(s => Number(s.period)).sort((a,b) => a-b)
  const startP = periods[0] || 1
  const endP = periods[periods.length - 1] || 1

  const subjectSummary = exportedSubjects.value.map(s => `${s.class_name}(${s.subject})`).join(', ')

  let finalCategory = teacherForm.value.category
  if (finalCategory === '[UNCATEGORIZED]') finalCategory = ''
  
  const finalReason = `${finalCategory} ${teacherForm.value.eventName.trim()}`.trim().toUpperCase()

  try {
    const { error } = await supabase.from('mmi_interruptions').insert({
      interruption_date: teacherForm.value.date,
      type: 'teacher',
      target_display: `TEACHER: ${teacher?.name || ''}`,
      start_period: startP,
      end_period: endP,
      reason: finalReason,
      remarks: `(INVOLVING SLOTS: PERIOD ${periods.join(', ')} | SUBJECTS: ${subjectSummary})`
    })

    if (error) throw error

    toast.success("TEACHER INTERRUPTION RECORD SAVED SUCCESSFULLY!")
    fetchLogs()
    teacherForm.value.eventName = ''
  } catch (err) {
    toast.error("FAILED TO SAVE: " + err.message)
  }
}

const fetchLogs = async () => {
  const { data } = await supabase
    .from('mmi_interruptions')
    .select('*')
    .order('interruption_date', { ascending: false })
  
  interruptionLogs.value = data || []
}

const exportLogsToExcel = () => {
  if (filteredLogs.value.length === 0) {
    return toast.error("NO DATA TO EXPORT!")
  }

  let csvContent = "\uFEFFDATE,TYPE,TARGET,TIME SLOT,REASON,NOTES\n"
  filteredLogs.value.forEach(item => {
    const typeStr = item.type === 'class' ? 'CLASS' : 'TEACHER'
    const row = [
      item.interruption_date,
      typeStr,
      `"${(item.target_display || '').replace(/"/g, '""')}"`,
      `"PERIOD ${item.start_period}-${item.end_period}"`,
      `"${(item.reason || '').replace(/"/g, '""')}"`,
      `"${(item.remarks || '').replace(/"/g, '""')}"`
    ]
    csvContent += row.join(",") + "\n"
  })

  const blob = new Blob([csvContent], { type: 'text/csv;charset=utf-8;' })
  const url = URL.createObjectURL(blob)
  const link = document.createElement('a')
  link.setAttribute('href', url)
  link.setAttribute('download', `MMI_INTERRUPTION_HISTORY_${getLocalToday()}.csv`)
  document.body.appendChild(link)
  link.click()
  document.body.removeChild(link)
  toast.success("REPORT EXPORTED SUCCESSFULLY!")
}

// 🌟 三语兼容的表格格式化
const formatTargetDisplay = (text) => {
  if (!text) return ''
  return text.replace(/^(KELAS|CLASS|班级)[:：]\s*/i, '').trim()
}

const deleteLog = async (log) => {
  if (!confirm(`ARE YOU SURE YOU WANT TO DELETE THIS MMI INTERRUPTION RECORD ON ${log.interruption_date}?`)) return

  try {
    const { error: mmiErr } = await supabase
      .from('mmi_interruptions')
      .delete()
      .eq('id', log.id)

    if (mmiErr) throw mmiErr

    // 🌟 三语兼容的教师判断
    if (log.type === 'teacher' || (log.target_display && (log.target_display.includes('GURU') || log.target_display.includes('TEACHER') || log.target_display.includes('教师')))) {
      let teacherName = ''
      if (log.target_display) {
        teacherName = log.target_display.replace(/(?:GURU|TEACHER|教师)[:：]?\s*/i, '').trim()
      }

      if (teacherName) {
        const { data: teacherObj } = await supabase
          .from('teachers')
          .select('id')
          .eq('name', teacherName)
          .single()

        if (teacherObj) {
          const { data: leaveReqs } = await supabase
            .from('leave_requests')
            .select('id')
            .eq('teacher_id', teacherObj.id)
            .eq('leave_date', log.interruption_date)

          if (leaveReqs && leaveReqs.length > 0) {
            const leaveIds = leaveReqs.map(l => l.id)

            await supabase
              .from('substitute_assignments')
              .delete()
              .in('leave_request_id', leaveIds)

            await supabase
              .from('leave_requests')
              .delete()
              .in('id', leaveIds)
          }
        }
      }
    }

    toast.success("INTERRUPTION AND RELATED LEAVE RECORDS DELETED SUCCESSFULLY!")
    fetchLogs()
  } catch (err) {
    toast.error("FAILED TO DELETE: " + err.message)
  }
}

onMounted(() => {
  const today = getLocalToday()
  classForm.value.date = today
  teacherForm.value.date = today

  loadTeachers()
  fetchLogs()
  fetchClasses()
})

onActivated(() => {
  const today = getLocalToday()
  classForm.value.date = today
  teacherForm.value.date = today
})
</script>