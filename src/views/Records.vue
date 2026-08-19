<template>
  <!-- 保持 min-w-[1024px] 确保表格不会无限变宽 -->
  <div class="p-4 sm:p-8 mx-auto min-h-screen space-y-8 min-w-[1024px] print:p-0 print:min-w-0 print:w-auto print:m-0">
    
    <!-- Screen Action Bar (Automatically hidden during printing) -->
    <div class="print:hidden bg-white rounded-3xl p-6 sm:p-8 shadow-sm ring-1 ring-slate-900/5 flex flex-col gap-6">
      
      <!-- Title & Subtitle -->
      <div class="space-y-2">
        <h1 class="text-2xl sm:text-3xl font-extrabold tracking-tight text-transparent bg-clip-text bg-gradient-to-r from-slate-900 via-indigo-800 to-violet-800 whitespace-nowrap flex items-center gap-3">
          <UsersRound class="w-8 h-8 text-indigo-700 shrink-0" />
          DAILY SUBSTITUTE TEACHER MANAGEMENT
        </h1>
        <p class="text-slate-500 text-xs sm:text-sm font-medium leading-relaxed whitespace-nowrap">
          CLICK TIMETABLE CELLS TO ASSIGN SUBSTITUTE TEACHERS, SUPPORTS ONE-CLICK SMART AUTOMATIC SCHEDULING
        </p>
      </div>

      <!-- Action Buttons Bar (🌟 修复：加回 flex-wrap，让按钮在空间不足时优雅换行，绝不刺破卡片) -->
      <div class="flex flex-wrap items-center gap-4 pt-4 border-t border-slate-100">
        
        <!-- 1. Session Switcher Tabs (MORNING / AFTERNOON) -->
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

        <!-- 2. Date Picker -->
        <div class="flex items-center gap-2 bg-slate-50 px-4 h-11 rounded-2xl border border-slate-200/80 shadow-2xs shrink-0">
          <span class="text-xs font-bold text-slate-500 whitespace-nowrap">SELECT DATE:</span>
          <input 
            type="date" 
            v-model="targetDate" 
            class="bg-transparent text-xs font-bold text-slate-800 focus:outline-none cursor-pointer"
          />
        </div>

        <!-- 3. Smart Auto-Scheduling -->
        <button 
          @click="handleAutoAssignAll"
          :disabled="isAutoAssigning"
          class="bg-indigo-600 hover:bg-indigo-700 disabled:opacity-50 text-white px-5 h-11 rounded-2xl text-xs font-bold shadow-sm transition-all flex items-center justify-center gap-2 cursor-pointer shrink-0 whitespace-nowrap"
        >
          <span v-if="isAutoAssigning" class="w-3.5 h-3.5 border-2 border-white/30 border-t-white rounded-full animate-spin"></span>
          <Zap v-else class="w-4 h-4" />
          <span>SMART SUBSTITUTE SCHEDULING</span>
        </button>

        <!-- 4. Print Button -->
        <button 
          @click="handlePrint"
          class="bg-slate-900 hover:bg-slate-800 text-white px-6 h-11 rounded-2xl text-xs font-bold shadow-sm transition-all flex items-center justify-center gap-2 cursor-pointer shrink-0 whitespace-nowrap"
        >
          <Printer class="w-4 h-4" />
          <span>PRINT TIMETABLE</span>
        </button>

      </div>
    </div>

    <!-- Main Table: Preview / Print Dedicated Area -->
    <div class="bg-white rounded-3xl shadow-sm ring-1 ring-slate-900/5 p-8 print:shadow-none print:ring-0 print:p-0 print:rounded-none print:break-inside-avoid">
      
      <div class="text-center mb-6 print:mb-2">
        <h2 class="text-xl font-black tracking-wider text-black font-serif">SJK (C) LADANG GRISEK</h2>
        <h3 class="text-lg font-bold tracking-widest text-black mt-1 font-serif underline">
          JADUAL GURU GANTI ({{ currentSession === 'morning' ? 'SESI PAGI' : 'SESI PETANG' }})
        </h3>
      </div>

      <div class="flex justify-between items-center mb-4 print:mb-2 font-bold text-sm font-serif border-b-2 border-black pb-2 print:pb-1">
        <div>
          <span class="underline underline-offset-4">TARIKH :</span> <span class="ml-2 border-b border-black px-4">{{ formattedDate }}</span>
        </div>
        <div>
          <span class="underline underline-offset-4">HARI :</span> <span class="ml-2 border-b border-black px-4 uppercase">{{ formattedDayName }}</span>
        </div>
      </div>

      <div class="w-full">
        <table class="w-full border-collapse border-2 border-black text-center text-xs font-serif table-fixed">
          <thead>
            <tr class="bg-slate-100 print:bg-white">
              <th class="border border-black p-1 w-28 font-bold" colspan="2">MASA</th>
              <th v-for="(time, index) in currentPeriodTimes" :key="index" class="border border-black p-1">
                <div class="font-bold">{{ index + 1 }}</div>
                <div class="text-[7px] font-normal mt-0.5 truncate">{{ time }}</div>
              </th>
            </tr>
          </thead>
          <tbody v-for="slotIndex in 5" :key="slotIndex" style="page-break-inside: avoid; break-inside: avoid;" class="print:break-inside-avoid">
              
              <template v-if="displayTeachersList[slotIndex - 1]">
                <tr>
                  <td class="border border-black p-1 bg-slate-50 print:bg-white align-middle text-center" rowspan="3" style="width: 120px; max-width: 120px;">
                    <div class="flex flex-col items-center justify-center w-full px-0.5 overflow-hidden">
                      <span class="text-[10px] uppercase leading-tight font-bold w-full text-center whitespace-normal">
                        {{ displayTeachersList[slotIndex - 1].name }}
                      </span>
                      <span v-if="displayTeachersList[slotIndex - 1].reason" class="text-[8.5px] font-normal text-slate-500 w-full text-center tracking-tighter mt-1 uppercase whitespace-normal leading-tight">
                        ({{ displayTeachersList[slotIndex - 1].reason }})
                      </span>
                    </div>
                  </td>
                  <td class="border border-black p-1 font-bold bg-slate-50 print:bg-white text-[10px]" style="width: 80px;">KELAS</td>
                  <td v-for="p in currentPeriodTimes.length" :key="p" class="border border-black p-0.5 font-semibold align-middle h-8" style="max-width: 0;">
                    <div class="w-full h-full flex items-center justify-center overflow-hidden px-0.5">
                      <span class="block w-full text-center text-[10px] tracking-tighter leading-tight text-slate-800 whitespace-normal">
                        {{ getTeacherPeriodData(displayTeachersList[slotIndex - 1].id, p, 'class_subject') }}
                      </span>
                    </div>
                  </td>
                </tr>
                <tr>
                  <td class="border border-black p-1 font-bold bg-slate-50 print:bg-white text-[10px]">GURU GANTI</td>
                  <td v-for="p in currentPeriodTimes.length" :key="p" 
                      @click="hasLeavePeriod(displayTeachersList[slotIndex - 1].id, p) ? handleCellClick(displayTeachersList[slotIndex - 1].id, p) : null"
                      :class="hasLeavePeriod(displayTeachersList[slotIndex - 1].id, p) ? 'cursor-pointer hover:bg-indigo-50 group' : ''"
                      class="print:hover:bg-transparent border border-black p-0.5 font-bold text-indigo-900 align-middle h-8 transition relative" 
                      style="max-width: 0;">
                    <div class="w-full h-full flex items-center justify-center overflow-hidden px-0.5">
                      <span class="block w-full text-center text-[9px] tracking-tighter leading-tight whitespace-normal">
                        {{ getTeacherPeriodData(displayTeachersList[slotIndex - 1].id, p, 'substitute_name') }}
                      </span>
                      <span v-if="hasLeavePeriod(displayTeachersList[slotIndex - 1].id, p)" class="print:hidden hidden group-hover:inline-block text-[9px] text-indigo-500 absolute right-1">✏️</span>
                    </div>
                  </td>
                </tr>
                <tr>
                  <td class="border border-black p-1 font-bold bg-slate-50 print:bg-white text-[8px] whitespace-nowrap">T/TANGAN</td>
                  <td v-for="p in currentPeriodTimes.length" :key="p" class="border border-black p-1 align-middle h-8"></td>
                </tr>
              </template>

              <template v-else>
                <tr>
                  <td contenteditable="true" 
                      @blur="saveManualEntry(slotIndex, 'name', 0, $event)" 
                      v-text="getManualEntry(slotIndex, 'name', 0)" 
                      class="border border-black p-1 font-bold bg-slate-50 print:bg-white align-middle text-center h-8 outline-none focus:bg-indigo-50/50 hover:bg-slate-100 cursor-text transition-colors overflow-hidden whitespace-pre-wrap break-words leading-tight uppercase text-[10px]" 
                      rowspan="3" style="width: 120px; max-width: 120px;"></td>
                  <td class="border border-black p-1 font-bold bg-slate-50 print:bg-white text-[10px]" style="width: 80px;">KELAS</td>
                  <td v-for="p in currentPeriodTimes.length" :key="'kelas-'+p" 
                      contenteditable="true" 
                      @blur="saveManualEntry(slotIndex, 'kelas', p, $event)" 
                      v-text="getManualEntry(slotIndex, 'kelas', p)" 
                      class="border border-black p-0.5 align-middle h-8 outline-none focus:bg-indigo-50/50 hover:bg-slate-100 cursor-text transition-colors text-[11px] font-semibold overflow-hidden whitespace-pre-wrap break-words leading-tight"
                      style="max-width: 0;"></td>
                </tr>
                <tr>
                  <td class="border border-black p-1 font-bold bg-slate-50 print:bg-white text-[10px]">GURU GANTI</td>
                  <td v-for="p in currentPeriodTimes.length" :key="'ganti-'+p" 
                      class="border border-black p-0.5 align-middle h-8 relative group" style="max-width: 0;">
                    <div class="w-full h-full relative flex items-center justify-center">
                      <div contenteditable="true" 
                          @blur="saveManualEntry(slotIndex, 'ganti', p, $event)" 
                          v-text="getManualEntry(slotIndex, 'ganti', p)" 
                          class="w-full h-full outline-none focus:bg-indigo-50/50 hover:bg-slate-100 cursor-text transition-colors text-[10px] font-bold text-indigo-900 overflow-hidden whitespace-pre-wrap break-words leading-tight flex items-center justify-center"></div>
                      <button contenteditable="false" @click.stop="openBlankModal(slotIndex, p, null)" class="print:hidden absolute right-0 top-0 hidden group-hover:flex bg-indigo-500 text-white rounded-bl px-1.5 py-0.5 text-[9px] cursor-pointer shadow-sm hover:bg-indigo-600 z-10 font-sans tracking-widest font-bold">ASSIGN</button>
                    </div>
                  </td>
                </tr>
                <tr>
                  <td class="border border-black p-1 font-bold bg-slate-50 print:bg-white text-[8px] whitespace-nowrap">T/TANGAN</td>
                  <td v-for="p in currentPeriodTimes.length" :key="'ttangan-'+p" 
                      contenteditable="true" 
                      @blur="saveManualEntry(slotIndex, 'ttangan', p, $event)" 
                      v-text="getManualEntry(slotIndex, 'ttangan', p)" 
                      class="border border-black p-1 align-middle h-8 outline-none focus:bg-indigo-50/50 hover:bg-slate-100 cursor-text transition-colors"></td>
                </tr>
              </template>
          </tbody>
        </table>
      </div>
    </div>

    <!-- Modal: Substitute Assignment Center -->
    <transition enter-active-class="transition duration-300 ease-out" enter-from-class="opacity-0 scale-95" enter-to-class="opacity-100 scale-100" leave-active-class="transition duration-200 ease-in" leave-from-class="opacity-100 scale-100" leave-to-class="opacity-0 scale-95">
      <div v-if="showModal" class="print:hidden fixed inset-0 z-50 flex items-center justify-center p-4 sm:p-6">
        <div class="absolute inset-0 bg-slate-900/30 backdrop-blur-sm" @click="showModal = false"></div>
        <div class="relative bg-white rounded-3xl shadow-2xl w-full max-w-2xl overflow-hidden ring-1 ring-slate-900/10 max-h-[90vh] flex flex-col">
          
          <div class="px-8 py-6 border-b border-slate-100 flex justify-between items-center bg-white/50 backdrop-blur-md shrink-0">
            <div>
              <h2 class="text-xl font-bold text-slate-900">SUBSTITUTE ASSIGNMENT CENTER</h2>
              <p class="text-sm text-slate-500 mt-1">SUPPORTED BY SMART RECOMMENDATIONS, OR SELECT ANY SAME-SESSION TEACHER MANUALLY BELOW</p>
            </div>
            <button @click="showModal = false" class="text-slate-400 hover:text-slate-600 bg-slate-100 hover:bg-slate-200 rounded-full p-2 transition">×</button>
          </div>
          
          <div class="p-8 bg-slate-50/50 space-y-6 overflow-y-auto">
            <div class="bg-white p-4 rounded-2xl border border-slate-200 shadow-sm">
              <h3 class="text-xs font-bold text-slate-700 mb-3 uppercase tracking-wider">🏷️ ASSIGNMENT TYPE:</h3>
              <div class="flex flex-col sm:flex-row gap-4">
                <label class="flex items-center gap-2 cursor-pointer bg-slate-50 px-4 py-2 rounded-xl border border-slate-100 hover:bg-indigo-50 transition">
                  <input type="radio" v-model="assignmentType" value="substitute" class="text-indigo-600 focus:ring-indigo-500 w-4 h-4" />
                  <span class="text-sm font-semibold text-slate-800">OFFICIAL SUBSTITUTE <span class="text-xs text-slate-400 font-normal ml-1">(INCLUDED IN WORKLOAD STATS)</span></span>
                </label>
                <label class="flex items-center gap-2 cursor-pointer bg-slate-50 px-4 py-2 rounded-xl border border-slate-100 hover:bg-indigo-50 transition">
                  <input type="radio" v-model="assignmentType" value="swap" class="text-indigo-600 focus:ring-indigo-500 w-4 h-4" />
                  <span class="text-sm font-semibold text-slate-800">TIMETABLE SWAP <span class="text-xs text-slate-400 font-normal ml-1">(EXCLUDED FROM STATS)</span></span>
                </label>
              </div>
            </div>
            
            <div class="bg-white p-4 rounded-2xl border border-slate-200 shadow-sm flex items-center gap-3">
              <span class="text-xs font-bold text-slate-700 whitespace-nowrap">📍 LOCATION / REMARK:</span>
              <input 
                v-model="assignmentRemark" 
                type="text" 
                placeholder="E.G. LIBRARY (IF STUDENTS NEED TO BE BROUGHT TO THE LIBRARY OR CLASSES COMBINED)" 
                class="w-full px-3 py-1.5 bg-slate-50 border border-slate-200 rounded-xl text-xs font-medium text-slate-800 focus:outline-none focus:ring-2 focus:ring-indigo-500/50"
              />
            </div>

            <div class="bg-indigo-50/60 p-5 rounded-2xl border border-indigo-100 shadow-sm">
              <h3 class="text-xs font-bold uppercase tracking-wider text-indigo-900 mb-3 flex items-center gap-2">
                <span>🛠️ MANUAL ASSIGNMENT (WITHOUT SMART RECOMMENDATION)</span>
              </h3>
              <div class="flex flex-col sm:flex-row items-center gap-3">
                <div class="relative w-full">
                  <select 
                    v-model="manualSelectedTeacherId" 
                    class="w-full px-3 py-2 bg-white border border-indigo-200 rounded-xl text-xs font-semibold text-slate-800 focus:outline-none focus:ring-2 focus:ring-indigo-500 appearance-none pr-8 truncate"
                  >
                    <option value="" disabled>-- PLEASE SELECT A SAME-SESSION TEACHER MANUALLY --</option>
                    <option v-for="t in allSameSessionTeachers" :key="t.id" :value="t.id">
                      {{ t.name }}{{ t.subject ? ` (SUBJECT: ${t.subject})` : '' }}
                    </option>
                  </select>
                  <div class="absolute right-3 top-1/2 -translate-y-1/2 pointer-events-none text-slate-500">
                    <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 9l-7 7-7-7"></path></svg>
                  </div>
                </div>
                <button 
                  @click="assignSubstitute(manualSelectedTeacherId)" 
                  :disabled="!manualSelectedTeacherId"
                  class="w-full sm:w-auto bg-indigo-600 hover:bg-indigo-700 disabled:opacity-50 text-white px-5 py-2 rounded-xl text-xs font-semibold shadow-sm transition-all shrink-0 whitespace-nowrap"
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
              
              <div v-if="loadingRecs" class="flex flex-col items-center justify-center py-6 space-y-3">
                <div class="w-6 h-6 border-4 border-indigo-500/30 border-t-indigo-600 rounded-full animate-spin"></div>
                <p class="text-xs text-slate-500 font-medium">SMART ALGORITHM IS CALCULATING...</p>
              </div>
              
              <div v-else-if="recommendations.length === 0" class="bg-white p-4 rounded-2xl border border-slate-200 text-xs text-slate-500 text-center">
                NO AUTOMATIC RECOMMENDATIONS AVAILABLE, PLEASE USE MANUAL ASSIGNMENT ABOVE.
              </div>

              <div v-else class="space-y-3">
                <div v-for="(teacher, index) in recommendations" 
                    :key="teacher.id" 
                    class="group flex flex-col sm:flex-row sm:justify-between sm:items-center p-4 bg-white border border-slate-200 rounded-2xl hover:border-indigo-300 hover:shadow-sm transition-all">
                  <div class="flex items-center gap-3 mb-3 sm:mb-0">
                    <div class="w-8 h-8 rounded-full bg-gradient-to-br from-indigo-100 to-violet-100 text-indigo-700 font-extrabold flex items-center justify-center text-xs">
                      #{{ index + 1 }}
                    </div>
                    <div>
                      <div class="font-bold text-slate-900 text-sm flex items-center gap-2">
                        {{ teacher.name }} 
                      </div>
                      <div class="text-[11px] text-slate-500 mt-1 flex items-center gap-2 flex-wrap">
                        <span>ORIGINAL DAILY CLASSES: <span class="font-bold text-slate-700">{{ teacher.originalClasses }} CLASSES</span></span>
                        <span>·</span>
                        <span>TODAY'S SUBSTITUTIONS: <span class="font-bold text-orange-600">{{ teacher.todaySubCount }} CLASSES</span></span>
                        <span>·</span>
                        <span>THIS WEEK'S SUBSTITUTIONS: <span class="font-bold text-slate-700">{{ teacher.currentSubCount }}/{{ teacher.max_substitute_per_week }}</span></span>
                      </div>
                    </div>
                  </div>
                  <button @click="assignSubstitute(teacher.id)" class="bg-slate-900 hover:bg-indigo-600 text-white px-4 py-2 rounded-xl text-xs font-semibold shadow-sm transition-all whitespace-nowrap">
                    SMART ASSIGN
                  </button>
                </div>
              </div>
            </div>

            <div v-if="currentLeaveItem && substituteAssignmentsMap[currentLeaveItem.id]" class="pt-2 border-t border-slate-100 flex justify-between items-center">
              <span class="text-xs text-red-500 font-medium">THIS SLOT ALREADY HAS A SUBSTITUTE / SWAP ASSIGNED</span>
              <button @click="removeAssignment" class="text-xs text-red-600 hover:text-red-800 font-bold px-3 py-1 bg-red-50 rounded-lg whitespace-nowrap">
                CANCEL CURRENT ASSIGNMENT
              </button>
            </div>

          </div>
        </div>
      </div>
    </transition>

    <!-- 动态附加的空白可编辑附页区域 -->
    <div v-for="(sheet, sIndex) in extraCustomSheets" :key="sheet.id" class="print-custom-sheet mt-12 print:mt-0 pt-8 print:pt-0 border-t-4 print:border-none border-dashed border-slate-300">
      
      <div class="print:hidden flex justify-between items-center mb-4 bg-amber-50 p-3 rounded-2xl border border-amber-200">
        <span class="text-xs font-bold text-amber-900">📄 ADDITIONAL / MANUAL TIMETABLE #{{ sIndex + 1 }}</span>
        <button @click="removeCustomSheet(sheet.id)" class="text-xs text-red-600 bg-white hover:bg-red-50 px-3 py-1.5 rounded-xl font-bold shadow-sm transition whitespace-nowrap">
          DELETE THIS TIMETABLE
        </button>
      </div>

      <div class="bg-white rounded-3xl shadow-sm ring-1 ring-slate-900/5 p-8 print:shadow-none print:ring-0 print:p-0 print:rounded-none print:break-inside-avoid">
        
        <div class="text-center mb-6 print:mb-2">
          <h2 class="text-xl font-black tracking-wider text-black font-serif">SJK (C) LADANG GRISEK</h2>
          <h3 class="text-lg font-bold tracking-widest text-black mt-1 font-serif underline">
            JADUAL GURU GANTI ({{ currentSession === 'morning' ? 'SESI PAGI' : 'SESI PETANG' }})
          </h3>
        </div>

        <div class="flex justify-between items-center mb-4 print:mb-2 font-bold text-sm font-serif border-b-2 border-black pb-2 print:pb-1">
          <div>
            <span class="underline underline-offset-4">TARIKH :</span> 
            <input v-model="sheet.date" @blur="saveCustomSheetsToCloud" type="text" placeholder="TARIKH" class="ml-2 border-b border-black px-2 py-0.5 text-sm font-normal w-32 focus:outline-none" />
          </div>
          <div>
            <span class="underline underline-offset-4">HARI :</span> 
            <input v-model="sheet.day" @blur="saveCustomSheetsToCloud" type="text" placeholder="HARI" class="ml-2 border-b border-black px-2 py-0.5 text-sm font-normal w-28 uppercase focus:outline-none" />
          </div>
        </div>

        <div class="w-full">
          <table class="w-full border-collapse border-2 border-black text-center text-xs font-serif table-fixed">
            <thead>
              <tr class="bg-slate-100 print:bg-white">
                <th class="border border-black p-1 w-28 font-bold" colspan="2">MASA</th>
                <th v-for="(time, index) in currentPeriodTimes" :key="index" class="border border-black p-1">
                  <div class="font-bold">{{ index + 1 }}</div>
                  <div class="text-[7px] font-normal mt-0.5 truncate">{{ time }}</div>
                </th>
              </tr>
            </thead>
            <tbody v-for="slotIndex in 5" :key="slotIndex" style="page-break-inside: avoid; break-inside: avoid;" class="print:break-inside-avoid">
                <tr>
                  <td contenteditable="true" 
                      @blur="saveManualEntry(`sheet_${sheet.id}_${slotIndex}`, 'name', 0, $event)"
                      v-text="getManualEntry(`sheet_${sheet.id}_${slotIndex}`, 'name', 0)"
                      class="border border-black p-1 font-bold bg-slate-50 print:bg-white align-middle text-center h-8 outline-none focus:bg-indigo-50/50 hover:bg-slate-100 cursor-text transition-colors overflow-hidden whitespace-pre-wrap break-words leading-tight uppercase text-[10px]" 
                      rowspan="3" style="width: 120px; max-width: 120px;"></td>
                  <td class="border border-black p-1 font-bold bg-slate-50 print:bg-white text-[10px]" style="width: 80px;">KELAS</td>
                  <td v-for="p in currentPeriodTimes.length" :key="'kelas-'+p" 
                      contenteditable="true" 
                      @blur="saveManualEntry(`sheet_${sheet.id}_${slotIndex}`, 'kelas', p, $event)"
                      v-text="getManualEntry(`sheet_${sheet.id}_${slotIndex}`, 'kelas', p)"
                      class="border border-black p-0.5 align-middle h-8 outline-none focus:bg-indigo-50/50 hover:bg-slate-100 cursor-text transition-colors text-[11px] font-semibold overflow-hidden whitespace-pre-wrap break-words leading-tight"
                      style="max-width: 0;"></td>
                </tr>
                <tr>
                  <td class="border border-black p-1 font-bold bg-slate-50 print:bg-white text-[10px]">GURU GANTI</td>
                  <td v-for="p in currentPeriodTimes.length" :key="'ganti-'+p" 
                      class="border border-black p-0.5 align-middle h-8 relative group" style="max-width: 0;">
                    <div class="w-full h-full relative flex items-center justify-center">
                      <div contenteditable="true" 
                          @blur="saveManualEntry(`sheet_${sheet.id}_${slotIndex}`, 'ganti', p, $event)"
                          v-text="getManualEntry(`sheet_${sheet.id}_${slotIndex}`, 'ganti', p)"
                          class="w-full h-full outline-none focus:bg-indigo-50/50 hover:bg-slate-100 cursor-text transition-colors text-[10px] font-bold text-indigo-900 overflow-hidden whitespace-pre-wrap break-words leading-tight flex items-center justify-center"></div>
                      <button contenteditable="false" @click.stop="openBlankModal(slotIndex, p, sheet.id)" class="print:hidden absolute right-0 top-0 hidden group-hover:flex bg-indigo-500 text-white rounded-bl px-1.5 py-0.5 text-[9px] cursor-pointer shadow-sm hover:bg-indigo-600 z-10 font-sans tracking-widest font-bold">ASSIGN</button>
                    </div>
                  </td>
                </tr>
                <tr>
                  <td class="border border-black p-1 font-bold bg-slate-50 print:bg-white text-[8px] whitespace-nowrap">T/TANGAN</td>
                  <td v-for="p in currentPeriodTimes.length" :key="'ttangan-'+p" 
                      contenteditable="true" 
                      @blur="saveManualEntry(`sheet_${sheet.id}_${slotIndex}`, 'ttangan', p, $event)"
                      v-text="getManualEntry(`sheet_${sheet.id}_${slotIndex}`, 'ttangan', p)"
                      class="border border-black p-1 align-middle h-8 outline-none focus:bg-indigo-50/50 hover:bg-slate-100 cursor-text transition-colors"></td>
                </tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>

    <!-- 放置在最底部的增加按钮 -->
    <div class="print:hidden mt-8 mb-12 flex justify-center w-full">
      <button @click="addBlankSheet" class="flex items-center gap-2 bg-slate-900 hover:bg-indigo-600 text-white px-8 py-3.5 rounded-2xl text-xs font-bold shadow-md transition-all whitespace-nowrap">
        <span class="text-base font-extrabold">+</span> ADD AN OFFICIAL BLANK TIMETABLE SHEET
      </button>
    </div>

    <!-- ⭐️ 新增弹窗：简易空白行代课指派 (支持虚拟负荷记录) (ENGLISH VERSION) -->
    <transition enter-active-class="transition duration-300 ease-out" enter-from-class="opacity-0 scale-95" enter-to-class="opacity-100 scale-100" leave-active-class="transition duration-200 ease-in" leave-from-class="opacity-100 scale-100" leave-to-class="opacity-0 scale-95">
      <div v-if="showBlankModal" class="print:hidden fixed inset-0 z-50 flex items-center justify-center p-4 sm:p-6">
        <div class="absolute inset-0 bg-slate-900/30 backdrop-blur-sm" @click="showBlankModal = false"></div>
        <div class="relative bg-white rounded-3xl shadow-2xl w-full max-w-lg overflow-hidden ring-1 ring-slate-900/10">
          
          <div class="px-6 py-5 border-b border-slate-100 flex justify-between items-center bg-slate-50">
            <div>
              <h2 class="text-lg font-bold text-slate-900 flex items-center gap-2"><span>📝 QUICK ASSIGN</span><span class="text-[10px] bg-indigo-100 text-indigo-700 px-2 py-0.5 rounded-full">SPECIAL TASK</span></h2>
            </div>
            <button @click="showBlankModal = false" class="text-slate-400 hover:text-slate-600 bg-white hover:bg-slate-200 rounded-full w-8 h-8 flex items-center justify-center transition cursor-pointer font-bold">✕</button>
          </div>
          
          <div class="p-6 space-y-5">
            <div>
              <label class="block text-xs font-bold text-slate-700 mb-2">🧑‍🏫 SELECT SUBSTITUTE TEACHER (SAME SESSION):</label>
              <select v-model="blankForm.teacherId" class="w-full px-3.5 py-2.5 bg-slate-50 border border-slate-200 rounded-xl text-xs font-semibold text-slate-800 focus:outline-none focus:ring-2 focus:ring-indigo-500 cursor-pointer">
                <option value="">-- NONE (LEAVE BLANK / TEXT ONLY) --</option>
                <option v-for="t in allSameSessionTeachers" :key="t.id" :value="t.id">
                  {{ t.name }} <span v-if="t.subject">({{ t.subject }})</span>
                </option>
              </select>
            </div>

            <div>
              <label class="block text-xs font-bold text-slate-700 mb-2">📍 REMARK (LOCATION/TASK E.G. DUTY):</label>
              <input v-model="blankForm.remark" type="text" placeholder="E.G.: PERPUSTAKAAN / LATIHAN SUKAN" class="w-full px-3.5 py-2.5 bg-slate-50 border border-slate-200 rounded-xl text-xs font-semibold text-slate-800 focus:outline-none focus:ring-2 focus:ring-indigo-500/50" />
            </div>

            <div class="bg-indigo-50/50 p-4 rounded-xl border border-indigo-100 flex items-start gap-3">
              <input type="checkbox" v-model="blankForm.kiraBeban" id="kiraBebanCb" class="mt-0.5 w-4 h-4 text-indigo-600 rounded cursor-pointer" />
              <div class="flex-1">
                <label for="kiraBebanCb" class="text-sm font-bold text-slate-800 cursor-pointer block mb-1">INCLUDE IN TEACHER'S WORKLOAD (KIRA BEBAN)</label>
                <p class="text-[10px] text-slate-500 font-medium leading-relaxed">If checked, the system will create a virtual record (does not affect MMI loss reports) and increase the teacher's daily class count by +1 in the background.</p>
              </div>
            </div>
          </div>

          <div class="px-6 py-4 bg-slate-50 border-t border-slate-100 flex justify-between items-center">
            <button v-if="hasExistingVirtual" @click="removeBlankAssignment" class="text-xs text-red-600 hover:text-red-800 font-bold px-4 py-2 bg-red-50 hover:bg-red-100 rounded-xl cursor-pointer transition">
              CLEAR CELL
            </button>
            <div v-else></div>
            <div class="flex gap-3">
              <button @click="showBlankModal = false" class="text-slate-500 hover:text-slate-700 px-4 py-2 text-xs font-bold transition cursor-pointer">CANCEL</button>
              <button @click="confirmBlankAssignment" class="bg-indigo-600 hover:bg-indigo-700 text-white px-5 py-2 rounded-xl text-xs font-bold shadow-sm transition-all cursor-pointer">
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
  Sparkles 
} from 'lucide-vue-next'

