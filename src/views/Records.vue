<template>
  <!-- 保持 min-w-[1024px] 确保表格不会无限变宽 -->
  <div class="p-4 sm:p-8 mx-auto min-h-screen space-y-8 min-w-[1024px] print:p-0 print:min-w-0 print:w-auto print:m-0 print:space-y-0">
    
    <!-- Screen Action Bar (Automatically hidden during printing) -->
    <div class="print:hidden bg-white rounded-3xl p-6 sm:p-8 shadow-sm ring-1 ring-slate-900/5 flex flex-col gap-6">
      
      <!-- Title & Subtitle -->
      <div class="space-y-2 max-w-4xl">
        <h1 class="text-2xl sm:text-3xl font-extrabold tracking-tight text-transparent bg-clip-text bg-gradient-to-r from-slate-900 via-indigo-800 to-violet-800 whitespace-nowrap flex items-center gap-3">
          <UsersRound class="w-8 h-8 text-indigo-700 shrink-0" />
          DAILY SUBSTITUTE TEACHER MANAGEMENT
        </h1>
        <p class="text-slate-500 text-xs sm:text-sm font-medium leading-relaxed whitespace-nowrap">
          CLICK TIMETABLE CELLS TO ASSIGN SUBSTITUTE TEACHERS, SUPPORTS ONE-CLICK SMART AUTOMATIC SCHEDULING
        </p>
      </div>

      <!-- Action Buttons Bar -->
      <div class="flex flex-wrap items-center gap-4 pt-4 border-t border-slate-100">
        
        <!-- 1. Smart Auto-Scheduling -->
        <button 
          @click="handleAutoAssignAll"
          :disabled="isAutoAssigning"
          class="bg-indigo-600 hover:bg-indigo-700 disabled:opacity-50 text-white px-5 h-11 rounded-2xl text-xs font-bold shadow-sm transition-all flex items-center justify-center gap-2 cursor-pointer shrink-0 whitespace-nowrap"
        >
          <span v-if="isAutoAssigning" class="w-3.5 h-3.5 border-2 border-white/30 border-t-white rounded-full animate-spin"></span>
          <Zap v-else class="w-4 h-4" />
          <span>⚡ SMART SUBSTITUTE SCHEDULING</span>
        </button>

        <!-- 2. Session Switcher Tabs (MORNING / AFTERNOON) -->
        <div class="flex bg-slate-100 p-1.5 rounded-2xl ring-1 ring-slate-900/5 h-11 items-center shrink-0 shadow-inner">
          <button 
            @click="currentSession = 'morning'" 
            :class="currentSession === 'morning' ? 'bg-slate-900 text-white shadow-sm' : 'text-slate-600 hover:text-slate-900'"
            class="px-5 py-2 rounded-xl text-xs font-bold transition-all cursor-pointer flex items-center justify-center gap-2 whitespace-nowrap"
          >
            <Sun class="w-4 h-4 text-amber-500" /> MORNING SESSION
          </button>
          <button 
            @click="currentSession = 'afternoon'" 
            :class="currentSession === 'afternoon' ? 'bg-slate-900 text-white shadow-sm' : 'text-slate-600 hover:text-slate-900'"
            class="px-5 py-2 rounded-xl text-xs font-bold transition-all cursor-pointer flex items-center justify-center gap-2 whitespace-nowrap"
          >
            <Moon class="w-4 h-4 text-indigo-400" /> AFTERNOON SESSION
          </button>
        </div>

        <!-- 3. Date Picker -->
        <div class="flex items-center gap-2 bg-slate-50 px-4 h-11 rounded-2xl border border-slate-200/80 shadow-2xs shrink-0">
          <span class="text-xs font-bold text-slate-500 whitespace-nowrap">SELECT DATE:</span>
          <input 
            type="date" 
            v-model="targetDate" 
            class="bg-transparent text-xs font-bold text-slate-800 focus:outline-none cursor-pointer"
          />
        </div>

        <!-- 4. Direct PDF Export -->
        <button 
          @click="handleExportPdf"
          :disabled="isExportingPdf"
          class="bg-slate-900 hover:bg-slate-800 disabled:opacity-50 text-white px-6 h-11 rounded-2xl text-xs font-bold shadow-sm transition-all flex items-center justify-center gap-2 shrink-0 cursor-pointer whitespace-nowrap"
        >
          <span v-if="isExportingPdf" class="w-4 h-4 border-2 border-white/30 border-t-white rounded-full animate-spin"></span>
          <Download v-else class="w-4 h-4" />
          <span>{{ isExportingPdf ? 'GENERATING PDF...' : 'DOWNLOAD PDF' }}</span>
        </button>

      </div>
    </div>

    <!-- Main Tables: Automatically paginated to 5 rows per page -->
    <div 
      v-for="(pageTeachers, pageIndex) in paginatedTeacherPages" 
      :key="pageIndex"
      :class="[
        'print-main-sheet bg-white rounded-3xl shadow-sm ring-1 ring-slate-900/5 p-8 print:shadow-none print:ring-0 print:p-0 print:rounded-none',
        pageIndex > 0 ? 'mt-12 print:mt-0' : ''
      ]"
    >
      <!-- Auto page-break for printing if not the first page -->
      <div v-if="pageIndex > 0" class="print-page-break" aria-hidden="true"></div>

      <div class="text-center mb-6 print:mb-2">
        <h2 class="text-xl font-black tracking-wider text-black font-serif">
          {{ schoolName || 'SJK (C) LADANG GRISEK' }}
        </h2>

        <h3 class="text-lg font-bold tracking-widest text-black mt-1 font-serif underline uppercase">
          SUBSTITUTE TEACHER TIMETABLE ({{ currentSession === 'morning' ? 'MORNING SESSION' : 'AFTERNOON SESSION' }})
        </h3>
      </div>

      <div class="flex justify-between items-center mb-4 print:mb-2 font-bold text-sm font-serif border-b-2 border-black pb-2 print:pb-1">
        <div>
          <span class="underline underline-offset-4">DATE :</span> <span class="ml-2 border-b border-black px-4">{{ formattedDate }}</span>
        </div>
        <div>
          <span class="underline underline-offset-4">DAY :</span> <span class="ml-2 border-b border-black px-4 uppercase">{{ formattedDayName }}</span>
        </div>
      </div>

      <div class="overflow-x-auto print:overflow-visible">
        <table class="w-full border-collapse border-2 border-black text-center text-xs font-serif table-fixed">
          <thead>
            <tr class="bg-slate-100 print:bg-white">
              <th class="border border-black p-1 font-bold" colspan="2" style="width: 130px; min-width: 130px; max-width: 130px;">TIME</th>

              <th
                v-for="(time, index) in currentPeriodTimes"
                :key="index"
                class="border border-black p-1"
              >
                <div class="font-bold">{{ index + 1 }}</div>
                <div class="text-[7px] font-normal mt-0.5 truncate">{{ time }}</div>
              </th>
            </tr>
          </thead>

          <tbody
            v-for="slotIndex in 5"
            :key="slotIndex"
            style="page-break-inside: avoid; break-inside: avoid;"
            class="print:break-inside-avoid"
          >
            
            <template v-if="pageTeachers[slotIndex - 1]">
              <tr>
                <td
                  class="border border-black p-1 bg-slate-50 print:bg-white align-middle text-center"
                  rowspan="3"
                  style="width: 85px; max-width: 85px;"
                >
                  <div class="flex flex-col items-center justify-center w-full px-0.5">
                    <span
                      class="uppercase font-bold w-full text-center whitespace-normal"
                      :style="getDynamicStyle(pageTeachers[slotIndex - 1].name, 10)"
                    >
                      {{ pageTeachers[slotIndex - 1].name }}
                    </span>

                    <span
                      v-if="pageTeachers[slotIndex - 1].reason"
                      class="font-normal text-slate-600 w-full text-center tracking-tighter mt-0.5 uppercase whitespace-normal"
                      :style="getDynamicStyle(`(${pageTeachers[slotIndex - 1].reason})`, 8.5)"
                    >
                      ({{ pageTeachers[slotIndex - 1].reason }})
                    </span>
                  </div>
                </td>

                <td
                  class="border border-black p-1 font-bold bg-slate-50 print:bg-white text-[10px]"
                  style="width: 45px; max-width: 45px;"
                >
                  CLASS
                </td>
                
                <td
                  v-for="p in currentPeriodTimes.length"
                  :key="p"
                  class="border border-black p-0.5 font-semibold align-middle h-8"
                  style="max-width: 0;"
                >
                  <div class="w-full h-full flex items-center justify-center px-0.5 overflow-hidden">
                    <span class="block w-full text-center text-[10px] tracking-tighter leading-tight text-slate-800 whitespace-normal">
                      {{ getTeacherPeriodData(pageTeachers[slotIndex - 1].id, p, 'class_subject') }}
                    </span>
                  </div>
                </td>
              </tr>

              <tr>
                <td class="border border-black p-0.5 font-bold bg-slate-50 print:bg-white text-[5.2px] tracking-tighter whitespace-nowrap">
                  SUBSTITUTE
                </td>

                <td
                  v-for="p in currentPeriodTimes.length"
                  :key="p"
                  @click="hasLeavePeriod(pageTeachers[slotIndex - 1].id, p) ? handleCellClick(pageTeachers[slotIndex - 1].id, p) : null"
                  :class="hasLeavePeriod(pageTeachers[slotIndex - 1].id, p) ? 'cursor-pointer hover:bg-indigo-50 group' : ''"
                  class="print:hover:bg-transparent border border-black p-0.5 font-bold text-indigo-900 align-middle h-8 transition relative"
                  style="max-width: 0;"
                >
                  <div class="w-full h-full flex items-center justify-center px-0.5 overflow-hidden">
                    <span class="block w-full text-center text-[9px] tracking-tighter leading-tight whitespace-normal">
                      {{ getTeacherPeriodData(pageTeachers[slotIndex - 1].id, p, 'substitute_name') }}
                    </span>

                    <span
                      v-if="hasLeavePeriod(pageTeachers[slotIndex - 1].id, p)"
                      class="print:hidden hidden group-hover:inline-block text-[9px] text-indigo-500 absolute right-1"
                    >
                      ✏️
                    </span>
                  </div>
                </td>
              </tr>

              <tr>
                <td class="border border-black p-0.5 font-bold bg-slate-50 print:bg-white text-[5.8px] tracking-tighter whitespace-nowrap">
                  SIGNATURE
                </td>

                <td
                  v-for="p in currentPeriodTimes.length"
                  :key="p"
                  class="border border-black p-1 align-middle h-8"
                ></td>
              </tr>
            </template>

            <!-- Manual entry rows (shown if fewer than 5 absent teachers) -->
            <template v-else>
              <tr>
                <!-- ⭐️ Class extract / Clear buttons -->
                <td
                  class="border border-black p-0 font-bold bg-slate-50 print:bg-white align-middle text-center h-8 relative group"
                  :style="{ width: '85px', maxWidth: '85px' }"
                  rowspan="3"
                >
                  <div class="w-full h-full relative flex items-center justify-center min-h-[70px]">
                    <div
                      contenteditable="true"
                      @blur="saveManualEntry(`page_${pageIndex}_${slotIndex}`, 'name', 0, $event)"
                      v-text="getManualEntry(`page_${pageIndex}_${slotIndex}`, 'name', 0)"
                      class="w-full h-full outline-none focus:bg-indigo-50/50 hover:bg-slate-100 cursor-text transition-colors whitespace-pre-wrap leading-tight uppercase flex items-center justify-center p-1"
                      :style="getDynamicStyle(getManualEntry(`page_${pageIndex}_${slotIndex}`, 'name', 0), 10)"
                    ></div>

                    <div class="print:hidden absolute right-0 top-0 hidden group-hover:flex flex-col z-10 gap-[1px]">
                      <button
                        contenteditable="false"
                        @click.stop="openClassPicker(pageIndex, slotIndex, null)"
                        class="bg-emerald-500 text-white rounded-bl px-1.5 py-0.5 text-[9px] cursor-pointer shadow-sm hover:bg-emerald-600 font-sans tracking-widest font-bold"
                      >
                        CLASS
                      </button>
                      <button
                        contenteditable="false"
                        @click.stop="clearManualRow(pageIndex, slotIndex, null)"
                        class="bg-red-500 text-white rounded-l px-1.5 py-0.5 text-[9px] cursor-pointer shadow-sm hover:bg-red-600 font-sans tracking-widest font-bold"
                      >
                        CLEAR
                      </button>
                    </div>
                  </div>
                </td>

                <td
                  class="border border-black p-1 font-bold bg-slate-50 print:bg-white text-[10px]"
                  style="width: 45px; max-width: 45px;"
                >
                  CLASS
                </td>

                <td
                  v-for="p in currentPeriodTimes.length"
                  :key="'kelas-'+p"
                  contenteditable="true"
                  @blur="saveManualEntry(`page_${pageIndex}_${slotIndex}`, 'kelas', p, $event)"
                  v-text="getManualEntry(`page_${pageIndex}_${slotIndex}`, 'kelas', p)"
                  class="border border-black p-0.5 align-middle h-8 outline-none focus:bg-indigo-50/50 hover:bg-slate-100 cursor-text transition-colors font-semibold text-[11px] whitespace-pre-wrap leading-tight text-center"
                  style="max-width: 0;"
                ></td>
              </tr>

              <tr>
                <td class="border border-black p-0.5 font-bold bg-slate-50 print:bg-white text-[5.2px] tracking-tighter whitespace-nowrap">
                  SUBSTITUTE
                </td>

                <td
                  v-for="p in currentPeriodTimes.length"
                  :key="'ganti-'+p"
                  class="border border-black p-0.5 align-middle h-8 relative group"
                  style="max-width: 0;"
                >
                  <div class="w-full h-full relative flex items-center justify-center">
                    <div
                      contenteditable="true"
                      @blur="saveManualEntry(`page_${pageIndex}_${slotIndex}`, 'ganti', p, $event)"
                      v-text="getManualEntry(`page_${pageIndex}_${slotIndex}`, 'ganti', p)"
                      class="w-full h-full outline-none focus:bg-indigo-50/50 hover:bg-slate-100 cursor-text transition-colors font-bold text-indigo-900 text-[10px] whitespace-pre-wrap leading-tight flex items-center justify-center text-center"
                    ></div>

                    <button
                      contenteditable="false"
                      @click.stop="openBlankModal(`page_${pageIndex}_${slotIndex}`, p, null)"
                      class="print:hidden absolute right-0 top-0 hidden group-hover:flex bg-indigo-500 text-white rounded-bl px-1.5 py-0.5 text-[9px] cursor-pointer shadow-sm hover:bg-indigo-600 z-10 font-sans tracking-widest font-bold"
                    >
                      ASSIGN
                    </button>
                  </div>
                </td>
              </tr>

              <tr>
                <td class="border border-black p-0.5 font-bold bg-slate-50 print:bg-white text-[5.2px] tracking-tighter whitespace-nowrap">
                  SIGNATURE
                </td>

                <td
                  v-for="p in currentPeriodTimes.length"
                  :key="'ttangan-'+p"
                  contenteditable="true"
                  @blur="saveManualEntry(`page_${pageIndex}_${slotIndex}`, 'ttangan', p, $event)"
                  v-text="getManualEntry(`page_${pageIndex}_${slotIndex}`, 'ttangan', p)"
                  class="border border-black p-1 align-middle h-8 outline-none focus:bg-indigo-50/50 hover:bg-slate-100 cursor-text transition-colors"
                ></td>
              </tr>
            </template>
          </tbody>
        </table>
      </div>
    </div>

    <!-- Main Modal: Substitute Assignment Center -->
    <transition
      enter-active-class="transition duration-300 ease-out"
      enter-from-class="opacity-0 scale-95"
      enter-to-class="opacity-100 scale-100"
      leave-active-class="transition duration-200 ease-in"
      leave-from-class="opacity-100 scale-100"
      leave-to-class="opacity-0 scale-95"
    >
      <div
        v-if="showModal"
        class="print:hidden fixed inset-0 z-50 flex items-center justify-center p-4 sm:p-6"
      >
        <div
          class="absolute inset-0 bg-slate-900/30 backdrop-blur-sm"
          @click="showModal = false"
        ></div>

        <div class="relative bg-white rounded-3xl shadow-2xl w-full max-w-2xl overflow-hidden ring-1 ring-slate-900/10 max-h-[90vh] flex flex-col">
          
          <div class="px-8 py-6 border-b border-slate-100 flex justify-between items-center bg-white/50 backdrop-blur-md shrink-0">
            <div>
              <h2 class="text-xl font-bold text-slate-900">SUBSTITUTE ASSIGNMENT CENTER</h2>
              <p class="text-sm text-slate-500 mt-1">
                SUPPORTED BY SMART RECOMMENDATIONS, OR SELECT ANY SAME-SESSION TEACHER MANUALLY BELOW
              </p>
            </div>

            <button
              @click="showModal = false"
              class="text-slate-400 hover:text-slate-600 bg-slate-100 hover:bg-slate-200 rounded-full p-2 transition cursor-pointer"
            >
              ×
            </button>
          </div>
          
          <div class="p-8 bg-slate-50/50 space-y-6 overflow-y-auto">
            <div class="bg-white p-4 rounded-2xl border border-slate-200 shadow-sm">
              <h3 class="text-xs font-bold text-slate-700 mb-3 uppercase tracking-wider">
                🏷️ ASSIGNMENT TYPE:
              </h3>

              <div class="flex flex-col sm:flex-row gap-4">
                <label class="flex items-center gap-2 cursor-pointer bg-slate-50 px-4 py-2 rounded-xl border border-slate-100 hover:bg-indigo-50 transition">
                  <input
                    type="radio"
                    v-model="assignmentType"
                    value="substitute"
                    class="text-indigo-600 focus:ring-indigo-500 w-4 h-4"
                  />
                  <span class="text-sm font-semibold text-slate-800">
                    OFFICIAL SUBSTITUTE
                    <span class="text-xs text-slate-400 font-normal ml-1">
                      (INCLUDED IN WORKLOAD STATS)
                    </span>
                  </span>
                </label>

                <label class="flex items-center gap-2 cursor-pointer bg-slate-50 px-4 py-2 rounded-xl border border-slate-100 hover:bg-indigo-50 transition">
                  <input
                    type="radio"
                    v-model="assignmentType"
                    value="swap"
                    class="text-indigo-600 focus:ring-indigo-500 w-4 h-4"
                  />
                  <span class="text-sm font-semibold text-slate-800">
                    TIMETABLE SWAP
                    <span class="text-xs text-slate-400 font-normal ml-1">
                      (EXCLUDED FROM STATS)
                    </span>
                  </span>
                </label>
              </div>
            </div>
          
            <div class="bg-white p-4 rounded-2xl border border-slate-200 shadow-sm flex items-center gap-3">
              <span class="text-xs font-bold text-slate-700 whitespace-nowrap">
                📍 LOCATION / REMARK:
              </span>

              <input
                v-model="assignmentRemark"
                type="text"
                placeholder="E.G. LIBRARY (IF NEED TO BRING TO LIBRARY OR COMBINE CLASSES)"
                class="w-full px-3.5 py-2.5 bg-slate-50 border border-slate-200 rounded-xl text-xs font-semibold text-slate-800 focus:outline-none focus:ring-2 focus:ring-indigo-500/50"
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

                  <option
                    v-for="t in allSameSessionTeachers"
                    :key="t.id"
                    :value="t.id"
                  >
                    {{ t.name }} <span v-if="t.subject">(SUBJECT: {{ t.subject }})</span>
                  </option>
                </select>

                <button
                  @click="assignSubstitute(manualSelectedTeacherId)"
                  :disabled="!manualSelectedTeacherId"
                  class="w-full sm:w-auto bg-indigo-600 hover:bg-indigo-700 disabled:opacity-50 text-white px-5 py-2.5 rounded-xl text-xs font-semibold shadow-sm transition-all shrink-0 cursor-pointer whitespace-nowrap"
                >
                  CONFIRM MANUAL ASSIGNMENT
                </button>
              </div>
            </div>

            <hr class="border-slate-200" />

            <div>
              <div class="flex justify-between items-center mb-3">
                <h3 class="text-xs font-bold uppercase tracking-wider text-slate-500 flex items-center gap-2">
                  <Sparkles class="w-4 h-4 text-indigo-600" />
                  ✨ SMART RECOMMENDATION CANDIDATES (TOTAL {{ recommendations.length }})
                </h3>
                <span v-if="recommendations.length > 0" class="text-[11px] text-slate-400 font-semibold">
                  PAGE {{ recCurrentPage }} / {{ recTotalPages }}
                </span>
              </div>
              
              <div
                v-if="loadingRecs"
                class="flex flex-col items-center justify-center py-6 space-y-3"
              >
                <div class="w-6 h-6 border-4 border-indigo-500/30 border-t-indigo-600 rounded-full animate-spin"></div>
                <p class="text-xs text-slate-500 font-medium">SMART ALGORITHM IS CALCULATING...</p>
              </div>
              
              <div
                v-else-if="recommendations.length === 0"
                class="bg-white p-4 rounded-2xl border border-slate-200 text-xs text-slate-500 text-center"
              >
                NO AUTOMATIC RECOMMENDATIONS AVAILABLE, PLEASE USE MANUAL ASSIGNMENT ABOVE.
              </div>

              <div v-else class="space-y-3">
                <div
                  v-for="(teacher, index) in paginatedRecommendations"
                  :key="teacher.id"
                  :class="[
                    'group flex flex-col sm:flex-row sm:justify-between sm:items-center p-4 border rounded-2xl transition-all',
                    teacher.isBusy ? 'bg-red-50/30 border-red-100' : 'bg-white border-slate-200 hover:border-indigo-300 hover:shadow-sm'
                  ]"
                >
                  <div class="flex items-center gap-3 mb-3 sm:mb-0">
                    <div :class="[
                      'w-8 h-8 rounded-full font-extrabold flex items-center justify-center text-xs',
                      teacher.isBusy ? 'bg-red-100 text-red-600' : 'bg-gradient-to-br from-indigo-100 to-violet-100 text-indigo-700'
                    ]">
                      #{{ (recCurrentPage - 1) * recPageSize + index + 1 }}
                    </div>

                    <div>
                      <div class="font-bold text-slate-900 text-sm flex items-center gap-2">
                        {{ teacher.name }}
                        <span v-if="teacher.isBusy" class="text-[10px] text-red-600 bg-red-100 px-2 py-0.5 rounded-full font-bold">
                          HAS CLASS
                        </span>
                      </div>

                      <div class="text-[11px] text-slate-500 mt-1 flex items-center gap-2 flex-wrap">
                        <span>
                          ORIGINAL DAILY CLASSES:
                          <span class="font-bold text-slate-700">
                            {{ teacher.originalClasses }} CLASSES
                          </span>
                        </span>

                        <span>·</span>

                        <span>
                          TODAY'S SUBSTITUTIONS:
                          <span class="font-bold text-orange-600">
                            {{ teacher.todaySubCount }} CLASSES
                          </span>
                        </span>

                        <span>·</span>

                        <span>
                          THIS WEEK'S SUBSTITUTIONS:
                          <span class="font-bold text-slate-700">
                            {{ teacher.currentSubCount }}{{ teacher.currentSubCount !== '-' ? '/' : '' }}{{ teacher.currentSubCount !== '-' ? teacher.max_substitute_per_week : '' }}
                          </span>
                        </span>
                      </div>
                    </div>
                  </div>

                  <button
                    @click="assignSubstitute(teacher.id)"
                    :class="[
                      'px-4 py-2 rounded-xl text-xs font-semibold shadow-sm transition-all cursor-pointer whitespace-nowrap',
                      teacher.isBusy ? 'bg-red-100 text-red-700 hover:bg-red-200' : 'bg-slate-900 hover:bg-indigo-600 text-white'
                    ]"
                  >
                    {{ teacher.isBusy ? 'FORCE ASSIGN' : 'SMART ASSIGN' }}
                  </button>
                </div>

                <!-- Pagination controls -->
                <div v-if="recTotalPages > 1" class="flex items-center justify-between pt-2 px-1">
                  <button 
                    @click="recCurrentPage = Math.max(1, recCurrentPage - 1)"
                    :disabled="recCurrentPage === 1"
                    class="px-3 py-1.5 bg-white border border-slate-200 rounded-xl text-xs font-bold text-slate-700 disabled:opacity-40 disabled:cursor-not-allowed hover:bg-slate-100 transition cursor-pointer"
                  >
                    PREVIOUS
                  </button>
                  <span class="text-xs font-semibold text-slate-500">
                    PAGE {{ recCurrentPage }} / {{ recTotalPages }}
                  </span>
                  <button 
                    @click="recCurrentPage = Math.min(recTotalPages, recCurrentPage + 1)"
                    :disabled="recCurrentPage === recTotalPages"
                    class="px-3 py-1.5 bg-white border border-slate-200 rounded-xl text-xs font-bold text-slate-700 disabled:opacity-40 disabled:cursor-not-allowed hover:bg-slate-100 transition cursor-pointer"
                  >
                    NEXT
                  </button>
                </div>
              </div>
            </div>

            <div
              v-if="currentLeaveItem && substituteAssignmentsMap[currentLeaveItem.id]"
              class="pt-2 border-t border-slate-100 flex justify-between items-center"
            >
              <span class="text-xs text-red-500 font-medium">
                THIS SLOT ALREADY HAS A SUBSTITUTE / SWAP ASSIGNED
              </span>

              <button
                @click="removeAssignment"
                class="text-xs text-red-600 hover:text-red-800 font-bold px-3 py-1 bg-red-50 rounded-lg cursor-pointer whitespace-nowrap"
              >
                CANCEL CURRENT ASSIGNMENT
              </button>
            </div>
          </div>
        </div>
      </div>
    </transition>

    <!-- ⭐️ Additional manual blank sheets -->
    <div
      v-for="(sheet, sIndex) in extraCustomSheets"
      :key="sheet.id"
    >
      <!-- Page break for printing -->
      <div class="print-page-break" aria-hidden="true"></div>

      <div class="print-custom-sheet mt-12 print:mt-0 pt-8 print:pt-0 border-t-4 print:border-none border-dashed border-slate-300">
      
        <div class="print:hidden flex justify-between items-center mb-4 bg-amber-50 p-3 rounded-2xl border border-amber-200">
          <span class="text-xs font-bold text-amber-900">
            📄 ADDITIONAL / MANUAL TIMETABLE #{{ sIndex + 1 }}
          </span>

          <button
            @click="removeCustomSheet(sheet.id)"
            class="text-xs text-red-600 bg-white hover:bg-red-50 px-3 py-1.5 rounded-xl font-bold shadow-sm transition cursor-pointer whitespace-nowrap"
          >
            DELETE THIS TIMETABLE
          </button>
        </div>

        <div class="bg-white rounded-3xl shadow-sm ring-1 ring-slate-900/5 p-8 print:shadow-none print:ring-0 print:p-0 print:rounded-none print:break-inside-avoid">
          
          <div class="text-center mb-6 print:mb-2">
            <h2 class="text-xl font-black tracking-wider text-black font-serif">
              {{ schoolName || 'SJK (C) LADANG GRISEK' }}
            </h2>

            <h3 class="text-lg font-bold tracking-widest text-black mt-1 font-serif underline uppercase">
              SUBSTITUTE TEACHER TIMETABLE ({{ currentSession === 'morning' ? 'MORNING SESSION' : 'AFTERNOON SESSION' }})
            </h3>
          </div>

          <div class="flex justify-between items-center mb-4 print:mb-2 font-bold text-sm font-serif border-b-2 border-black pb-2 print:pb-1">
            <div>
              <span class="underline underline-offset-4">DATE :</span>

              <input
                v-model="sheet.date"
                @blur="saveCustomSheetsToCloud"
                type="text"
                placeholder="DATE"
                class="ml-2 border-b border-black px-2 py-0.5 text-sm font-normal w-32 focus:outline-none"
              />
            </div>

            <div>
              <span class="underline underline-offset-4">DAY :</span>

              <input
                v-model="sheet.day"
                @blur="saveCustomSheetsToCloud"
                type="text"
                placeholder="DAY"
                class="ml-2 border-b border-black px-2 py-0.5 text-sm font-normal w-28 uppercase focus:outline-none"
              />
            </div>
          </div>

          <div class="w-full overflow-x-auto print:overflow-visible">
            <table class="w-full border-collapse border-2 border-black text-center text-xs font-serif table-fixed">
              <thead>
                <tr class="bg-slate-100 print:bg-white">
                  <th
                    class="border border-black p-1 font-bold"
                    colspan="2"
                    style="width: 130px; min-width: 130px; max-width: 130px;"
                  >
                    TIME
                  </th>

                  <th
                    v-for="(time, index) in currentPeriodTimes"
                    :key="index"
                    class="border border-black p-1"
                  >
                    <div class="font-bold">{{ index + 1 }}</div>
                    <div class="text-[7px] font-normal mt-0.5 truncate">
                      {{ time }}
                    </div>
                  </th>
                </tr>
              </thead>

              <tbody
                v-for="slotIndex in 5"
                :key="slotIndex"
              >
                <tr>
                  <!-- ⭐ Buttons (Additional manual sheet) -->
                  <td
                    class="border border-black p-0 font-bold bg-slate-50 print:bg-white align-middle text-center h-8 relative group"
                    :style="{ width: '85px', maxWidth: '85px' }"
                    rowspan="3"
                  >
                    <div class="w-full h-full relative flex items-center justify-center min-h-[70px]">
                      <div
                        contenteditable="true"
                        @blur="saveManualEntry(`sheet_${sheet.id}_${slotIndex}`, 'name', 0, $event)"
                        v-text="getManualEntry(`sheet_${sheet.id}_${slotIndex}`, 'name', 0)"
                        class="w-full h-full outline-none focus:bg-indigo-50/50 hover:bg-slate-100 cursor-text transition-colors whitespace-pre-wrap leading-tight uppercase flex items-center justify-center p-1"
                        :style="getDynamicStyle(getManualEntry(`sheet_${sheet.id}_${slotIndex}`, 'name', 0), 10)"
                      ></div>

                      <div class="print:hidden absolute right-0 top-0 hidden group-hover:flex flex-col z-10 gap-[1px]">
                        <button
                          contenteditable="false"
                          @click.stop="openClassPicker(null, slotIndex, sheet.id)"
                          class="bg-emerald-500 text-white rounded-bl px-1.5 py-0.5 text-[9px] cursor-pointer shadow-sm hover:bg-emerald-600 font-sans tracking-widest font-bold"
                        >
                          CLASS
                        </button>
                        <button
                          contenteditable="false"
                          @click.stop="clearManualRow(null, slotIndex, sheet.id)"
                          class="bg-red-500 text-white rounded-l px-1.5 py-0.5 text-[9px] cursor-pointer shadow-sm hover:bg-red-600 font-sans tracking-widest font-bold"
                        >
                          CLEAR
                        </button>
                      </div>
                    </div>
                  </td>

                  <td
                    class="border border-black p-1 font-bold bg-slate-50 print:bg-white text-[10px]"
                    style="width: 45px; max-width: 45px;"
                  >
                    CLASS
                  </td>

                  <td
                    v-for="p in currentPeriodTimes.length"
                    :key="'kelas-'+p"
                    contenteditable="true"
                    @blur="saveManualEntry(`sheet_${sheet.id}_${slotIndex}`, 'kelas', p, $event)"
                    v-text="getManualEntry(`sheet_${sheet.id}_${slotIndex}`, 'kelas', p)"
                    class="border border-black p-0.5 align-middle h-8 outline-none focus:bg-indigo-50/50 hover:bg-slate-100 cursor-text transition-colors font-semibold text-[11px] whitespace-pre-wrap leading-tight text-center"
                    style="max-width: 0;"
                  ></td>
                </tr>

                <tr>
                  <td class="border border-black p-0.5 font-bold bg-slate-50 print:bg-white text-[5.2px] tracking-tighter whitespace-nowrap">
                    SUBSTITUTE
                  </td>

                  <td
                    v-for="p in currentPeriodTimes.length"
                    :key="'ganti-'+p"
                    class="border border-black p-0.5 align-middle h-8 relative group"
                    style="max-width: 0;"
                  >
                    <div class="w-full h-full relative flex items-center justify-center">
                      <div
                        contenteditable="true"
                        @blur="saveManualEntry(`sheet_${sheet.id}_${slotIndex}`, 'ganti', p, $event)"
                        v-text="getManualEntry(`sheet_${sheet.id}_${slotIndex}`, 'ganti', p)"
                        class="w-full h-full outline-none focus:bg-indigo-50/50 hover:bg-slate-100 cursor-text transition-colors font-bold text-[10px] text-indigo-900 whitespace-pre-wrap leading-tight flex items-center justify-center text-center"
                      ></div>

                      <button
                        contenteditable="false"
                        @click.stop="openBlankModal(slotIndex, p, sheet.id)"
                        class="print:hidden absolute right-0 top-0 hidden group-hover:flex bg-indigo-500 text-white rounded-bl px-1.5 py-0.5 text-[9px] cursor-pointer shadow-sm hover:bg-indigo-600 z-10 font-sans tracking-widest font-bold"
                      >
                        ASSIGN
                      </button>
                    </div>
                  </td>
                </tr>

                <tr>
                  <td class="border border-black p-0.5 font-bold bg-slate-50 print:bg-white text-[5.2px] tracking-tighter whitespace-nowrap">
                    SIGNATURE
                  </td>

                  <td
                    v-for="p in currentPeriodTimes.length"
                    :key="'ttangan-'+p"
                    contenteditable="true"
                    @blur="saveManualEntry(`sheet_${sheet.id}_${slotIndex}`, 'ttangan', p, $event)"
                    v-text="getManualEntry(`sheet_${sheet.id}_${slotIndex}`, 'ttangan', p)"
                    class="border border-black p-1 align-middle h-8 outline-none focus:bg-indigo-50/50 hover:bg-slate-100 cursor-text transition-colors"
                  ></td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>
      </div>
    </div>

    <!-- Bottom Add Button -->
    <div class="print:hidden mt-8 mb-12 flex justify-center w-full">
      <button
        @click="addBlankSheet"
        class="flex items-center gap-2 bg-slate-900 hover:bg-indigo-600 text-white px-8 py-3.5 rounded-2xl text-xs font-bold shadow-md transition-all cursor-pointer whitespace-nowrap"
      >
        <span class="text-base font-extrabold">+</span>
        ADD AN OFFICIAL BLANK TIMETABLE SHEET
      </button>
    </div>

    <!-- ⭐️ Modal for extracting full day class subjects -->
    <transition
      enter-active-class="transition duration-300 ease-out"
      enter-from-class="opacity-0 scale-95"
      enter-to-class="opacity-100 scale-100"
      leave-active-class="transition duration-200 ease-in"
      leave-from-class="opacity-100 scale-100"
      leave-to-class="opacity-0 scale-95"
    >
      <div
        v-if="showClassPickerModal"
        class="print:hidden fixed inset-0 z-50 flex items-center justify-center p-4 sm:p-6"
      >
        <div class="absolute inset-0 bg-slate-900/30 backdrop-blur-sm" @click="showClassPickerModal = false"></div>
        <div class="relative bg-white rounded-3xl shadow-2xl w-full max-w-sm overflow-hidden ring-1 ring-slate-900/10">
          
          <div class="px-6 py-5 border-b border-slate-100 flex justify-between items-center bg-slate-50">
            <h2 class="text-lg font-bold text-slate-900 flex items-center gap-2">
              <span>📚 EXTRACT FULL DAY CLASS SUBJECTS</span>
            </h2>
            <button @click="showClassPickerModal = false" class="text-slate-400 hover:text-slate-600 bg-white hover:bg-slate-200 rounded-full w-8 h-8 flex items-center justify-center transition cursor-pointer font-bold">
              ✕
            </button>
          </div>
          
          <div class="p-6 space-y-5">
            <div>
              <label class="block text-xs font-bold text-slate-700 mb-2">SELECT CLASS TO EXTRACT:</label>
              <select v-model="selectedClassToFill" class="w-full px-3.5 py-2.5 bg-slate-50 border border-slate-200 rounded-xl text-xs font-semibold text-slate-800 focus:outline-none focus:ring-2 focus:ring-emerald-500 cursor-pointer">
                <option value="">-- PLEASE SELECT A CLASS --</option>
                <option v-for="cls in allClassesList" :key="cls" :value="cls">{{ cls }}</option>
              </select>
            </div>
          </div>

          <div class="px-6 py-4 bg-slate-50 border-t border-slate-100 flex justify-end gap-3">
            <button @click="showClassPickerModal = false" class="text-slate-500 hover:text-slate-700 px-4 py-2 text-xs font-bold transition cursor-pointer">
              CANCEL
            </button>
            <button @click="confirmClassPicker" class="bg-emerald-600 hover:bg-emerald-700 text-white px-5 py-2 rounded-xl text-xs font-bold shadow-sm transition-all cursor-pointer">
              CONFIRM EXTRACTION
            </button>
          </div>

        </div>
      </div>
    </transition>

    <!-- Quick Assignment Modal -->
    <transition
      enter-active-class="transition duration-300 ease-out"
      enter-from-class="opacity-0 scale-95"
      enter-to-class="opacity-100 scale-100"
      leave-active-class="transition duration-200 ease-in"
      leave-from-class="opacity-100 scale-100"
      leave-to-class="opacity-0 scale-95"
    >
      <div
        v-if="showBlankModal"
        class="print:hidden fixed inset-0 z-50 flex items-center justify-center p-4 sm:p-6"
      >
        <div
          class="absolute inset-0 bg-slate-900/30 backdrop-blur-sm"
          @click="showBlankModal = false"
        ></div>

        <div class="relative bg-white rounded-3xl shadow-2xl w-full max-w-lg overflow-hidden ring-1 ring-slate-900/10">
          
          <div class="px-6 py-5 border-b border-slate-100 flex justify-between items-center bg-slate-50">
            <div>
              <h2 class="text-lg font-bold text-slate-900 flex items-center gap-2">
                <span>📝 QUICK ASSIGN</span>
                <span class="text-[10px] bg-indigo-100 text-indigo-700 px-2 py-0.5 rounded-full">
                  SPECIAL TASK
                </span>
              </h2>
            </div>

            <button
              @click="showBlankModal = false"
              class="text-slate-400 hover:text-slate-600 bg-white hover:bg-slate-200 rounded-full w-8 h-8 flex items-center justify-center transition cursor-pointer font-bold"
            >
              ✕
            </button>
          </div>
          
          <div class="p-6 space-y-5">
            <div>
              <label class="block text-xs font-bold text-slate-700 mb-2">
                🧑‍🏫 SELECT SUBSTITUTE TEACHER (SAME SESSION):
              </label>

              <select
                v-model="blankForm.teacherId"
                class="w-full px-3.5 py-2.5 bg-slate-50 border border-slate-200 rounded-xl text-xs font-semibold text-slate-800 focus:outline-none focus:ring-2 focus:ring-indigo-500 cursor-pointer"
              >
                <option value="">-- NONE (LEAVE BLANK / TEXT ONLY) --</option>

                <option
                  v-for="t in allSameSessionTeachers"
                  :key="t.id"
                  :value="t.id"
                >
                  {{ t.name }} <span v-if="t.subject">({{ t.subject }})</span>
                </option>
              </select>
            </div>

            <div>
              <label class="block text-xs font-bold text-slate-700 mb-2">
                📍 REMARK (LOCATION/TASK E.G. DUTY):
              </label>

              <input
                v-model="blankForm.remark"
                type="text"
                placeholder="E.G.: PERPUSTAKAAN / LATIHAN SUKAN"
                class="w-full px-3.5 py-2.5 bg-slate-50 border border-slate-200 rounded-xl text-xs font-semibold text-slate-800 focus:outline-none focus:ring-2 focus:ring-indigo-500/50"
              />
            </div>

            <div class="bg-indigo-50/50 p-4 rounded-xl border border-indigo-100 flex items-start gap-3">
              <input
                type="checkbox"
                v-model="blankForm.kiraBeban"
                id="kiraBebanCb"
                class="mt-0.5 w-4 h-4 text-indigo-600 rounded cursor-pointer"
              />

              <div class="flex-1">
                <label
                  for="kiraBebanCb"
                  class="text-sm font-bold text-slate-800 cursor-pointer block mb-1"
                >
                  INCLUDE IN TEACHER'S WORKLOAD
                </label>

                <p class="text-[10px] text-slate-500 font-medium leading-relaxed">
                  If checked, the system will create a virtual record (does not affect MMI loss reports) and increase the teacher's daily class count by +1 in the background.
                </p>
              </div>
            </div>
          </div>

          <div class="px-6 py-4 bg-slate-50 border-t border-slate-100 flex justify-between items-center">
            <button
              v-if="hasExistingVirtual"
              @click="removeBlankAssignment"
              class="text-xs text-red-600 hover:text-red-800 font-bold px-4 py-2 bg-red-50 hover:bg-red-100 rounded-xl cursor-pointer transition whitespace-nowrap"
            >
              CLEAR CELL
            </button>

            <div v-else></div>

            <div class="flex gap-3">
              <button
                @click="showBlankModal = false"
                class="text-slate-500 hover:text-slate-700 px-4 py-2 text-xs font-bold transition cursor-pointer"
              >
                CANCEL
              </button>

              <button
                @click="confirmBlankAssignment"
                class="bg-indigo-600 hover:bg-indigo-700 text-white px-5 py-2 rounded-xl text-xs font-bold shadow-sm transition-all cursor-pointer"
              >
                CONFIRM
              </button>
            </div>
          </div>

        </div>
      </div>
    </transition>

  </div>