const toast = useToast()
const targetDate = ref(new Date().toISOString().split('T')[0])
const currentSession = ref('morning')

const morningTimes = [
  '7.00-7.30', '7.30-8.00', '8.00-8.30', '8.30-9.00', '9.00-9.30',
  '10.00-10.30', '10.30-11.00', '11.00-11.30', '11.30-12.00', '12.00-12.30', '12.30-1.00'
]
const afternoonTimes = [
  '1.00-1.30', '1.30-2.00', '2.00-2.30', '2.30-3.00', '3.00-3.30',
  '3.50-4.20', '4.20-4.50', '4.50-5.20', '5.20-5.50', '5.50-6.20'
]

const currentPeriodTimes = computed(() => currentSession.value === 'morning' ? morningTimes : afternoonTimes)
const dayNames = ['Ahad', 'Isnin', 'Selasa', 'Rabu', 'Khamis', 'Jumaat', 'Sabtu']

const leaveRequests = ref([])
const substituteAssignmentsMap = ref({})
const teachersMap = ref({})
const allSameSessionTeachers = ref([])

const showModal = ref(false)
const loadingRecs = ref(false)
const recommendations = ref([])
const currentLeaveItem = ref(null)
const assignmentRemark = ref('')
const assignmentType = ref('substitute')
const manualSelectedTeacherId = ref('')
const isAutoAssigning = ref(false)