</template>

<script setup>
import { ref, onMounted, computed, watch } from 'vue'
import { supabase } from '../services/supabase'
import { recommendSubstitute } from '../utils/algorithm'
import { useToast } from '../utils/toast'
import { 
  UsersRound, 
  Sun, 
  Moon, 
  Zap, 
  Printer, 
  Download,
  Sparkles 
} from 'lucide-vue-next'

const toast = useToast()

const targetDate = ref(new Date().toISOString().split('T')[0])
const currentSession = ref('morning')

/*
 * School Name
 * Source: Supabase -> school_settings -> id = 1
 * fallback: localStorage.school_name
 */
const schoolName = ref('')

const morningTimes = [
  '7.00-7.30',
  '7.30-8.00',
  '8.00-8.30',
  '8.30-9.00',
  '9.00-9.30',
  '10.00-10.30',
  '10.30-11.00',
  '11.00-11.30',
  '11.30-12.00',
  '12.00-12.30',
  '12.30-1.00'
]

const afternoonTimes = [
  '1.00-1.30',
  '1.30-2.00',
  '2.00-2.30',
  '2.30-3.00',
  '3.00-3.30',
  '3.50-4.20',
  '4.20-4.50',
  '4.50-5.20',
  '5.20-5.50',
  '5.50-6.20'
]

const currentPeriodTimes = computed(() =>
  currentSession.value === 'morning'
    ? morningTimes
    : afternoonTimes
)

const dayNames = [
  'Sunday',
  'Monday',
  'Tuesday',
  'Wednesday',
  'Thursday',
  'Friday',
  'Saturday'
]

const leaveRequests = ref([])
const substituteAssignmentsMap = ref({})
const teachersMap = ref({})
const allSameSessionTeachers = ref([])

// Main Modal Status
const showModal = ref(false)
const loadingRecs = ref(false)
const recommendations = ref([])
const currentLeaveItem = ref(null)
const assignmentRemark = ref('')
const assignmentType = ref('substitute')
const manualSelectedTeacherId = ref('')
const isAutoAssigning = ref(false)
const isExportingPdf = ref(false)

// Smart recommendation pagination status (max 10 per page)
const recCurrentPage = ref(1)
const recPageSize = 10

const recTotalPages = computed(() => {
  return Math.max(1, Math.ceil(recommendations.value.length / recPageSize))
})

const paginatedRecommendations = computed(() => {
  const start = (recCurrentPage.value - 1) * recPageSize
  return recommendations.value.slice(start, start + recPageSize)
})

// Blank row modal status
const showBlankModal = ref(false)
const blankTarget = ref({
  slot: null,
  period: null,
  sheetId: null
})
const blankForm = ref({
  teacherId: '',
  remark: '',
  kiraBeban: true
})
const hasExistingVirtual = ref(false)