// ⭐️ 新增：简易空白行弹窗状态
const showBlankModal = ref(false)
const blankTarget = ref({ slot: null, period: null, sheetId: null })
const blankForm = ref({ teacherId: '', remark: '', kiraBeban: true })
const hasExistingVirtual = ref(false)

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
    // ⭐️ 核心过滤：彻底屏蔽我们在后台偷偷建的“虚拟请假(VIRTUAL_CLASS)”记录，不让它们显示在原本的缺席名单中
    if (req.class_name === 'VIRTUAL_CLASS') return;

    const teacher = teachersMap.value[req.teacher_id]
    if (teacher && (teacher.session || 'morning') === currentSession.value) {
      
      // 🌟 核心修改点：在这里把原因里的 [个人请假] 等前缀过滤掉，只保留实际原因
      let cleanReason = (req.reason || '').replace(/\[.*?\]\s*/, '');
      
      map[req.teacher_id] = { 
        id: req.teacher_id, 
        name: teacher.name, 
        reason: cleanReason // 这里使用清理干净的原因
      }
    }
  })
  return Object.values(map)
})

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
        sessionCustomSheets.value[currentSession.value] = data.draft_data.__custom_sheets__
      }
    }
  } catch (err) {
    console.error("Failed to fetch drafts:", err)
  }
}