// Class subject extraction status
const classSchedulesMap = ref({})
const allClassesList = ref([])

// Entire row extraction modal status
const showClassPickerModal = ref(false)
const classPickerTarget = ref({
  pageIndex: null,
  slotIndex: null,
  sheetId: null
})
const selectedClassToFill = ref('')

// =====================================================
// School Identity
// =====================================================

const fetchSchoolIdentity = async () => {
  try {
    const { data, error } = await supabase
      .from('school_settings')
      .select('school_name')
      .eq('id', 1)
      .single()

    if (error) throw error

    schoolName.value =
      data?.school_name?.trim() ||
      localStorage.getItem('school_name')?.trim() ||
      'SJK (C) LADANG GRISEK'

  } catch (err) {
    console.error('Failed to load school name:', err)

    schoolName.value =
      localStorage.getItem('school_name')?.trim() ||
      'SJK (C) LADANG GRISEK'
  }
}

// Logic to dynamically resize the name and absence reason
const getDynamicStyle = (text, baseSize) => {
  if (!text) {
    return {
      fontSize: `${baseSize}px`,
      wordBreak: 'normal',
      overflowWrap: 'normal'
    }
  }
  
  const maxCharsAllowed = 10
  
  const words = String(text)
    .split(/\s+/)
    .filter(Boolean)

  let maxWordLen = 0

  words.forEach(w => {
    let charCount = 0

    for (let i = 0; i < w.length; i++) {
      if (w.charCodeAt(i) > 255) {
        charCount += 1.8
      } else if (/[A-Z]/.test(w[i])) {
        charCount += 1.2
      } else {
        charCount += 1
      }
    }

    if (charCount > maxWordLen) {
      maxWordLen = charCount
    }
  })

  if (maxWordLen <= maxCharsAllowed) {
    return {
      fontSize: `${baseSize}px`,
      lineHeight: '1.15',
      wordBreak: 'normal',
      overflowWrap: 'normal'
    }
  }

  // Downsize scaling mechanism
  const scaledSize =
    baseSize *
    (maxCharsAllowed / maxWordLen) *
    0.95
  
  return {
    fontSize: `${Math.max(4, scaledSize).toFixed(2)}px`,
    lineHeight: '1.1',
    wordBreak: 'normal',
    overflowWrap: 'normal'
  }
}

const formattedDate = computed(() => {
  if (!targetDate.value) return ''
  const [y, m, d] = targetDate.value.split('-')
  return `${d}.${m}.${y}`
})

const formattedDayName = computed(() => {
  if (!targetDate.value) return ''
  const dateObj = new Date(targetDate.value)
  return dayNames[dateObj.getDay()]
})

const displayTeachersList = computed(() => {
  const map = {}

  leaveRequests.value.forEach(req => {
    if (req.class_name === 'VIRTUAL_CLASS') return

    const teacher = teachersMap.value[req.teacher_id]

    if (
      teacher &&
      (teacher.session || 'morning') === currentSession.value
    ) {
      let cleanReason = (req.reason || '')
        .replace(/\[.*?\]\s*/, '')

      map[req.teacher_id] = {
        id: req.teacher_id,
        name: teacher.name,
        reason: cleanReason
      }
    }
  })

  return Object.values(map)
})

const paginatedTeacherPages = computed(() => {
  const list = displayTeachersList.value
  const pages = []
  const pageCount = Math.max(1, Math.ceil(list.length / 5))
  for (let i = 0; i < pageCount; i++) {
    pages.push(list.slice(i * 5, (i + 1) * 5))
  }
  return pages
})

// ================= Manual Drafts & Additional Timetables (Cloud Sync) =================

const manualEntries = ref({})

const sessionCustomSheets = ref({
  morning: [],
  afternoon: []
})

const fetchManualDrafts = async () => {
  manualEntries.value = {}
  sessionCustomSheets.value[currentSession.value] = []

  try {
    const { data, error } = await supabase
      .from('jadual_manual_drafts')
      .select('draft_data')
      .eq('target_date', targetDate.value)
      .eq('session', currentSession.value)
      .maybeSingle()

    if (data && data.draft_data) {
      manualEntries.value = data.draft_data

      if (data.draft_data.__custom_sheets__) {
        sessionCustomSheets.value[currentSession.value] =
          data.draft_data.__custom_sheets__
      }
    }
  } catch (err) {
    console.error('Failed to load manual drafts:', err)
  }
}

const saveCustomSheetsToCloud = async () => {
  manualEntries.value['__custom_sheets__'] =
    sessionCustomSheets.value[currentSession.value]

  try {
    await supabase
      .from('jadual_manual_drafts')
      .upsert(
        {
          target_date: targetDate.value,
          session: currentSession.value,
          draft_data: manualEntries.value
        },
        {
          onConflict: 'target_date,session'
        }
      )
  } catch (err) {
    console.error('Failed to save additional timetables:', err)
  }
}

const saveManualEntry = async (
  slotIndex,
  type,
  period,
  event
) => {
  const text = event.target.innerText
    .trim()
    .replace(/\n+/g, '\n')

  const key = `${slotIndex}-${type}-${period}`

  if (manualEntries.value[key] === text) return

  manualEntries.value[key] = text

  try {
    await supabase
      .from('jadual_manual_drafts')
      .upsert(
        {
          target_date: targetDate.value,
          session: currentSession.value,
          draft_data: manualEntries.value
        },
        {
          onConflict: 'target_date,session'
        }
      )
  } catch (err) {
    console.error('Failed to save temporary draft:', err)
  }
}

const getManualEntry = (slotIndex, type, period) => {
  const key = `${slotIndex}-${type}-${period}`
  return manualEntries.value[key] || ''
}

// =================================================================

const fetchData = async () => {
  await fetchManualDrafts()

  const { data: tData } = await supabase
    .from('teachers')
    .select('*')

  if (tData) {
    tData.forEach(t => {
      teachersMap.value[t.id] = t
    })
  }

  const { data: lData } = await supabase
    .from('leave_requests')
    .select('*')
    .eq('leave_date', targetDate.value)

  if (lData) {
    leaveRequests.value = lData

    const leaveIds = lData.map(l => l.id)

    if (leaveIds.length > 0) {
      const { data: sData } = await supabase
        .from('substitute_assignments')
        .select('*')
        .in('leave_request_id', leaveIds)

      if (sData) {
        const map = {}
        sData.forEach(s => {
          map[s.leave_request_id] = s
        })
        substituteAssignmentsMap.value = map
      } else {
        substituteAssignmentsMap.value = {}
      }
    } else {
      substituteAssignmentsMap.value = {}
    }
  }
}

const hasLeavePeriod = (teacherId, periodNum) => {
  return leaveRequests.value.some(
    r =>
      r.teacher_id === teacherId &&
      Number(r.period) === Number(periodNum) &&
      r.class_name !== 'VIRTUAL_CLASS'
  )
}

const getTeacherPeriodData = (
  teacherId,
  periodNum,
  type
) => {
  const leaveItem = leaveRequests.value.find(
    r =>
      r.teacher_id === teacherId &&
      Number(r.period) === Number(periodNum) &&
      r.class_name !== 'VIRTUAL_CLASS'
  )

  if (!leaveItem) return ''

  if (type === 'class_subject') {
    return `${leaveItem.class_name} ${leaveItem.subject}`
  }

  if (type === 'substitute_name') {
    const subItem =
      substituteAssignmentsMap.value[leaveItem.id]

    if (!subItem || !subItem.sub_teacher_id) {
      return ''
    }

    const subTeacher =
      teachersMap.value[subItem.sub_teacher_id]

    let name = subTeacher ? subTeacher.name : ''

    if (subItem.assignment_type === 'swap') {
      name += ' ✦'
    }

    return subItem.remark
      ? `${name} (${subItem.remark})`
      : name
  }

  return ''
}

const loadSameSessionTeachers = async () => {
  const { data } = await supabase
    .from('teachers')
    .select('*')
    .eq('is_active', true)
    .eq('session', currentSession.value)

  allSameSessionTeachers.value = (data || []).sort((a, b) => 
    a.name.localeCompare(b.name, 'en', { sensitivity: 'base' })
  )
}

// ⭐️ Logic for parsing class schedules and extracting PMPI / subjects
const loadClassSchedulesForTargetDate = async () => {
  try {
    const dateObj = new Date(targetDate.value)
    const dayNum = dateObj.getDay()
    const weekdayCalc = dayNum === 0 ? 7 : dayNum // 1-7 (Monday-Sunday)

    const { data: ttData, error } = await supabase
      .from('timetable')
      .select('class_name, subject, period')
      .eq('weekday', weekdayCalc)

    if (error) throw error

    const map = {}
    const classSet = new Set()

    if (ttData) {
      ttData.forEach(row => {
        if (!row.class_name || !row.subject) return;

        // 1. Get raw class name and remove any subjects tangled in it (e.g. 3A PM, 3A PI)
        const rawClassName = String(row.class_name).toUpperCase();
        let cleanedClassName = rawClassName
          .replace(/\b(PM|PI|MORAL|AGAMA|ISLAM|PENDIDIKAN)\b/g, '')
          .replace(/\s+/g, ' ')
          .trim();
        
        // 2. Remove separators at start/end
        cleanedClassName = cleanedClassName.replace(/^[/-]+|[/-]+$/g, '').trim();

        // 3. Extract combined classes (e.g., "3A/3B" split into "3A" and "3B")
        const classTokens = cleanedClassName.split(/[/,&+,]|\b(DAN|AND)\b/).map(c => c ? c.trim() : '').filter(Boolean);

        classTokens.forEach(cName => {
          if (!map[cName]) {
            map[cName] = {}
          }
          
          const existingSubject = map[cName][row.period]
          const currentSubject = String(row.subject).toUpperCase().trim()

          if (existingSubject) {
            const s1 = existingSubject
            const s2 = currentSubject
            
            const hasPM = s1.includes('PM') || s1.includes('MORAL') || s2.includes('PM') || s2.includes('MORAL')
            const hasPI = s1.includes('PI') || s1.includes('ISLAM') || s1.includes('AGAMA') || s2.includes('PI') || s2.includes('ISLAM') || s2.includes('AGAMA')
            
            if (hasPM && hasPI) {
              map[cName][row.period] = 'PMPI'
            } else {
              // If not PMPI & no duplication, join subjects (e.g., BM/BI)
              if (!s1.includes(s2)) {
                map[cName][row.period] = `${s1}/${s2}`
              }
            }
          } else {
            map[cName][row.period] = currentSubject
          }
          
          classSet.add(cName)
        })
      })
    }

    classSchedulesMap.value = map
    allClassesList.value = Array.from(classSet).sort((a, b) => a.localeCompare(b, 'en', { numeric: true }))
  } catch (err) {
    console.error('Failed to load class timetable data:', err)
  }
}