const saveCustomSheetsToCloud = async () => {
  manualEntries.value['__custom_sheets__'] = sessionCustomSheets.value[currentSession.value]
  try {
    await supabase
      .from('jadual_manual_drafts')
      .upsert({ 
        target_date: targetDate.value,
        session: currentSession.value,
        draft_data: manualEntries.value
      }, { onConflict: 'target_date,session' }) 
  } catch (err) {
    console.error("Failed to save custom sheets:", err)
  }
}

const saveManualEntry = async (slotIndex, type, period, event) => {
  const text = event.target.innerText.trim().replace(/\n+/g, '\n')
  const key = `${slotIndex}-${type}-${period}`
  
  if (manualEntries.value[key] === text) return;
  manualEntries.value[key] = text

  try {
    await supabase
      .from('jadual_manual_drafts')
      .upsert({ 
        target_date: targetDate.value,
        session: currentSession.value,
        draft_data: manualEntries.value
      }, { onConflict: 'target_date,session' }) 
  } catch (err) {
    console.error("Failed to save entry:", err)
  }
}

const getManualEntry = (slotIndex, type, period) => {
  const key = `${slotIndex}-${type}-${period}`
  return manualEntries.value[key] || ''
}

const fetchData = async () => {
  await fetchManualDrafts()

  const { data: tData } = await supabase.from('teachers').select('*')
  if (tData) {
    tData.forEach(t => { teachersMap.value[t.id] = t })
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
        sData.forEach(s => { map[s.leave_request_id] = s })
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
  return leaveRequests.value.some(r => r.teacher_id === teacherId && Number(r.period) === Number(periodNum) && r.class_name !== 'VIRTUAL_CLASS')
}

const getTeacherPeriodData = (teacherId, periodNum, type) => {
  const leaveItem = leaveRequests.value.find(r => r.teacher_id === teacherId && Number(r.period) === Number(periodNum) && r.class_name !== 'VIRTUAL_CLASS')
  if (!leaveItem) return ''

  if (type === 'class_subject') {
    return `${leaveItem.class_name} ${leaveItem.subject}`
  }

  if (type === 'substitute_name') {
    const subItem = substituteAssignmentsMap.value[leaveItem.id]
    if (!subItem || !subItem.sub_teacher_id) return ''
    
    const subTeacher = teachersMap.value[subItem.sub_teacher_id]
    let name = subTeacher ? subTeacher.name : ''
    
    if (subItem.assignment_type === 'swap') {
      name += ' ✦'
    }
    
    return subItem.remark ? `${name} (${subItem.remark})` : name
  }
  return ''
}

const loadSameSessionTeachers = async () => {
  if (allSameSessionTeachers.value.length === 0) {
    const { data } = await supabase
      .from('teachers')
      .select('*')
      .eq('is_active', true)
      .eq('session', currentSession.value)
    allSameSessionTeachers.value = data || []
  }
}

const handleCellClick = async (teacherId, periodNum) => {
  const leaveItem = leaveRequests.value.find(r => r.teacher_id === teacherId && Number(r.period) === Number(periodNum) && r.class_name !== 'VIRTUAL_CLASS')
  if (!leaveItem) {
    toast.error("No leave record found for this period!")
    return
  }

  currentLeaveItem.value = leaveItem
  assignmentRemark.value = ''
  manualSelectedTeacherId.value = ''
  assignmentType.value = 'substitute' 

  const existingSub = substituteAssignmentsMap.value[leaveItem.id]
  if (existingSub) {
    assignmentRemark.value = existingSub.remark || ''
    manualSelectedTeacherId.value = existingSub.sub_teacher_id || ''
    assignmentType.value = existingSub.assignment_type || 'substitute' 
  }

  showModal.value = true
  loadingRecs.value = true

  try {
    recommendations.value = await recommendSubstitute(leaveItem)
    await loadSameSessionTeachers()
  } catch (err) {
    toast.error("Failed to load scheduling data: " + err.message)
    recommendations.value = []
  } finally {
    loadingRecs.value = false
  }
}

const assignSubstitute = async (teacherId) => {
  if (!teacherId || !currentLeaveItem.value) return
  try {
    const leaveId = currentLeaveItem.value.id
    const existing = substituteAssignmentsMap.value[leaveId]

    const payload = {
      sub_teacher_id: teacherId, 
      remark: assignmentRemark.value ? assignmentRemark.value.trim() : null,
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

    await supabase.from('leave_requests').update({ status: 'assigned' }).eq('id', leaveId)

    toast.success(assignmentType.value === 'swap' ? "Swap assigned successfully!" : "Substitute assigned successfully!")
    showModal.value = false
    fetchData()
  } catch (err) {
    toast.error("Assignment failed: " + err.message)
  }
}

const removeAssignment = async () => {
  if (!currentLeaveItem.value) return
  try {
    const leaveId = currentLeaveItem.value.id
    const existing = substituteAssignmentsMap.value[leaveId]
    if (existing) {
      await supabase.from('substitute_assignments').delete().eq('id', existing.id)
      await supabase.from('leave_requests').update({ status: 'pending' }).eq('id', leaveId)
      toast.success("Assignment cancelled successfully")
      showModal.value = false
      fetchData()
    }
  } catch (err) {
    toast.error("Operation failed: " + err.message)
  }
}

const handleAutoAssignAll = async () => {
  const pendingRequests = leaveRequests.value.filter(req => {
    if (req.class_name === 'VIRTUAL_CLASS') return false; 

    const teacher = teachersMap.value[req.teacher_id]
    const inCurrentSession = teacher && (teacher.session || 'morning') === currentSession.value
    const notAssigned = !substituteAssignmentsMap.value[req.id] || !substituteAssignmentsMap.value[req.id].sub_teacher_id
    return inCurrentSession && notAssigned
  })

  if (pendingRequests.length === 0) {
    toast.success("No pending assignments for the current session!")
    return
  }

  isAutoAssigning.value = true
  let successCount = 0

  try {
    for (const req of pendingRequests) {
      const recs = await recommendSubstitute(req)
      if (recs && recs.length > 0) {
        const bestTeacherId = recs[0].id
        const { error: insertErr } = await supabase
          .from('substitute_assignments')
          .insert({
            leave_request_id: req.id,
            sub_teacher_id: bestTeacherId,
            remark: null,
            assignment_type: 'substitute'
          })
        if (!insertErr) {
          await supabase.from('leave_requests').update({ status: 'assigned' }).eq('id', req.id)
          successCount++
        }
      }
    }
    toast.success(`Success! Auto-assigned ${successCount} classes.`)
    fetchData()
  } catch (err) {
    toast.error("Error during auto-assignment: " + err.message)
  } finally {
    isAutoAssigning.value = false
  }
}

// ================= ⭐️ 新增：空白行与虚拟负荷指派逻辑 =================
const openBlankModal = async (slot, period, sheetId) => {
  blankTarget.value = { slot, period, sheetId }
  blankForm.value = { teacherId: '', remark: '', kiraBeban: true }
  
  const prefix = sheetId ? `sheet_${sheetId}_${slot}` : slot
  const virtualLeaveId = manualEntries.value[`${prefix}_virtual_leave_${period}`]
  
  hasExistingVirtual.value = !!virtualLeaveId

  if (virtualLeaveId) {
    const existingSub = substituteAssignmentsMap.value[virtualLeaveId]
    if (existingSub) {
      blankForm.value.teacherId = existingSub.sub_teacher_id || ''
      blankForm.value.remark = existingSub.remark || ''
      blankForm.value.kiraBeban = true
    }
  }

  await loadSameSessionTeachers()
  showBlankModal.value = true
}

const confirmBlankAssignment = async () => {
  const { slot, period, sheetId } = blankTarget.value
  const prefix = sheetId ? `sheet_${sheetId}_${slot}` : slot
  const textKey = `${prefix}-ganti-${period}`
  const virtualLeaveKey = `${prefix}_virtual_leave_${period}`

  if (!blankForm.value.teacherId && blankForm.value.kiraBeban) {
    return toast.error('Please select a teacher to include in the workload calculation!')
  }

  let teacherName = ''
  if (blankForm.value.teacherId) {
    const t = allSameSessionTeachers.value.find(x => x.id === blankForm.value.teacherId) || teachersMap.value[blankForm.value.teacherId]
    teacherName = t ? t.name : ''
  }
  
  let displayText = teacherName
  if (blankForm.value.remark) {
    displayText = teacherName ? `${teacherName} (${blankForm.value.remark})` : blankForm.value.remark
  }
  
  const existingVirtualLeaveId = manualEntries.value[virtualLeaveKey]

  const dateObj = new Date(targetDate.value)
  const dayNum = dateObj.getDay()
  const weekdayCalc = dayNum === 0 ? 7 : dayNum

  try {
    if (blankForm.value.kiraBeban) {
      // 1. Check if the teacher already has a record for this period
      const { data: existingLeave } = await supabase.from('leave_requests')
        .select('id, class_name')
        .eq('teacher_id', blankForm.value.teacherId)
        .eq('leave_date', targetDate.value)
        .eq('period', period)
        .maybeSingle()

      let targetLeaveId = null

      if (existingLeave) {
        if (existingLeave.class_name !== 'VIRTUAL_CLASS') {
          return toast.error('This teacher already has an official leave record for this period!')
        }
        targetLeaveId = existingLeave.id
        await supabase.from('leave_requests')
          .update({ reason: blankForm.value.remark || 'TUGAS KHAS' })
          .eq('id', targetLeaveId)
      } else {
        const { data: newLeave, error: leaveErr } = await supabase.from('leave_requests').insert({
          teacher_id: blankForm.value.teacherId, 
          leave_date: targetDate.value,
          weekday: weekdayCalc,
          period: period,
          reason: blankForm.value.remark || 'TUGAS KHAS',
          class_name: 'VIRTUAL_CLASS',
          subject: 'VIRTUAL_SUB',
          status: 'assigned'
        }).select().single()
        
        if (leaveErr) throw leaveErr
        targetLeaveId = newLeave.id
      }

      // 2. Sync substitute_assignments
      const { data: existingSub } = await supabase.from('substitute_assignments')
        .select('id')
        .eq('leave_request_id', targetLeaveId)
        .maybeSingle()

      if (existingSub) {
        await supabase.from('substitute_assignments')
          .update({ sub_teacher_id: blankForm.value.teacherId, remark: blankForm.value.remark })
          .eq('id', existingSub.id)
      } else {
        await supabase.from('substitute_assignments').insert({
          leave_request_id: targetLeaveId,
          sub_teacher_id: blankForm.value.teacherId,
          assignment_type: 'substitute',
          remark: blankForm.value.remark
        })
      }

      // 3. Garbage Collection for old teacher
      if (existingVirtualLeaveId && existingVirtualLeaveId !== targetLeaveId) {
        await supabase.from('substitute_assignments').delete().eq('leave_request_id', existingVirtualLeaveId)
        await supabase.from('leave_requests').delete().eq('id', existingVirtualLeaveId)
      }

      manualEntries.value[virtualLeaveKey] = targetLeaveId
    } else {
      if (existingVirtualLeaveId) {
        await supabase.from('substitute_assignments').delete().eq('leave_request_id', existingVirtualLeaveId)
        await supabase.from('leave_requests').delete().eq('id', existingVirtualLeaveId)
        delete manualEntries.value[virtualLeaveKey]
      }
    }

    manualEntries.value[textKey] = displayText
    await saveCustomSheetsToCloud() 
    
    toast.success(blankForm.value.kiraBeban ? 'Assigned successfully! Recorded in workload stats.' : 'Text saved successfully! Excluded from workload.')
    showBlankModal.value = false
    fetchData() 
  } catch (err) {
    toast.error('Save failed: ' + err.message)
  }
}

const removeBlankAssignment = async () => {
  const { slot, period, sheetId } = blankTarget.value
  const prefix = sheetId ? `sheet_${sheetId}_${slot}` : slot
  const virtualLeaveKey = `${prefix}_virtual_leave_${period}`
  const textKey = `${prefix}-ganti-${period}`
  const existingVirtualLeaveId = manualEntries.value[virtualLeaveKey]

  try {
    if (existingVirtualLeaveId) {
        await supabase.from('substitute_assignments').delete().eq('leave_request_id', existingVirtualLeaveId)
        await supabase.from('leave_requests').delete().eq('id', existingVirtualLeaveId)
        delete manualEntries.value[virtualLeaveKey]
    }
    manualEntries.value[textKey] = ''
    await saveCustomSheetsToCloud()
    
    toast.success('Cell cleared and workload assignment cancelled!')
    showBlankModal.value = false
    fetchData()
  } catch(err) {
    toast.error('Clear failed: ' + err.message)
  }
}
// =================================================================

watch([targetDate, currentSession], () => {
  fetchData()
})

onMounted(() => {
  fetchData()
})

const handlePrint = () => {
  window.print()
}

const extraCustomSheets = computed(() => {
  return sessionCustomSheets.value[currentSession.value] || []
})

const addBlankSheet = async () => {
  sessionCustomSheets.value[currentSession.value].push({
    id: Date.now(),
    date: '',
    day: '',
  })
  await saveCustomSheetsToCloud() 
}

const removeCustomSheet = async (id) => {
  const list = sessionCustomSheets.value[currentSession.value]
  const index = list.findIndex(sheet => sheet.id === id)
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
  
  body {
    background: white !important;
    -webkit-print-color-adjust: exact;
  }

  table, tbody, tr, td {
    page-break-inside: avoid !important;
    break-inside: avoid !important;
  }

  .force-page-break {
    break-after: page !important;
    page-break-after: always !important;
  }

  .print-custom-sheet {
    break-before: page !important;
    page-break-before: always !important;
  }
}
</style>