// =================================================================
// ⭐️ Logic for clearing the entire manual row
const clearManualRow = async (pageIndex, slotIndex, sheetId) => {
  if (!window.confirm('Are you sure you want to clear all contents in this row (including any assigned substitute records)?')) {
    return
  }

  const prefix = sheetId ? `sheet_${sheetId}_${slotIndex}` : `page_${pageIndex}_${slotIndex}`
  const periods = currentPeriodTimes.value.length

  try {
    for (let p = 1; p <= periods; p++) {
      const virtualLeaveKey = `${prefix}_virtual_leave_${p}`
      const existingVirtualLeaveId = manualEntries.value[virtualLeaveKey]

      // Remove assignment record from database
      if (existingVirtualLeaveId) {
        await supabase.from('substitute_assignments').delete().eq('leave_request_id', existingVirtualLeaveId)
        await supabase.from('leave_requests').delete().eq('id', existingVirtualLeaveId)
        delete manualEntries.value[virtualLeaveKey]
      }

      // Clear text content
      manualEntries.value[`${prefix}-kelas-${p}`] = ''
      manualEntries.value[`${prefix}-ganti-${p}`] = ''
      manualEntries.value[`${prefix}-ttangan-${p}`] = ''
    }

    manualEntries.value[`${prefix}-name-0`] = ''

    await saveCustomSheetsToCloud()
    toast.success('Row content has been cleared!')
    
    // Refresh assignment table
    fetchData()
  } catch (err) {
    toast.error('Failed to clear: ' + err.message)
  }
}

// =================================================================
// Open modal to extract class subjects for the entire row
const openClassPicker = (pageIndex, slotIndex, sheetId) => {
  classPickerTarget.value = { pageIndex, slotIndex, sheetId }
  selectedClassToFill.value = ''
  showClassPickerModal.value = true
}

// Confirm class extraction to the CLASS row
const confirmClassPicker = async () => {
  if (!selectedClassToFill.value) {
    toast.error('Please select a class!')
    return
  }

  const { pageIndex, slotIndex, sheetId } = classPickerTarget.value
  const prefix = sheetId ? `sheet_${sheetId}_${slotIndex}` : `page_${pageIndex}_${slotIndex}`
  const className = selectedClassToFill.value

  // 1. Insert class name into the name cell
  manualEntries.value[`${prefix}-name-0`] = className

  // 2. Auto-fill subjects into the CLASS row
  for (let p = 1; p <= currentPeriodTimes.value.length; p++) {
    const subject = classSchedulesMap.value[className]?.[p] || ''
    // Output format: Class name + Subject (e.g., 3A PMPI)
    const text = subject ? `${className} ${subject}` : '' 
    manualEntries.value[`${prefix}-kelas-${p}`] = text
  }

  await saveCustomSheetsToCloud()
  toast.success(`Successfully extracted all subjects for ${className} today!`)
  showClassPickerModal.value = false
}

// =================================================================

const handleCellClick = async (
  teacherId,
  periodNum
) => {
  const leaveItem = leaveRequests.value.find(
    r =>
      r.teacher_id === teacherId &&
      Number(r.period) === Number(periodNum) &&
      r.class_name !== 'VIRTUAL_CLASS'
  )

  if (!leaveItem) {
    toast.error('No leave record found for this period!')
    return
  }

  currentLeaveItem.value = leaveItem
  assignmentRemark.value = ''
  manualSelectedTeacherId.value = ''
  assignmentType.value = 'substitute'

  const existingSub =
    substituteAssignmentsMap.value[leaveItem.id]

  if (existingSub) {
    assignmentRemark.value =
      existingSub.remark || ''

    manualSelectedTeacherId.value =
      existingSub.sub_teacher_id || ''

    assignmentType.value =
      existingSub.assignment_type || 'substitute'
  }

  showModal.value = true
  loadingRecs.value = true
  recCurrentPage.value = 1

  try {
    let results = await recommendSubstitute(leaveItem, 100)
    if (!results) results = []

    await loadSameSessionTeachers()
    
    const absentTeacherId = leaveItem.teacher_id
    const recIds = new Set(results.map(t => t.id))

    if (results.length < allSameSessionTeachers.value.length - 1) {
      const weekday = new Date(targetDate.value).getDay() || 7
      const { data: ttData } = await supabase
        .from('timetable')
        .select('teacher_id, period')
        .eq('weekday', weekday)

      const originalClassMap = {}
      const busyPeriodsMap = {}

      if (ttData) {
        ttData.forEach(row => {
          originalClassMap[row.teacher_id] = (originalClassMap[row.teacher_id] || 0) + 1
          if (!busyPeriodsMap[row.teacher_id]) busyPeriodsMap[row.teacher_id] = new Set()
          busyPeriodsMap[row.teacher_id].add(Number(row.period))
        })
      }

      const todaySubMap = {}
      Object.values(substituteAssignmentsMap.value).forEach(sub => {
        if (sub.sub_teacher_id) {
          todaySubMap[sub.sub_teacher_id] = (todaySubMap[sub.sub_teacher_id] || 0) + 1
        }
      })

      const restTeachers = allSameSessionTeachers.value
        .filter(t => !recIds.has(t.id) && t.id !== absentTeacherId)
        .map(t => {
          const isBusy = busyPeriodsMap[t.id]?.has(Number(periodNum))
          return {
            id: t.id,
            name: t.name,
            originalClasses: originalClassMap[t.id] || 0,
            todaySubCount: todaySubMap[t.id] || 0,
            currentSubCount: '-',
            max_substitute_per_week: t.max_substitute_per_week || 8,
            isBusy: isBusy
          }
        })

      restTeachers.sort((a, b) => {
        if (a.isBusy !== b.isBusy) return a.isBusy ? 1 : -1
        if (a.originalClasses !== b.originalClasses) return a.originalClasses - b.originalClasses
        return a.name.localeCompare(b.name, 'en', { sensitivity: 'base' })
      })

      results = [...results, ...restTeachers]
    }

    recommendations.value = results
  } catch (err) {
    toast.error('Failed to load scheduling data: ' + err.message)
    recommendations.value = []
  } finally {
    loadingRecs.value = false
  }
}

const assignSubstitute = async (teacherId) => {
  if (!teacherId || !currentLeaveItem.value) {
    return
  }

  try {
    const leaveId = currentLeaveItem.value.id
    const existing =
      substituteAssignmentsMap.value[leaveId]

    const payload = {
      sub_teacher_id: teacherId,
      remark: assignmentRemark.value
        ? assignmentRemark.value.trim()
        : null,
      assignment_type: assignmentType.value
    }

    if (existing) {
      const { error } = await supabase
        .from('substitute_assignments')
        .update(payload)
        .eq('id', existing.id)

      if (error) throw error
    } else {
      const { error } = await supabase
        .from('substitute_assignments')
        .insert({
          leave_request_id: leaveId,
          ...payload
        })

      if (error) throw error
    }

    await supabase
      .from('leave_requests')
      .update({
        status: 'assigned'
      })
      .eq('id', leaveId)

    toast.success(
      assignmentType.value === 'swap'
        ? 'Swap assigned successfully!'
        : 'Substitute assigned successfully!'
    )

    showModal.value = false

    fetchData()
  } catch (err) {
    toast.error('Assignment failed: ' + err.message)
  }
}

const removeAssignment = async () => {
  if (!currentLeaveItem.value) return

  try {
    const leaveId = currentLeaveItem.value.id

    const existing =
      substituteAssignmentsMap.value[leaveId]

    if (existing) {
      await supabase
        .from('substitute_assignments')
        .delete()
        .eq('id', existing.id)

      await supabase
        .from('leave_requests')
        .update({
          status: 'pending'
        })
        .eq('id', leaveId)

      toast.success('Assignment cancelled successfully')

      showModal.value = false

      fetchData()
    }
  } catch (err) {
    toast.error('Operation failed: ' + err.message)
  }
}

const handleAutoAssignAll = async () => {
  const pendingRequests =
    leaveRequests.value.filter(req => {
      if (req.class_name === 'VIRTUAL_CLASS') {
        return false
      }

      const teacher =
        teachersMap.value[req.teacher_id]

      const inCurrentSession =
        teacher &&
        (teacher.session || 'morning') ===
          currentSession.value

      const notAssigned =
        !substituteAssignmentsMap.value[req.id] ||
        !substituteAssignmentsMap.value[req.id]
          .sub_teacher_id

      return (
        inCurrentSession &&
        notAssigned
      )
    })

  if (pendingRequests.length === 0) {
    toast.success('No pending assignments for the current session!')
    return
  }

  isAutoAssigning.value = true

  let successCount = 0

  try {
    for (const req of pendingRequests) {
      const recs =
        await recommendSubstitute(req)

      if (recs && recs.length > 0) {
        const bestTeacherId = recs[0].id

        const { error: insertErr } =
          await supabase
            .from('substitute_assignments')
            .insert({
              leave_request_id: req.id,
              sub_teacher_id: bestTeacherId,
              remark: null,
              assignment_type: 'substitute'
            })

        if (!insertErr) {
          await supabase
            .from('leave_requests')
            .update({
              status: 'assigned'
            })
            .eq('id', req.id)

          successCount++
        }
      }
    }

    toast.success(`Success! Auto-assigned ${successCount} classes.`)

    fetchData()
  } catch (err) {
    toast.error('Error during auto-assignment: ' + err.message)
  } finally {
    isAutoAssigning.value = false
  }
}

// ================= Blank row & virtual assignment logic =================

const openBlankModal = async (
  slot,
  period,
  sheetId
) => {
  blankTarget.value = {
    slot,
    period,
    sheetId
  }

  blankForm.value = {
    teacherId: '',
    remark: '',
    kiraBeban: true
  }

  const prefix = sheetId
    ? `sheet_${sheetId}_${slot}`
    : slot

  const virtualLeaveId =
    manualEntries.value[
      `${prefix}_virtual_leave_${period}`
    ]

  hasExistingVirtual.value =
    !!virtualLeaveId

  if (virtualLeaveId) {
    const existingSub =
      substituteAssignmentsMap.value[
        virtualLeaveId
      ]

    if (existingSub) {
      blankForm.value.teacherId =
        existingSub.sub_teacher_id || ''

      blankForm.value.remark =
        existingSub.remark || ''

      blankForm.value.kiraBeban = true
    }
  }

  await loadSameSessionTeachers()

  showBlankModal.value = true
}

const confirmBlankAssignment = async () => {
  const {
    slot,
    period,
    sheetId
  } = blankTarget.value

  const prefix = sheetId
    ? `sheet_${sheetId}_${slot}`
    : slot

  const textKey =
    `${prefix}-ganti-${period}`

  const virtualLeaveKey =
    `${prefix}_virtual_leave_${period}`

  if (
    !blankForm.value.teacherId &&
    blankForm.value.kiraBeban
  ) {
    return toast.error('Please select a teacher if you want to include in workload calculation!')
  }

  let teacherName = ''

  if (blankForm.value.teacherId) {
    const t =
      allSameSessionTeachers.value.find(
        x =>
          x.id ===
          blankForm.value.teacherId
      ) ||
      teachersMap.value[
        blankForm.value.teacherId
      ]

    teacherName = t ? t.name : ''
  }
  
  let displayText = teacherName

  if (blankForm.value.remark) {
    displayText = teacherName
      ? `${teacherName} (${blankForm.value.remark})`
      : blankForm.value.remark
  }
  
  const existingVirtualLeaveId =
    manualEntries.value[
      virtualLeaveKey
    ]

  const dateObj =
    new Date(targetDate.value)

  const dayNum =
    dateObj.getDay()

  const weekdayCalc =
    dayNum === 0 ? 7 : dayNum

  try {
    if (blankForm.value.kiraBeban) {
      const {
        data: existingLeave
      } = await supabase
        .from('leave_requests')
        .select('id, class_name')
        .eq(
          'teacher_id',
          blankForm.value.teacherId
        )
        .eq(
          'leave_date',
          targetDate.value
        )
        .eq('period', period)
        .maybeSingle()

      let targetLeaveId = null

      if (existingLeave) {
        if (
          existingLeave.class_name !==
          'VIRTUAL_CLASS'
        ) {
          return toast.error(
            'This teacher already has an official leave record for this period!'
          )
        }

        targetLeaveId =
          existingLeave.id

        await supabase
          .from('leave_requests')
          .update({
            reason:
              blankForm.value.remark ||
              'SPECIAL TASK'
          })
          .eq(
            'id',
            targetLeaveId
          )
      } else {
        const {
          data: newLeave,
          error: leaveErr
        } = await supabase
          .from('leave_requests')
          .insert({
            teacher_id:
              blankForm.value.teacherId,
            leave_date:
              targetDate.value,
            weekday:
              weekdayCalc,
            period:
              period,
            reason:
              blankForm.value.remark ||
              'SPECIAL TASK',
            class_name:
              'VIRTUAL_CLASS',
            subject:
              'VIRTUAL_SUB',
            status:
              'assigned'
          })
          .select()
          .single()
        
        if (leaveErr) throw leaveErr

        targetLeaveId =
          newLeave.id
      }

      const {
        data: existingSub
      } = await supabase
        .from('substitute_assignments')
        .select('id')
        .eq(
          'leave_request_id',
          targetLeaveId
        )
        .maybeSingle()

      if (existingSub) {
        await supabase
          .from('substitute_assignments')
          .update({
            sub_teacher_id:
              blankForm.value.teacherId,
            remark:
              blankForm.value.remark
          })
          .eq(
            'id',
            existingSub.id
          )
      } else {
        await supabase
          .from('substitute_assignments')
          .insert({
            leave_request_id:
              targetLeaveId,
            sub_teacher_id:
              blankForm.value.teacherId,
            assignment_type:
              'substitute',
            remark:
              blankForm.value.remark
          })
      }

      if (
        existingVirtualLeaveId &&
        existingVirtualLeaveId !==
          targetLeaveId
      ) {
        await supabase
          .from('substitute_assignments')
          .delete()
          .eq(
            'leave_request_id',
            existingVirtualLeaveId
          )

        await supabase
          .from('leave_requests')
          .delete()
          .eq(
            'id',
            existingVirtualLeaveId
          )
      }

      manualEntries.value[
        virtualLeaveKey
      ] = targetLeaveId

    } else {
      if (existingVirtualLeaveId) {
        await supabase
          .from('substitute_assignments')
          .delete()
          .eq(
            'leave_request_id',
            existingVirtualLeaveId
          )

        await supabase
          .from('leave_requests')
          .delete()
          .eq(
            'id',
            existingVirtualLeaveId
          )

        delete manualEntries.value[
          virtualLeaveKey
        ]
      }
    }

    manualEntries.value[textKey] =
      displayText

    await saveCustomSheetsToCloud()
    
    toast.success(
      blankForm.value.kiraBeban
        ? 'Assigned successfully! Recorded in workload stats.'
        : 'Text saved successfully! Excluded from workload.'
    )

    showBlankModal.value = false

    fetchData()

  } catch (err) {
    toast.error('Save failed: ' + err.message)
  }
}

const removeBlankAssignment = async () => {
  const {
    slot,
    period,
    sheetId
  } = blankTarget.value

  const prefix = sheetId
    ? `sheet_${sheetId}_${slot}`
    : slot

  const virtualLeaveKey =
    `${prefix}_virtual_leave_${period}`

  const textKey =
    `${prefix}-ganti-${period}`

  const existingVirtualLeaveId =
    manualEntries.value[
      virtualLeaveKey
    ]

  try {
    if (existingVirtualLeaveId) {
      await supabase
        .from('substitute_assignments')
        .delete()
        .eq(
          'leave_request_id',
          existingVirtualLeaveId
        )

      await supabase
        .from('leave_requests')
        .delete()
        .eq(
          'id',
          existingVirtualLeaveId
        )

      delete manualEntries.value[
        virtualLeaveKey
      ]
    }

    manualEntries.value[textKey] = ''

    await saveCustomSheetsToCloud()
    
    toast.success('Cell cleared and workload assignment cancelled!')

    showBlankModal.value = false

    fetchData()

  } catch (err) {
    toast.error('Clear failed: ' + err.message)
  }
}

// =================================================================

watch(
  [targetDate, currentSession],
  () => {
    fetchData()
    loadClassSchedulesForTargetDate() // Reload schedule when date/session changes
  }
)

onMounted(async () => {
  await fetchSchoolIdentity()
  await fetchData()
  await loadClassSchedulesForTargetDate() // Load current date schedule on mount
})

// ========================= Direct PDF Export =========================

const handleExportPdf = async () => {
  if (isExportingPdf.value) return

  isExportingPdf.value = true

  try {
    const { jsPDF } = await import('jspdf')

    const doc = new jsPDF({
      orientation: 'landscape',
      unit: 'mm',
      format: 'a4',
      compress: true
    })

    const PAGE_W = 297
    const PAGE_H = 210
    const M = 5
    const CONTENT_W =
      PAGE_W - (M * 2)

    const BLACK = [0, 0, 0]
    const BLUE = [20, 28, 115]

    const arrayBufferToBase64 = (
      buffer
    ) => {
      const bytes =
        new Uint8Array(buffer)

      const chunkSize = 0x8000
      let binary = ''

      for (
        let i = 0;
        i < bytes.length;
        i += chunkSize
      ) {
        binary += String.fromCharCode(
          ...bytes.subarray(
            i,
            i + chunkSize
          )
        )
      }

      return btoa(binary)
    }

    const loadPdfFont = async (
      url,
      fileName,
      fontStyle
    ) => {
      const response =
        await fetch(url)

      if (!response.ok) {
        throw new Error(
          `Failed to download PDF font file: ${url} (HTTP ${response.status})`
        )
      }

      const fontBuffer =
        await response.arrayBuffer()

      const fontBase64 =
        arrayBufferToBase64(
          fontBuffer
        )

      doc.addFileToVFS(
        fileName,
        fontBase64
      )

      doc.addFont(
        fileName,
        'Georgia',
        fontStyle
      )
    }

    await loadPdfFont(
      '/fonts/Georgia.ttf',
      'Georgia.ttf',
      'normal'
    )

    await loadPdfFont(
      '/fonts/Georgia%20Bold.ttf',
      'Georgia Bold.ttf',
      'bold'
    )

    doc.setFont(
      'Georgia',
      'normal'
    )

    const safeText = (
      value
    ) => {
      if (
        value === null ||
        value === undefined
      ) {
        return ''
      }

      return String(value).trim()
    }

    const drawCenteredText = (
      value,
      x,
      y,
      width,
      height,
      fontSize = 8,
      bold = false,
      color = BLACK
    ) => {
      let text = safeText(value)

      if (!text) return

      text = text.replace(/•/g, '-')
      text = text.replace(/ \(/g, '\n(')

      const innerWidth = Math.max(width - 1.6, 1)
      const innerHeight = Math.max(height - 0.8, 1)

      doc.setFont('Georgia', bold ? 'bold' : 'normal')
      doc.setTextColor(...color)

      let size = fontSize
      let lines = []

      const words = text.split(/[\s\n]+/)

      for (let attempt = 0; attempt < 20; attempt++) {
        doc.setFontSize(size)
        
        let isWordTooWide = false
        for (const word of words) {
          if (doc.getTextWidth(word) > innerWidth) {
            isWordTooWide = true
            break
          }
        }

        lines = doc.splitTextToSize(text, innerWidth)
        const lineHeight = Math.max(size * 0.38, 1.8)
        const totalHeight = lines.length * lineHeight

        if (totalHeight <= innerHeight && !isWordTooWide && size <= fontSize) {
          break
        }

        if (totalHeight > innerHeight || isWordTooWide) {
          size -= 0.25

          if (size <= 3.2) {
            size = 3.2
            break
          }
        } else {
          break
        }
      }

      doc.setFontSize(size)
      lines = doc.splitTextToSize(text, innerWidth)

      let lineHeight = Math.max(size * 0.40, 2.0)

      if (lines.length > 1 && lines.length * lineHeight > innerHeight) {
        lineHeight = innerHeight / lines.length
      }

      const totalHeight = lines.length * lineHeight

      let startY = y + Math.max((height - totalHeight) / 2 + lineHeight * 0.55, lineHeight)

      lines.forEach(line => {
        doc.text(line, x + width / 2, startY, {
          align: 'center',
          baseline: 'middle'
        })
        startY += lineHeight
      })
    }

    const drawCell = (
      x,
      y,
      w,
      h,
      value,
      options = {}
    ) => {
      const {
        fontSize = 7,
        bold = false,
        color = BLACK,
        fill = null,
        padding = 1
      } = options

      if (fill) {
        doc.setFillColor(
          ...fill
        )

        doc.rect(
          x,
          y,
          w,
          h,
          'F'
        )
      }

      doc.setDrawColor(
        ...BLACK
      )

      doc.setLineWidth(
        0.35
      )

      doc.rect(
        x,
        y,
        w,
        h,
        'S'
      )

      drawCenteredText(
        value,
        x + padding,
        y + padding,
        w - padding * 2,
        h - padding * 2,
        fontSize,
        bold,
        color
      )
    }

    const drawPageHeader = (
      dateText,
      dayText,
      sessionText
    ) => {
      let y = M + 1

      doc.setTextColor(
        ...BLACK
      )

      doc.setFont(
        'Georgia',
        'bold'
      )

      doc.setFontSize(18)

      doc.text(
        schoolName.value ||
          'SJK (C) LADANG GRISEK',
        PAGE_W / 2,
        y + 5,
        {
          align: 'center'
        }
      )

      doc.setFontSize(14)

      doc.text(
        `SUBSTITUTE TEACHER TIMETABLE (${sessionText})`,
        PAGE_W / 2,
        y + 14,
        {
          align: 'center'
        }
      )

      const infoY =
        y + 27

      doc.setFontSize(
        8.5
      )

      doc.text(
        'DATE :',
        M,
        infoY,
        {
          align: 'left'
        }
      )

      doc.setFont(
        'Georgia',
        'normal'
      )

      doc.text(
        safeText(dateText),
        M + 22,
        infoY,
        {
          align: 'left'
        }
      )

      doc.setFont(
        'Georgia',
        'bold'
      )

      doc.text(
        'DAY :',
        PAGE_W - M - 42,
        infoY,
        {
          align: 'left'
        }
      )

      doc.setFont(
        'Georgia',
        'normal'
      )

      doc.text(
        safeText(
          dayText
        ).toUpperCase(),
        PAGE_W - M - 28,
        infoY,
        {
          align: 'left'
        }
      )

      doc.setDrawColor(
        ...BLACK
      )

      doc.setLineWidth(
        0.55
      )

      doc.line(
        M,
        infoY + 3,
        PAGE_W - M,
        infoY + 3
      )

      return infoY + 7
    }

    const getDisplayValue = (
      teacher,
      period,
      type
    ) => {
      return getTeacherPeriodData(
        teacher.id,
        period,
        type
      )
    }

    const getManualValue = (
      slotIndex,
      type,
      period,
      sheetId = null,
      pageIndex = null
    ) => {
      let prefix = sheetId
        ? `sheet_${sheetId}_${slotIndex}`
        : (pageIndex !== null ? `page_${pageIndex}_${slotIndex}` : slotIndex)

      return getManualEntry(
        prefix,
        type,
        period
      )
    }

    const drawTimetable = ({
      dateText,
      dayText,
      sessionText,
      sheetId = null,
      pageIndex = null,
      pageTeachers = []
    }) => {
      const tableTop =
        drawPageHeader(
          dateText,
          dayText,
          sessionText
        )

      const teacherW = 31
      const labelW = 20

      const periodW =
        (
          CONTENT_W -
          teacherW -
          labelW
        ) /
        currentPeriodTimes
          .value.length

      const headerH = 11
      const rowH = 8.3
      const tableX = M

      let x = tableX

      drawCell(
        x,
        tableTop,
        teacherW + labelW,
        headerH,
        'TIME',
        {
          fontSize: 8,
          bold: true
        }
      )

      x +=
        teacherW +
        labelW

      currentPeriodTimes.value.forEach(
        (time, index) => {
          doc.setDrawColor(
            ...BLACK
          )

          doc.setLineWidth(
            0.35
          )

          doc.rect(
            x,
            tableTop,
            periodW,
            headerH,
            'S'
          )

          doc.setFont(
            'Georgia',
            'bold'
          )

          doc.setFontSize(
            7.5
          )

          doc.setTextColor(
            ...BLACK
          )

          doc.text(
            String(index + 1),
            x +
              periodW / 2,
            tableTop + 4.5,
            {
              align: 'center'
            }
          )

          doc.setFont(
            'Georgia',
            'normal'
          )

          doc.setFontSize(
            4.4
          )

          doc.text(
            safeText(time),
            x +
              periodW / 2,
            tableTop + 8.5,
            {
              align: 'center'
            }
          )

          x += periodW
        }
      )

      let y =
        tableTop +
        headerH

      for (
        let slot = 1;
        slot <= 5;
        slot++
      ) {
        const teacher = pageTeachers[slot - 1] || null

        const teacherName =
          teacher
            ? teacher.name
            : getManualValue(
                slot,
                'name',
                0,
                sheetId,
                pageIndex
              )

        const teacherReason =
          teacher
            ? (
                teacher.reason
                  ? `(${teacher.reason})`
                  : ''
              )
            : ''

        drawCell(
          tableX,
          y,
          teacherW,
          rowH * 3,
          `${teacherName}${
            teacherReason
              ? `\n${teacherReason}`
              : ''
          }`,
          {
            fontSize:
              teacherReason
                ? 6.2
                : 7,
            bold: true
          }
        )

        drawCell(
          tableX + teacherW,
          y,
          labelW,
          rowH,
          'CLASS',
          {
            fontSize: 6.7,
            bold: true
          }
        )

        currentPeriodTimes.value.forEach(
          (_, pIndex) => {
            const period =
              pIndex + 1

            const value =
              teacher
                ? getDisplayValue(
                    teacher,
                    period,
                    'class_subject'
                  )
                : getManualValue(
                    slot,
                    'kelas',
                    period,
                    sheetId,
                    pageIndex
                  )

            drawCell(
              tableX +
                teacherW +
                labelW +
                pIndex *
                  periodW,
              y,
              periodW,
              rowH,
              value,
              {
                fontSize: 6.0,
                bold: true
              }
            )
          }
        )

        drawCell(
          tableX + teacherW,
          y + rowH,
          labelW,
          rowH,
          'SUBSTITUTE',
          {
            fontSize: 5.2, // ⭐️ 完美适配 PDF 导出时的 SUBSTITUTE 字号
            bold: true
          }
        )

        currentPeriodTimes.value.forEach(
          (_, pIndex) => {
            const period =
              pIndex + 1

            const value =
              teacher
                ? getDisplayValue(
                    teacher,
                    period,
                    'substitute_name'
                  )
                : getManualValue(
                    slot,
                    'ganti',
                    period,
                    sheetId,
                    pageIndex
                  )

            drawCell(
              tableX +
                teacherW +
                labelW +
                pIndex *
                  periodW,
              y + rowH,
              periodW,
              rowH,
              value,
              {
                fontSize: 5.5,
                bold: true,
                color: BLUE
              }
            )
          }
        )

        drawCell(
          tableX + teacherW,
          y + rowH * 2,
          labelW,
          rowH,
          'SIGNATURE',
          {
            fontSize: 5.2, // ⭐️ 完美适配 PDF 导出时的 SIGNATURE 字号
            bold: true
          }
        )

        currentPeriodTimes.value.forEach(
          (_, pIndex) => {
            const period =
              pIndex + 1

            const value =
              teacher
                ? ''
                : getManualValue(
                    slot,
                    'ttangan',
                    period,
                    sheetId,
                    pageIndex
                  )

            drawCell(
              tableX +
                teacherW +
                labelW +
                pIndex *
                  periodW,
              y + rowH * 2,
              periodW,
              rowH,
              value,
              {
                fontSize: 5.8
              }
            )
          }
        )

        y += rowH * 3
      }
    }

    const dateText =
      formattedDate.value

    const dayText =
      formattedDayName.value

    const sessionText =
      currentSession.value ===
      'morning'
        ? 'MORNING SESSION'
        : 'AFTERNOON SESSION'

    const pages = paginatedTeacherPages.value
    pages.forEach((pageTeachers, pIndex) => {
      if (pIndex > 0) {
        doc.addPage()
      }
      drawTimetable({
        dateText,
        dayText,
        sessionText,
        pageIndex: pIndex,
        pageTeachers
      })
    })

    for (
      const sheet of
        extraCustomSheets.value
    ) {
      doc.addPage()

      drawTimetable({
        dateText:
          sheet.date ||
          dateText,
        dayText:
          sheet.day ||
          dayText,
        sessionText,
        sheetId:
          sheet.id,
        pageIndex: null,
        pageTeachers: []
      })
    }

    const safeDate =
      targetDate.value.replace(
        /[^0-9-]/g,
        ''
      )

    const sessionName =
      currentSession.value ===
      'morning'
        ? 'MORNING_SESSION'
        : 'AFTERNOON_SESSION'

    doc.save(
      `SUBSTITUTE_TIMETABLE_${safeDate}_${sessionName}.pdf`
    )

    toast.success(
      `PDF generated successfully, total of ${
        pages.length +
        extraCustomSheets.value.length
      } pages.`
    )

  } catch (err) {
    console.error('PDF export failed:', err)
    toast.error(`PDF generation failed: ${err?.message || err}`)
  } finally {
    isExportingPdf.value = false
  }
}

const extraCustomSheets = computed(() => {
  return (
    sessionCustomSheets.value[
      currentSession.value
    ] || []
  )
})

const addBlankSheet = async () => {
  sessionCustomSheets.value[
    currentSession.value
  ].push({
    id: Date.now(),
    date: '',
    day: ''
  })

  await saveCustomSheetsToCloud()
}

const removeCustomSheet = async (
  id
) => {
  const list =
    sessionCustomSheets.value[
      currentSession.value
    ]

  const index =
    list.findIndex(
      sheet =>
        sheet.id === id
    )

  if (index !== -1) {
    list.splice(index, 1)

    await saveCustomSheetsToCloud()
  }
}
</script>

<style scoped>
</style>

<style>
@media print {
  @page {
    size: A4 landscape !important;
    margin: 5mm !important;
  }

  html,
  body {
    background: #fff !important;
    margin: 0 !important;
    padding: 0 !important;
  }

  body {
    -webkit-print-color-adjust: exact !important;
    print-color-adjust: exact !important;
  }

  body > * {
    margin-top: 0 !important;
  }

  .print-main-sheet {
    break-inside: auto !important;
    page-break-inside: auto !important;
  }

  .print-main-sheet tbody,
  .print-main-sheet tbody tr,
  .print-main-sheet tbody td {
    break-inside: avoid !important;
    page-break-inside: avoid !important;
  }

  .print-main-sheet thead {
    display: table-header-group !important;
  }

  .print-page-break {
    display: block !important;
    height: 0 !important;
    margin: 0 !important;
    padding: 0 !important;
    border: 0 !important;

    break-before: page !important;
    page-break-before: always !important;
  }

  .print-custom-sheet {
    display: block !important;
    break-inside: avoid !important;
    page-break-inside: avoid !important;

    margin-top: 0 !important;
    padding-top: 0 !important;
  }

  .print-custom-sheet > div {
    break-inside: avoid !important;
    page-break-inside: avoid !important;
  }

  .print-custom-sheet table,
  .print-custom-sheet tbody,
  .print-custom-sheet tr,
  .print-custom-sheet td,
  .print-custom-sheet th {
    break-inside: avoid !important;
    page-break-inside: avoid !important;
  }

  .print-custom-sheet thead {
    display: table-header-group !important;
  }

  .force-page-break {
    break-after: auto !important;
    page-break-after: auto !important;
  }

  .print\:hidden {
    display: none !important;
  }

  .print-main-sheet .overflow-x-auto,
  .print-custom-sheet .overflow-x-auto {
    overflow: visible !important;
  }
}
</style>