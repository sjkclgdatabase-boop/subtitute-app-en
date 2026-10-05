<template>
  <!-- KEEP MIN-W-[1024PX] TO ENSURE THE TABLE DOES NOT STRETCH INFINITELY -->
  <div class="p-4 sm:p-8 mx-auto min-h-screen space-y-8 min-w-[1024px] print:p-0 print:min-w-0 print:w-auto print:m-0 print:space-y-0">
    
    <!-- SCREEN ACTION BAR (AUTOMATICALLY HIDDEN DURING PRINTING) -->
    <div class="print:hidden bg-white rounded-3xl p-6 sm:p-8 shadow-sm ring-1 ring-slate-900/5 flex flex-col gap-6">
      
      <!-- TITLE & SUBTITLE -->
      <div class="space-y-2 max-w-4xl">
        <h1 class="text-2xl sm:text-3xl font-extrabold tracking-tight text-transparent bg-clip-text bg-gradient-to-r from-slate-900 via-indigo-800 to-violet-800 whitespace-nowrap flex items-center gap-3 uppercase">
          <UsersRound class="w-8 h-8 text-indigo-700 shrink-0" />
          DAILY SUBSTITUTE TEACHER MANAGEMENT
        </h1>
        <p class="text-slate-500 text-xs sm:text-sm font-medium leading-relaxed whitespace-nowrap uppercase">
          CLICK TIMETABLE CELLS TO ASSIGN SUBSTITUTE TEACHERS, SUPPORTS ONE-CLICK SMART AUTOMATIC SCHEDULING.
        </p>
      </div>

      <!-- ACTION BUTTONS BAR -->
      <div class="flex flex-wrap items-center gap-4 pt-4 border-t border-slate-100">
        
        <!-- 1. SMART AUTO-SCHEDULING -->
        <button 
          @click="handleAutoAssignAll"
          :disabled="isAutoAssigning"
          class="bg-indigo-600 hover:bg-indigo-700 disabled:opacity-50 text-white px-5 h-11 rounded-2xl text-xs font-bold shadow-sm transition-all flex items-center justify-center gap-2 cursor-pointer shrink-0 whitespace-nowrap uppercase"
        >
          <span v-if="isAutoAssigning" class="w-3.5 h-3.5 border-2 border-white/30 border-t-white rounded-full animate-spin"></span>
          <Zap v-else class="w-4 h-4" />
          <span>⚡ SMART SUBSTITUTE SCHEDULING</span>
        </button>

        <!-- 2. SESSION SWITCHER TABS (MORNING / AFTERNOON) -->
        <div class="flex bg-slate-100 p-1.5 rounded-2xl ring-1 ring-slate-900/5 h-11 items-center shrink-0 shadow-inner uppercase">
          <button 
            @click="currentSession = 'morning'" 
            :class="currentSession === 'morning' ? 'bg-slate-900 text-white shadow-sm' : 'text-slate-600 hover:text-slate-900'"
            class="px-5 py-2 rounded-xl text-xs font-bold transition-all cursor-pointer flex items-center justify-center gap-2 whitespace-nowrap uppercase"
          >
            <Sun class="w-4 h-4 text-amber-500" /> MORNING SESSION
          </button>
          <button 
            @click="currentSession = 'afternoon'" 
            :class="currentSession === 'afternoon' ? 'bg-slate-900 text-white shadow-sm' : 'text-slate-600 hover:text-slate-900'"
            class="px-5 py-2 rounded-xl text-xs font-bold transition-all cursor-pointer flex items-center justify-center gap-2 whitespace-nowrap uppercase"
          >
            <Moon class="w-4 h-4 text-indigo-400" /> AFTERNOON SESSION
          </button>
        </div>

        <!-- 3. DATE PICKER -->
        <div class="flex items-center gap-2 bg-slate-50 px-4 h-11 rounded-2xl border border-slate-200/80 shadow-2xs shrink-0 uppercase">
          <span class="text-xs font-bold text-slate-500 whitespace-nowrap uppercase">SELECT DATE:</span>
          <input 
            type="date" 
            v-model="targetDate" 
            class="bg-transparent text-xs font-bold text-slate-800 focus:outline-none cursor-pointer uppercase"
          />
        </div>

        <!-- 4. DIRECT PDF EXPORT -->
        <button 
          @click="handleExportPdf"
          :disabled="isExportingPdf"
          class="bg-slate-900 hover:bg-slate-800 disabled:opacity-50 text-white px-6 h-11 rounded-2xl text-xs font-bold shadow-sm transition-all flex items-center justify-center gap-2 shrink-0 cursor-pointer whitespace-nowrap uppercase"
        >
          <span v-if="isExportingPdf" class="w-4 h-4 border-2 border-white/30 border-t-white rounded-full animate-spin"></span>
          <Download v-else class="w-4 h-4" />
          <span>{{ isExportingPdf ? 'GENERATING PDF...' : 'DOWNLOAD PDF' }}</span>
        </button>

      </div>
    </div>

    <!-- MAIN TABLES: AUTOMATICALLY PAGINATED TO 5 ROWS PER PAGE -->
    <div 
      v-for="(pageTeachers, pageIndex) in paginatedTeacherPages" 
      :key="pageIndex"
      :class="[
        'print-main-sheet bg-white rounded-3xl shadow-sm ring-1 ring-slate-900/5 p-8 print:shadow-none print:ring-0 print:p-0 print:rounded-none',
        pageIndex > 0 ? 'mt-12 print:mt-0' : ''
      ]"
    >
      <!-- AUTO PAGE-BREAK FOR PRINTING IF NOT THE FIRST PAGE -->
      <div v-if="pageIndex > 0" class="print-page-break" aria-hidden="true"></div>

      <div class="text-center mb-6 print:mb-2">
        <h2 class="text-xl font-black tracking-wider text-black font-serif uppercase">
          {{ schoolName || 'SJK (C) LADANG GRISEK' }}
        </h2>

        <h3 class="text-lg font-bold tracking-widest text-black mt-1 font-serif underline uppercase">
          SUBSTITUTE TEACHER TIMETABLE ({{ currentSession === 'morning' ? 'MORNING SESSION' : 'AFTERNOON SESSION' }})
        </h3>
      </div>

      <div class="flex justify-between items-center mb-4 print:mb-2 font-bold text-sm font-serif border-b-2 border-black pb-2 print:pb-1 uppercase">
        <div>
          <span class="underline underline-offset-4">DATE :</span> <span class="ml-2 border-b border-black px-4">{{ formattedDate }}</span>
        </div>
        <div>
          <span class="underline underline-offset-4">DAY :</span> <span class="ml-2 border-b border-black px-4 uppercase">{{ formattedDayName }}</span>
        </div>
      </div>

      <div class="overflow-x-auto print:overflow-visible">
        <table class="w-full border-collapse border-2 border-black text-center text-xs font-serif table-fixed uppercase">
          <thead>
            <tr class="bg-slate-100 print:bg-white">
              <th class="border border-black p-1 font-bold uppercase" colspan="2" style="width: 130px; min-width: 130px; max-width: 130px;">TIME</th>

              <th
                v-for="(time, index) in currentPeriodTimes"
                :key="index"
                class="border border-black p-1 uppercase"
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
            class="print:break-inside-avoid uppercase"
          >
            
            <template v-if="pageTeachers[slotIndex - 1]">
              <tr>
                <td
                  class="border border-black p-1 bg-slate-50 print:bg-white align-middle text-center uppercase"
                  rowspan="3"
                  style="width: 85px; max-width: 85px;"
                >
                  <div class="flex flex-col items-center justify-center w-full px-0.5 uppercase">
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
                  class="border border-black p-1 font-bold bg-slate-50 print:bg-white text-[10px] uppercase"
                  style="width: 45px; max-width: 45px;"
                >
                  CLASS
                </td>
                
                <td
                  v-for="p in currentPeriodTimes.length"
                  :key="p"
                  class="border border-black p-0.5 font-semibold align-middle h-8 uppercase"
                  style="max-width: 0;"
                >
                  <div class="w-full h-full flex items-center justify-center px-0.5 overflow-hidden uppercase">
                    <span class="block w-full text-center text-[10px] tracking-tighter leading-tight text-slate-800 whitespace-normal uppercase">
                      {{ getTeacherPeriodData(pageTeachers[slotIndex - 1].id, p, 'class_subject') }}
                    </span>
                  </div>
                </td>
              </tr>

              <tr>
                <td class="border border-black p-1 font-bold bg-slate-50 print:bg-white text-[10px] uppercase">
                  SUBSTITUTE
                </td>

                <td
                  v-for="p in currentPeriodTimes.length"
                  :key="p"
                  @click="hasLeavePeriod(pageTeachers[slotIndex - 1].id, p) ? handleCellClick(pageTeachers[slotIndex - 1].id, p) : null"
                  :class="hasLeavePeriod(pageTeachers[slotIndex - 1].id, p) ? 'cursor-pointer hover:bg-indigo-50 group uppercase' : 'uppercase'"
                  class="print:hover:bg-transparent border border-black p-0.5 font-bold text-indigo-900 align-middle h-8 transition relative uppercase"
                  style="max-width: 0;"
                >
                  <div class="w-full h-full flex items-center justify-center px-0.5 overflow-hidden uppercase">
                    <span class="block w-full text-center text-[9px] tracking-tighter leading-tight whitespace-normal uppercase">
                      {{ getTeacherPeriodData(pageTeachers[slotIndex - 1].id, p, 'substitute_name') }}
                    </span>

                    <span
                      v-if="hasLeavePeriod(pageTeachers[slotIndex - 1].id, p)"
                      class="print:hidden hidden group-hover:inline-block text-[9px] text-indigo-500 absolute right-1 uppercase"
                    >
                      ✏️
                    </span>
                  </div>
                </td>
              </tr>

              <tr>
                <td class="border border-black p-1 font-bold bg-slate-50 print:bg-white text-[8px] whitespace-nowrap uppercase">
                  SIGNATURE
                </td>

                <td
                  v-for="p in currentPeriodTimes.length"
                  :key="p"
                  class="border border-black p-1 align-middle h-8 uppercase"
                ></td>
              </tr>
            </template>

            <!-- MANUAL ENTRY ROWS (SHOWN IF FEWER THAN 5 ABSENT TEACHERS) -->
            <template v-else>
              <tr>
                <!-- ⭐️ KLIK BUTANG EKSTRAK / PADAM -->
                <td
                  class="border border-black p-0 font-bold bg-slate-50 print:bg-white align-middle text-center h-8 relative group uppercase"
                  :style="{ width: '85px', maxWidth: '85px' }"
                  rowspan="3"
                >
                  <div class="w-full h-full relative flex items-center justify-center min-h-[70px] uppercase">
                    <div
                      contenteditable="true"
                      @blur="saveManualEntry(`page_${pageIndex}_${slotIndex}`, 'name', 0, $event)"
                      v-text="getManualEntry(`page_${pageIndex}_${slotIndex}`, 'name', 0)"
                      class="w-full h-full outline-none focus:bg-indigo-50/50 hover:bg-slate-100 cursor-text transition-colors whitespace-pre-wrap leading-tight uppercase flex items-center justify-center p-1 uppercase"
                      :style="getDynamicStyle(getManualEntry(`page_${pageIndex}_${slotIndex}`, 'name', 0), 10)"
                    ></div>

                    <div class="print:hidden absolute right-0 top-0 hidden group-hover:flex flex-col z-10 gap-[1px] uppercase">
                      <button
                        contenteditable="false"
                        @click.stop="openClassPicker(pageIndex, slotIndex, null)"
                        class="bg-emerald-500 text-white rounded-bl px-1.5 py-0.5 text-[9px] cursor-pointer shadow-sm hover:bg-emerald-600 font-sans tracking-widest font-bold uppercase"
                      >
                        CLASS
                      </button>
                      <button
                        contenteditable="false"
                        @click.stop="clearManualRow(pageIndex, slotIndex, null)"
                        class="bg-red-500 text-white rounded-l px-1.5 py-0.5 text-[9px] cursor-pointer shadow-sm hover:bg-red-600 font-sans tracking-widest font-bold uppercase"
                      >
                        CLEAR
                      </button>
                    </div>
                  </div>
                </td>

                <td
                  class="border border-black p-1 font-bold bg-slate-50 print:bg-white text-[10px] uppercase"
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
                  class="border border-black p-0.5 align-middle h-8 outline-none focus:bg-indigo-50/50 hover:bg-slate-100 cursor-text transition-colors font-semibold text-[11px] whitespace-pre-wrap leading-tight text-center uppercase"
                  style="max-width: 0;"
                ></td>
              </tr>

              <tr>
                <td class="border border-black p-1 font-bold bg-slate-50 print:bg-white text-[10px] uppercase">
                  SUBSTITUTE
                </td>

                <td
                  v-for="p in currentPeriodTimes.length"
                  :key="'ganti-'+p"
                  class="border border-black p-0.5 align-middle h-8 relative group uppercase"
                  style="max-width: 0;"
                >
                  <div class="w-full h-full relative flex items-center justify-center uppercase">
                    <div
                      contenteditable="true"
                      @blur="saveManualEntry(`page_${pageIndex}_${slotIndex}`, 'ganti', p, $event)"
                      v-text="getManualEntry(`page_${pageIndex}_${slotIndex}`, 'ganti', p)"
                      class="w-full h-full outline-none focus:bg-indigo-50/50 hover:bg-slate-100 cursor-text transition-colors font-bold text-indigo-900 text-[10px] whitespace-pre-wrap leading-tight flex items-center justify-center text-center uppercase"
                    ></div>

                    <button
                      contenteditable="false"
                      @click.stop="openBlankModal(`page_${pageIndex}_${slotIndex}`, p, null)"
                      class="print:hidden absolute right-0 top-0 hidden group-hover:flex bg-indigo-500 text-white rounded-bl px-1.5 py-0.5 text-[9px] cursor-pointer shadow-sm hover:bg-indigo-600 z-10 font-sans tracking-widest font-bold uppercase"
                    >
                      ASSIGN
                    </button>
                  </div>
                </td>
              </tr>

              <tr>
                <td class="border border-black p-1 font-bold bg-slate-50 print:bg-white text-[8px] whitespace-nowrap uppercase">
                  SIGNATURE
                </td>

                <td
                  v-for="p in currentPeriodTimes.length"
                  :key="'ttangan-'+p"
                  contenteditable="true"
                  @blur="saveManualEntry(`page_${pageIndex}_${slotIndex}`, 'ttangan', p, $event)"
                  v-text="getManualEntry(`page_${pageIndex}_${slotIndex}`, 'ttangan', p)"
                  class="border border-black p-1 align-middle h-8 outline-none focus:bg-indigo-50/50 hover:bg-slate-100 cursor-text transition-colors uppercase"
                ></td>
              </tr>
            </template>
          </tbody>
        </table>
      </div>

      <!-- ⭐️ BAHAGIAN CATATAN (DISUSUN MELINTANG KE KANAN PADA UI TANPA NOMBOR) -->
      <div v-if="remarksList.length > 0 && remarksList.some(r => r.trim())" class="mt-4 pt-3 border-t border-dashed border-slate-300 uppercase">
        <h4 class="text-xs font-bold text-black font-serif uppercase underline mb-2">REMARKS:</h4>
        <div class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 gap-4 uppercase">
          <template v-for="(rmk, rIdx) in remarksList" :key="rIdx">
            <div v-if="rmk.trim()" class="bg-slate-50 p-2.5 rounded-xl border border-slate-200 text-xs font-serif leading-relaxed uppercase">
              <div class="whitespace-pre-wrap text-slate-700 uppercase">{{ rmk }}</div>
            </div>
          </template>
        </div>
      </div>
    </div>

    <!-- ⭐️ KAWASAN PENGURUSAN CATATAN DINAMIK -->
    <div class="print:hidden bg-white rounded-3xl p-6 shadow-sm ring-1 ring-slate-900/5 space-y-4 uppercase">
      <div class="flex items-center justify-between uppercase">
        <label class="text-xs font-bold text-slate-700 uppercase tracking-wide flex items-center gap-2 uppercase">
          <span>📝 REMARKS MANAGEMENT</span>
        </label>
        <button
          @click="addRemarkBox"
          class="bg-emerald-600 hover:bg-emerald-700 text-white px-3.5 py-1.5 rounded-xl text-xs font-bold shadow-sm transition cursor-pointer flex items-center gap-1.5 whitespace-nowrap uppercase"
        >
          <span>+ ADD REMARK BOX</span>
        </button>
      </div>

      <div class="space-y-3 uppercase">
        <div v-for="(rmk, index) in remarksList" :key="index" class="flex items-start gap-3 bg-slate-50 p-3 rounded-2xl border border-slate-200 uppercase">
          <textarea
            v-model="remarksList[index]"
            @input="syncRemarksToGlobal"
            @blur="saveCustomSheetsToCloud"
            rows="3"
            placeholder="ENTER REMARKS HERE..."
            class="w-full p-3 bg-white border border-slate-200 rounded-xl text-xs font-medium text-slate-800 focus:outline-none focus:ring-2 focus:ring-indigo-500/50 resize-y uppercase"
          ></textarea>
          <button
            @click="removeRemarkBox(index)"
            class="text-xs text-red-600 hover:text-red-800 font-bold px-3 py-2 bg-white hover:bg-red-50 border border-red-200 rounded-xl cursor-pointer transition whitespace-nowrap shadow-2xs mt-1 uppercase"
          >
            DELETE
          </button>
        </div>
        <div v-if="remarksList.length === 0" class="text-center py-4 text-xs text-slate-400 font-medium uppercase">
          CLICK "+ ADD REMARK BOX" BUTTON ABOVE TO ADD YOUR FIRST REMARK BOX.
        </div>
      </div>
    </div>

    <!-- MODAL UTAMA: PUSAT PENETAPAN GURU GANTI -->
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
        class="print:hidden fixed inset-0 z-50 flex items-center justify-center p-4 sm:p-6 uppercase"
      >
        <div
          class="absolute inset-0 bg-slate-900/30 backdrop-blur-sm uppercase"
          @click="showModal = false"
        ></div>

        <div class="relative bg-white rounded-3xl shadow-2xl w-full max-w-2xl overflow-hidden ring-1 ring-slate-900/10 max-h-[90vh] flex flex-col uppercase">
          
          <div class="px-8 py-6 border-b border-slate-100 flex justify-between items-center bg-white/50 backdrop-blur-md shrink-0 uppercase">
            <div>
              <h2 class="text-xl font-bold text-slate-900 uppercase">SUBSTITUTE ASSIGNMENT CENTER</h2>
              <p class="text-sm text-slate-500 mt-1 uppercase">
                SUPPORTED BY SMART RECOMMENDATIONS, OR SELECT ANY SAME-SESSION TEACHER MANUALLY BELOW
              </p>
            </div>

            <button
              @click="showModal = false"
              class="text-slate-400 hover:text-slate-600 bg-slate-100 hover:bg-slate-200 rounded-full p-2 transition cursor-pointer uppercase"
            >
              ×
            </button>
          </div>
          
          <div class="p-8 bg-slate-50/50 space-y-6 overflow-y-auto uppercase">
            <div class="bg-white p-4 rounded-2xl border border-slate-200 shadow-sm uppercase">
              <h3 class="text-xs font-bold text-slate-700 mb-3 uppercase tracking-wider">
                🏷️ ASSIGNMENT TYPE:
              </h3>

              <div class="flex flex-col sm:flex-row gap-4 uppercase">
                <label class="flex items-center gap-2 cursor-pointer bg-slate-50 px-4 py-2 rounded-xl border border-slate-100 hover:bg-indigo-50 transition uppercase">
                  <input
                    type="radio"
                    v-model="assignmentType"
                    value="substitute"
                    class="text-indigo-600 focus:ring-indigo-500 w-4 h-4"
                  />
                  <span class="text-sm font-semibold text-slate-800 uppercase">
                    OFFICIAL SUBSTITUTE
                    <span class="text-xs text-slate-400 font-normal ml-1 uppercase">
                      (INCLUDED IN WORKLOAD STATS)
                    </span>
                  </span>
                </label>

                <label class="flex items-center gap-2 cursor-pointer bg-slate-50 px-4 py-2 rounded-xl border border-slate-100 hover:bg-indigo-50 transition uppercase">
                  <input
                    type="radio"
                    v-model="assignmentType"
                    value="swap"
                    class="text-indigo-600 focus:ring-indigo-500 w-4 h-4"
                  />
                  <span class="text-sm font-semibold text-slate-800 uppercase">
                    TIMETABLE SWAP
                    <span class="text-xs text-slate-400 font-normal ml-1 uppercase">
                      (EXCLUDED FROM STATS)
                    </span>
                  </span>
                </label>
              </div>
            </div>
          
            <div class="bg-white p-4 rounded-2xl border border-slate-200 shadow-sm flex items-center gap-3 uppercase">
              <span class="text-xs font-bold text-slate-700 whitespace-nowrap uppercase">
                📍 LOCATION / REMARK:
              </span>

              <input
                v-model="assignmentRemark"
                type="text"
                placeholder="E.G. LIBRARY (IF NEED TO BRING TO LIBRARY OR COMBINE CLASSES)"
                class="w-full px-3.5 py-2.5 bg-slate-50 border border-slate-200 rounded-xl text-xs font-semibold text-slate-800 focus:outline-none focus:ring-2 focus:ring-indigo-500/50 uppercase"
              />
            </div>

            <div class="bg-indigo-50/60 p-5 rounded-2xl border border-indigo-100 shadow-sm uppercase">
              <h3 class="text-xs font-bold uppercase tracking-wider text-indigo-900 mb-3 flex items-center gap-2 uppercase">
                <span>🛠️ MANUAL ASSIGNMENT (WITHOUT SMART RECOMMENDATION)</span>
              </h3>

              <div class="flex flex-col sm:flex-row items-center gap-3 uppercase">
                <select
                  v-model="manualSelectedTeacherId"
                  class="w-full px-3.5 py-2.5 bg-white border border-indigo-200 rounded-xl text-xs font-semibold text-slate-800 focus:outline-none focus:ring-2 focus:ring-indigo-500 cursor-pointer uppercase"
                >
                  <option value="" disabled class="uppercase">-- PLEASE SELECT A SAME-SESSION TEACHER MANUALLY --</option>

                  <option
                    v-for="t in allSameSessionTeachers"
                    :key="t.id"
                    :value="t.id"
                    class="uppercase"
                  >
                    {{ t.name }} <span v-if="t.subject">(SUBJECT: {{ t.subject }})</span>
                  </option>
                </select>

                <button
                  @click="assignSubstitute(manualSelectedTeacherId)"
                  :disabled="!manualSelectedTeacherId"
                  class="w-full sm:w-auto bg-indigo-600 hover:bg-indigo-700 disabled:opacity-50 text-white px-5 py-2.5 rounded-xl text-xs font-semibold shadow-sm transition-all shrink-0 cursor-pointer whitespace-nowrap uppercase"
                >
                  CONFIRM MANUAL ASSIGNMENT
                </button>
              </div>
            </div>

            <hr class="border-slate-200" />

            <div class="uppercase">
              <div class="flex justify-between items-center mb-3 uppercase">
                <h3 class="text-xs font-bold uppercase tracking-wider text-slate-500 flex items-center gap-2 uppercase">
                  <Sparkles class="w-4 h-4 text-indigo-600" />
                  ✨ SMART RECOMMENDATION CANDIDATES (TOTAL {{ recommendations.length }})
                </h3>
                <span v-if="recommendations.length > 0" class="text-[11px] text-slate-400 font-semibold uppercase">
                  PAGE {{ recCurrentPage }} / {{ recTotalPages }}
                </span>
              </div>
              
              <div
                v-if="loadingRecs"
                class="flex flex-col items-center justify-center py-6 space-y-3 uppercase"
              >
                <div class="w-6 h-6 border-4 border-indigo-500/30 border-t-indigo-600 rounded-full animate-spin uppercase"></div>
                <p class="text-xs text-slate-500 font-medium uppercase">SMART ALGORITHM IS CALCULATING...</p>
              </div>
              
              <div
                v-else-if="recommendations.length === 0"
                class="bg-white p-4 rounded-2xl border border-slate-200 text-xs text-slate-500 text-center uppercase"
              >
                NO AUTOMATIC RECOMMENDATIONS AVAILABLE, PLEASE USE MANUAL ASSIGNMENT ABOVE.
              </div>

              <div v-else class="space-y-3 uppercase">
                <div
                  v-for="(teacher, index) in paginatedRecommendations"
                  :key="teacher.id"
                  :class="[
                    'group flex flex-col sm:flex-row sm:justify-between sm:items-center p-4 border rounded-2xl transition-all uppercase',
                    teacher.isBusy ? 'bg-red-50/30 border-red-100 uppercase' : 'bg-white border-slate-200 hover:border-indigo-300 hover:shadow-sm uppercase'
                  ]"
                >
                  <div class="flex items-center gap-3 mb-3 sm:mb-0 uppercase">
                    <div :class="[
                      'w-8 h-8 rounded-full font-extrabold flex items-center justify-center text-xs uppercase',
                      teacher.isBusy ? 'bg-red-100 text-red-600 uppercase' : 'bg-gradient-to-br from-indigo-100 to-violet-100 text-indigo-700 uppercase'
                    ]">
                      #{{ (recCurrentPage - 1) * recPageSize + index + 1 }}
                    </div>

                    <div class="uppercase">
                      <div class="font-bold text-slate-900 text-sm flex items-center gap-2 uppercase">
                        {{ teacher.name }}
                        <span v-if="teacher.isBusy" class="text-[10px] text-red-600 bg-red-100 px-2 py-0.5 rounded-full font-bold uppercase">
                          HAS CLASS
                        </span>
                      </div>

                      <div class="text-[11px] text-slate-500 mt-1 flex items-center gap-2 flex-wrap uppercase">
                        <span>
                          ORIGINAL DAILY CLASSES:
                          <span class="font-bold text-slate-700 uppercase">
                            {{ teacher.originalClasses }} CLASSES
                          </span>
                        </span>

                        <span>·</span>

                        <span>
                          TODAY'S SUBSTITUTIONS:
                          <span class="font-bold text-orange-600 uppercase">
                            {{ teacher.todaySubCount }} CLASSES
                          </span>
                        </span>

                        <span>·</span>

                        <span>
                          THIS WEEK'S SUBSTITUTIONS:
                          <span class="font-bold text-slate-700 uppercase">
                            {{ teacher.currentSubCount }}{{ teacher.currentSubCount !== '-' ? '/' : '' }}{{ teacher.currentSubCount !== '-' ? teacher.max_substitute_per_week : '' }}
                          </span>
                        </span>
                      </div>
                    </div>
                  </div>

                  <button
                    @click="assignSubstitute(teacher.id)"
                    :class="[
                      'px-4 py-2 rounded-xl text-xs font-semibold shadow-sm transition-all cursor-pointer whitespace-nowrap uppercase',
                      teacher.isBusy ? 'bg-red-100 text-red-700 hover:bg-red-200 uppercase' : 'bg-slate-900 hover:bg-indigo-600 text-white uppercase'
                    ]"
                  >
                    {{ teacher.isBusy ? 'FORCE ASSIGN' : 'SMART ASSIGN' }}
                  </button>
                </div>

                <!-- Pagination controls -->
                <div v-if="recTotalPages > 1" class="flex items-center justify-between pt-2 px-1 uppercase">
                  <button 
                    @click="recCurrentPage = Math.max(1, recCurrentPage - 1)"
                    :disabled="recCurrentPage === 1"
                    class="px-3 py-1.5 bg-white border border-slate-200 rounded-xl text-xs font-bold text-slate-700 disabled:opacity-40 disabled:cursor-not-allowed hover:bg-slate-100 transition cursor-pointer uppercase"
                  >
                    PREVIOUS
                  </button>
                  <span class="text-xs font-semibold text-slate-500 uppercase">
                    PAGE {{ recCurrentPage }} / {{ recTotalPages }}
                  </span>
                  <button 
                    @click="recCurrentPage = Math.min(recTotalPages, recCurrentPage + 1)"
                    :disabled="recCurrentPage === recTotalPages"
                    class="px-3 py-1.5 bg-white border border-slate-200 rounded-xl text-xs font-bold text-slate-700 disabled:opacity-40 disabled:cursor-not-allowed hover:bg-slate-100 transition cursor-pointer uppercase"
                  >
                    NEXT
                  </button>
                </div>
              </div>
            </div>

            <div
              v-if="currentLeaveItem && substituteAssignmentsMap[currentLeaveItem.id]"
              class="pt-2 border-t border-slate-100 flex justify-between items-center uppercase"
            >
              <span class="text-xs text-red-500 font-medium uppercase">
                THIS SLOT ALREADY HAS A SUBSTITUTE / SWAP ASSIGNED
              </span>

              <button
                @click="removeAssignment"
                class="text-xs text-red-600 hover:text-red-800 font-bold px-3 py-1 bg-red-50 rounded-lg cursor-pointer whitespace-nowrap uppercase"
              >
                CANCEL CURRENT ASSIGNMENT
              </button>
            </div>
          </div>
        </div>
      </div>
    </transition>

    <!-- ⭐️ ADDITIONAL MANUAL BLANK SHEETS -->
    <div
      v-for="(sheet, sIndex) in extraCustomSheets"
      :key="sheet.id"
      class="uppercase"
    >
      <!-- Page break for printing -->
      <div class="print-page-break" aria-hidden="true"></div>

      <div class="print-custom-sheet mt-12 print:mt-0 pt-8 print:pt-0 border-t-4 print:border-none border-dashed border-slate-300 uppercase">
      
        <div class="print:hidden flex justify-between items-center mb-4 bg-amber-50 p-3 rounded-2xl border border-amber-200 uppercase">
          <span class="text-xs font-bold text-amber-900 uppercase">
            📄 ADDITIONAL / MANUAL TIMETABLE #{{ sIndex + 1 }}
          </span>

          <button
            @click="removeCustomSheet(sheet.id)"
            class="text-xs text-red-600 bg-white hover:bg-red-50 px-3 py-1.5 rounded-xl font-bold shadow-sm transition cursor-pointer whitespace-nowrap uppercase"
          >
            DELETE THIS TIMETABLE
          </button>
        </div>

        <div class="bg-white rounded-3xl shadow-sm ring-1 ring-slate-900/5 p-8 print:shadow-none print:ring-0 print:p-0 print:rounded-none print:break-inside-avoid uppercase">
          
          <div class="text-center mb-6 print:mb-2 uppercase">
            <h2 class="text-xl font-black tracking-wider text-black font-serif uppercase">
              {{ schoolName || 'SJK (C) LADANG GRISEK' }}
            </h2>

            <h3 class="text-lg font-bold tracking-widest text-black mt-1 font-serif underline uppercase">
              SUBSTITUTE TEACHER TIMETABLE ({{ currentSession === 'morning' ? 'MORNING SESSION' : 'AFTERNOON SESSION' }})
            </h3>
          </div>

          <div class="flex justify-between items-center mb-4 print:mb-2 font-bold text-sm font-serif border-b-2 border-black pb-2 print:pb-1 uppercase">
            <div>
              <span class="underline underline-offset-4 uppercase">DATE :</span>

              <input
                v-model="sheet.date"
                @blur="saveCustomSheetsToCloud"
                type="text"
                placeholder="DATE"
                class="ml-2 border-b border-black px-2 py-0.5 text-sm font-normal w-32 focus:outline-none uppercase"
              />
            </div>

            <div>
              <span class="underline underline-offset-4 uppercase">DAY :</span>

              <input
                v-model="sheet.day"
                @blur="saveCustomSheetsToCloud"
                type="text"
                placeholder="DAY"
                class="ml-2 border-b border-black px-2 py-0.5 text-sm font-normal w-28 uppercase focus:outline-none uppercase"
              />
            </div>
          </div>

          <div class="w-full overflow-x-auto print:overflow-visible uppercase">
            <table class="w-full border-collapse border-2 border-black text-center text-xs font-serif table-fixed uppercase">
              <thead>
                <tr class="bg-slate-100 print:bg-white uppercase">
                  <th
                    class="border border-black p-1 font-bold uppercase"
                    colspan="2"
                    style="width: 130px; min-width: 130px; max-width: 130px;"
                  >
                    TIME
                  </th>

                  <th
                    v-for="(time, index) in currentPeriodTimes"
                    :key="index"
                    class="border border-black p-1 uppercase"
                  >
                    <div class="font-bold uppercase">{{ index + 1 }}</div>
                    <div class="text-[7px] font-normal mt-0.5 truncate uppercase">
                      {{ time }}
                    </div>
                  </th>
                </tr>
              </thead>

              <tbody
                v-for="slotIndex in 5"
                :key="slotIndex"
                class="uppercase"
              >
                <tr>
                  <!-- ⭐ Butang ekstrak kelas / butang padam (Kawasan tambahan) -->
                  <td
                    class="border border-black p-0 font-bold bg-slate-50 print:bg-white align-middle text-center h-8 relative group uppercase"
                    :style="{ width: '85px', maxWidth: '85px' }"
                    rowspan="3"
                  >
                    <div class="w-full h-full relative flex items-center justify-center min-h-[70px] uppercase">
                      <div
                        contenteditable="true"
                        @blur="saveManualEntry(`sheet_${sheet.id}_${slotIndex}`, 'name', 0, $event)"
                        v-text="getManualEntry(`sheet_${sheet.id}_${slotIndex}`, 'name', 0)"
                        class="w-full h-full outline-none focus:bg-indigo-50/50 hover:bg-slate-100 cursor-text transition-colors whitespace-pre-wrap leading-tight uppercase flex items-center justify-center p-1 uppercase"
                        :style="getDynamicStyle(getManualEntry(`sheet_${sheet.id}_${slotIndex}`, 'name', 0), 10)"
                      ></div>

                      <div class="print:hidden absolute right-0 top-0 hidden group-hover:flex flex-col z-10 gap-[1px] uppercase">
                        <button
                          contenteditable="false"
                          @click.stop="openClassPicker(null, slotIndex, sheet.id)"
                          class="bg-emerald-500 text-white rounded-bl px-1.5 py-0.5 text-[9px] cursor-pointer shadow-sm hover:bg-emerald-600 font-sans tracking-widest font-bold uppercase"
                        >
                          CLASS
                        </button>
                        <button
                          contenteditable="false"
                          @click.stop="clearManualRow(null, slotIndex, sheet.id)"
                          class="bg-red-500 text-white rounded-l px-1.5 py-0.5 text-[9px] cursor-pointer shadow-sm hover:bg-red-600 font-sans tracking-widest font-bold uppercase"
                        >
                          CLEAR
                        </button>
                      </div>
                    </div>
                  </td>

                  <td
                    class="border border-black p-1 font-bold bg-slate-50 print:bg-white text-[10px] uppercase"
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
                    class="border border-black p-0.5 align-middle h-8 outline-none focus:bg-indigo-50/50 hover:bg-slate-100 cursor-text transition-colors font-semibold text-[11px] whitespace-pre-wrap leading-tight text-center uppercase"
                    style="max-width: 0;"
                  ></td>
                </tr>

                <tr>
                  <td class="border border-black p-1 font-bold bg-slate-50 print:bg-white text-[10px] uppercase">
                    SUBSTITUTE
                  </td>

                  <td
                    v-for="p in currentPeriodTimes.length"
                    :key="'ganti-'+p"
                    class="border border-black p-0.5 align-middle h-8 relative group uppercase"
                    style="max-width: 0;"
                  >
                    <div class="w-full h-full relative flex items-center justify-center uppercase">
                      <div
                        contenteditable="true"
                        @blur="saveManualEntry(`sheet_${sheet.id}_${slotIndex}`, 'ganti', p, $event)"
                        v-text="getManualEntry(`sheet_${sheet.id}_${slotIndex}`, 'ganti', p)"
                        class="w-full h-full outline-none focus:bg-indigo-50/50 hover:bg-slate-100 cursor-text transition-colors font-bold text-indigo-900 text-[10px] whitespace-pre-wrap leading-tight flex items-center justify-center text-center uppercase"
                      ></div>

                      <button
                        contenteditable="false"
                        @click.stop="openBlankModal(slotIndex, p, sheet.id)"
                        class="print:hidden absolute right-0 top-0 hidden group-hover:flex bg-indigo-500 text-white rounded-bl px-1.5 py-0.5 text-[9px] cursor-pointer shadow-sm hover:bg-indigo-600 z-10 font-sans tracking-widest font-bold uppercase"
                      >
                        ASSIGN
                      </button>
                    </div>
                  </td>
                </tr>

                <tr>
                  <td class="border border-black p-1 font-bold bg-slate-50 print:bg-white text-[8px] whitespace-nowrap uppercase">
                    SIGNATURE
                  </td>

                  <td
                    v-for="p in currentPeriodTimes.length"
                    :key="'ttangan-'+p"
                    contenteditable="true"
                    @blur="saveManualEntry(`sheet_${sheet.id}_${slotIndex}`, 'ttangan', p, $event)"
                    v-text="getManualEntry(`sheet_${sheet.id}_${slotIndex}`, 'ttangan', p)"
                    class="border border-black p-1 align-middle h-8 outline-none focus:bg-indigo-50/50 hover:bg-slate-100 cursor-text transition-colors uppercase"
                  ></td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>
      </div>
    </div>

    <!-- BUTANG TAMBAH DI BAHAGIAN BAWAH -->
    <div class="print:hidden mt-8 mb-12 flex justify-center w-full uppercase">
      <button
        @click="addBlankSheet"
        class="flex items-center gap-2 bg-slate-900 hover:bg-indigo-600 text-white px-8 py-3.5 rounded-2xl text-xs font-bold shadow-md transition-all cursor-pointer whitespace-nowrap uppercase"
      >
        <span class="text-base font-extrabold uppercase">+</span>
        ADD AN OFFICIAL BLANK TIMETABLE SHEET
      </button>
    </div>

    <!-- ⭐️ MODAL KHAS UNTUK EKSTRAK SUBJEK KELAS BAGI BARIS KESELURUHAN -->
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
        class="print:hidden fixed inset-0 z-50 flex items-center justify-center p-4 sm:p-6 uppercase"
      >
        <div class="absolute inset-0 bg-slate-900/30 backdrop-blur-sm uppercase" @click="showClassPickerModal = false"></div>
        <div class="relative bg-white rounded-3xl shadow-2xl w-full max-w-sm overflow-hidden ring-1 ring-slate-900/10 uppercase">
          
          <div class="px-6 py-5 border-b border-slate-100 flex justify-between items-center bg-slate-50 uppercase">
            <h2 class="text-lg font-bold text-slate-900 flex items-center gap-2 uppercase">
              <span>📚 EXTRACT FULL DAY CLASS SUBJECTS</span>
            </h2>
            <button @click="showClassPickerModal = false" class="text-slate-400 hover:text-slate-600 bg-white hover:bg-slate-200 rounded-full w-8 h-8 flex items-center justify-center transition cursor-pointer font-bold uppercase">
              ✕
            </button>
          </div>
          
          <div class="p-6 space-y-5 uppercase">
            <div>
              <label class="block text-xs font-bold text-slate-700 mb-2 uppercase">SELECT CLASS TO EXTRACT:</label>
              <select v-model="selectedClassToFill" class="w-full px-3.5 py-2.5 bg-slate-50 border border-slate-200 rounded-xl text-xs font-semibold text-slate-800 focus:outline-none focus:ring-2 focus:ring-emerald-500 cursor-pointer uppercase">
                <option value="" class="uppercase">-- PLEASE SELECT A CLASS --</option>
                <option v-for="cls in allClassesList" :key="cls" :value="cls" class="uppercase">{{ cls }}</option>
              </select>
            </div>
          </div>

          <div class="px-6 py-4 bg-slate-50 border-t border-slate-100 flex justify-end gap-3 uppercase">
            <button @click="showClassPickerModal = false" class="text-slate-500 hover:text-slate-700 px-4 py-2 text-xs font-bold transition cursor-pointer uppercase">
              CANCEL
            </button>
            <button @click="confirmClassPicker" class="bg-emerald-600 hover:bg-emerald-700 text-white px-5 py-2 rounded-xl text-xs font-bold shadow-sm transition-all cursor-pointer uppercase">
              CONFIRM EXTRACTION
            </button>
          </div>

        </div>
      </div>
    </transition>

    <!-- MODAL TUGASAN GURU GANTI RINGKAS -->
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
        class="print:hidden fixed inset-0 z-50 flex items-center justify-center p-4 sm:p-6 uppercase"
      >
        <div
          class="absolute inset-0 bg-slate-900/30 backdrop-blur-sm uppercase"
          @click="showBlankModal = false"
        ></div>

        <div class="relative bg-white rounded-3xl shadow-2xl w-full max-w-lg overflow-hidden ring-1 ring-slate-900/10 uppercase">
          
          <div class="px-6 py-5 border-b border-slate-100 flex justify-between items-center bg-slate-50 uppercase">
            <div>
              <h2 class="text-lg font-bold text-slate-900 flex items-center gap-2 uppercase">
                <span>📝 QUICK ASSIGN</span>
                <span class="text-[10px] bg-indigo-100 text-indigo-700 px-2 py-0.5 rounded-full uppercase">
                  SPECIAL TASK
                </span>
              </h2>
            </div>

            <button
              @click="showBlankModal = false"
              class="text-slate-400 hover:text-slate-600 bg-white hover:bg-slate-200 rounded-full w-8 h-8 flex items-center justify-center transition cursor-pointer font-bold uppercase"
            >
              ✕
            </button>
          </div>
          
          <div class="p-6 space-y-5 uppercase">
            <div>
              <label class="block text-xs font-bold text-slate-700 mb-2 uppercase">
                🧑‍🏫 SELECT SUBSTITUTE TEACHER (SAME SESSION):
              </label>

              <select
                v-model="blankForm.teacherId"
                class="w-full px-3.5 py-2.5 bg-slate-50 border border-slate-200 rounded-xl text-xs font-semibold text-slate-800 focus:outline-none focus:ring-2 focus:ring-indigo-500 cursor-pointer uppercase"
              >
                <option value="" class="uppercase">-- NONE (LEAVE BLANK / TEXT ONLY) --</option>

                <option
                  v-for="t in allSameSessionTeachers"
                  :key="t.id"
                  :value="t.id"
                  class="uppercase"
                >
                  {{ t.name }} <span v-if="t.subject">({{ t.subject }})</span>
                </option>
              </select>
            </div>

            <div>
              <label class="block text-xs font-bold text-slate-700 mb-2 uppercase">
                📍 REMARK (LOCATION/TASK E.G. DUTY):
              </label>

              <input
                v-model="blankForm.remark"
                type="text"
                placeholder="E.G.: PERPUSTAKAAN / LATIHAN SUKAN"
                class="w-full px-3.5 py-2.5 bg-slate-50 border border-slate-200 rounded-xl text-xs font-semibold text-slate-800 focus:outline-none focus:ring-2 focus:ring-indigo-500/50 uppercase"
              />
            </div>

            <div class="bg-indigo-50/50 p-4 rounded-xl border border-indigo-100 flex items-start gap-3 uppercase">
              <input
                type="checkbox"
                v-model="blankForm.kiraBeban"
                id="kiraBebanCb"
                class="mt-0.5 w-4 h-4 text-indigo-600 rounded cursor-pointer uppercase"
              />

              <div class="flex-1 uppercase">
                <label
                  for="kiraBebanCb"
                  class="text-sm font-bold text-slate-800 cursor-pointer block mb-1 uppercase"
                >
                  INCLUDE IN TEACHER'S WORKLOAD
                </label>

                <p class="text-[10px] text-slate-500 font-medium leading-relaxed uppercase">
                  IF CHECKED, THE SYSTEM WILL CREATE A VIRTUAL RECORD (DOES NOT AFFECT MMI LOSS REPORTS) AND INCREASE THE TEACHER'S DAILY CLASS COUNT BY +1 IN THE BACKGROUND.
                </p>
              </div>
            </div>
          </div>

          <div class="px-6 py-4 bg-slate-50 border-t border-slate-100 flex justify-between items-center uppercase">
            <button
              v-if="hasExistingVirtual"
              @click="removeBlankAssignment"
              class="text-xs text-red-600 hover:text-red-800 font-bold px-4 py-2 bg-red-50 hover:bg-red-100 rounded-xl cursor-pointer transition whitespace-nowrap uppercase"
            >
              CLEAR CELL
            </button>

            <div v-else></div>

            <div class="flex gap-3 uppercase">
              <button
                @click="showBlankModal = false"
                class="text-slate-500 hover:text-slate-700 px-4 py-2 text-xs font-bold transition cursor-pointer uppercase"
              >
                CANCEL
              </button>

              <button
                @click="confirmBlankAssignment"
                class="bg-indigo-600 hover:bg-indigo-700 text-white px-5 py-2 rounded-xl text-xs font-bold shadow-sm transition-all cursor-pointer uppercase"
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
 * Nama Sekolah
 * Sumber: Supabase -> school_settings -> id = 1
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
  'AHAD',
  'ISNIN',
  'SELASA',
  'RABU',
  'KHAMIS',
  'JUMAAT',
  'SABTU'
]

const leaveRequests = ref([])
const substituteAssignmentsMap = ref({})
const teachersMap = ref({})
const allSameSessionTeachers = ref([])

// Status Modal Utama
const showModal = ref(false)
const loadingRecs = ref(false)
const recommendations = ref([])
const currentLeaveItem = ref(null)
const assignmentRemark = ref('')
const assignmentType = ref('substitute')
const manualSelectedTeacherId = ref('')
const isAutoAssigning = ref(false)
const isExportingPdf = ref(false)

// Status pagination senarai cadangan (maksimum 10 setiap halaman)
const recCurrentPage = ref(1)
const recPageSize = 10

const recTotalPages = computed(() => {
  return Math.max(1, Math.ceil(recommendations.value.length / recPageSize))
})

const paginatedRecommendations = computed(() => {
  const start = (recCurrentPage.value - 1) * recPageSize
  return recommendations.value.slice(start, start + recPageSize)
})

// Status modal baris kosong
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

// Status pengekstrakan subjek kelas
const classSchedulesMap = ref({})
const allClassesList = ref([])

// Status modal untuk pengekstrakan seluruh baris
const showClassPickerModal = ref(false)
const classPickerTarget = ref({
  pageIndex: null,
  slotIndex: null,
  sheetId: null
})
const selectedClassToFill = ref('')

// ⭐️ Senarai Catatan Dinamik (Array of Textareas)
const remarksList = ref([''])

// Sinkronisasi senarai array ke string global (huruf besar)
const syncRemarksToGlobal = () => {
  globalPageRemark.value = remarksList.value.map(r => r ? r.toUpperCase() : '').join('|||')
}

// Tambah satu kotak teks catatan baru
const addRemarkBox = () => {
  remarksList.value.push('')
  syncRemarksToGlobal()
  saveCustomSheetsToCloud()
}

// Padam kotak teks catatan tertentu
const removeRemarkBox = (index) => {
  remarksList.value.splice(index, 1)
  if (remarksList.value.length === 0) {
    remarksList.value = ['']
  }
  syncRemarksToGlobal()
  saveCustomSheetsToCloud()
}

// Pembolehubah string global untuk simpanan awan
const globalPageRemark = ref('')

// =====================================================
// Identiti Sekolah
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
      (data?.school_name?.trim() ||
      localStorage.getItem('school_name')?.trim() ||
      'SJK (C) LADANG GRISEK').toUpperCase()

  } catch (err) {
    console.error('GAGAL MEMUATKAN NAMA SEKOLAH:', err)

    schoolName.value =
      (localStorage.getItem('school_name')?.trim() ||
      'SJK (C) LADANG GRISEK').toUpperCase()
  }
}

// Logik untuk mengecilkan saiz nama dan sebab ketiadaan secara dinamik
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
    .toUpperCase()
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

  // Mekanisme pengecilan tulisan
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
        .toUpperCase()

      map[req.teacher_id] = {
        id: req.teacher_id,
        name: teacher.name ? teacher.name.toUpperCase() : '',
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

// ================= Draf Manual & Jadual Tambahan (Penyimpanan Awan) =================

const manualEntries = ref({})

const sessionCustomSheets = ref({
  morning: [],
  afternoon: []
})

const fetchManualDrafts = async () => {
  manualEntries.value = {}
  sessionCustomSheets.value[currentSession.value] = []
  globalPageRemark.value = ''
  remarksList.value = ['']

  try {
    const { data, error } = await supabase
      .from('jadual_manual_drafts')
      .select('draft_data')
      .eq('target_date', targetDate.value)
      .eq('session', currentSession.value)
      .maybeSingle()

    if (data && data.draft_data) {
      const upperDraft = {}
      for (const k in data.draft_data) {
        if (k === '__custom_sheets__' && Array.isArray(data.draft_data[k])) {
          upperDraft[k] = data.draft_data[k].map(sheet => ({
            ...sheet,
            day: sheet.day ? sheet.day.toUpperCase() : ''
          }))
        } else if (typeof data.draft_data[k] === 'string') {
          upperDraft[k] = data.draft_data[k].toUpperCase()
        } else {
          upperDraft[k] = data.draft_data[k]
        }
      }
      manualEntries.value = upperDraft

      if (upperDraft.__custom_sheets__) {
        sessionCustomSheets.value[currentSession.value] = upperDraft.__custom_sheets__
      }
      if (upperDraft.__global_remark__) {
        globalPageRemark.value = upperDraft.__global_remark__.toUpperCase()
        // 关键修复：从数据库读取时，根据特殊分隔符 '|||' 来拆分还原成数组，不再使用 \n
        const parsed = globalPageRemark.value.split('|||').map(s => s.toUpperCase())
        remarksList.value = parsed.length > 0 ? parsed : ['']
      }
    }
  } catch (err) {
    console.error('GAGAL MEMUATKAN DRAF MANUAL:', err)
  }
}

const saveCustomSheetsToCloud = async () => {
  syncRemarksToGlobal()
  sessionCustomSheets.value[currentSession.value] = sessionCustomSheets.value[currentSession.value].map(s => ({
    ...s,
    day: s.day ? s.day.toUpperCase() : ''
  }))

  manualEntries.value['__custom_sheets__'] =
    sessionCustomSheets.value[currentSession.value]
  manualEntries.value['__global_remark__'] = globalPageRemark.value.toUpperCase()

  for (const k in manualEntries.value) {
    if (k !== '__custom_sheets__' && typeof manualEntries.value[k] === 'string') {
      manualEntries.value[k] = manualEntries.value[k].toUpperCase()
    }
  }

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
    console.error('GAGAL MENYIMPAN JADUAL TAMBAHAN KE AWAN:', err)
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
    .toUpperCase()

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
    console.error('GAGAL MENYIMPAN DRAF SEMENTARA:', err)
  }
}

const getManualEntry = (slotIndex, type, period) => {
  const key = `${slotIndex}-${type}-${period}`
  return (manualEntries.value[key] || '').toUpperCase()
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
    return `${leaveItem.class_name} ${leaveItem.subject}`.toUpperCase()
  }

  if (type === 'substitute_name') {
    const subItem =
      substituteAssignmentsMap.value[leaveItem.id]

    if (!subItem || !subItem.sub_teacher_id) {
      return ''
    }

    const subTeacher =
      teachersMap.value[subItem.sub_teacher_id]

    let name = subTeacher && subTeacher.name ? subTeacher.name.toUpperCase() : ''

    if (subItem.assignment_type === 'swap') {
      name += ' ✦'
    }

    return subItem.remark
      ? `${name} (${subItem.remark.toUpperCase()})`
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

  allSameSessionTeachers.value = (data || []).map(t => ({
    ...t,
    name: t.name ? t.name.toUpperCase() : '',
    subject: t.subject ? t.subject.toUpperCase() : ''
  })).sort((a, b) => 
    a.name.localeCompare(b.name, 'en', { sensitivity: 'base' })
  )
}

// ⭐️ Penyelesaian sempurna untuk pengekstrakan kelas gabung (PMPI) & pelbagai subjek (UPPERCASE)
const loadClassSchedulesForTargetDate = async () => {
  try {
    const dateObj = new Date(targetDate.value)
    const dayNum = dateObj.getDay()
    const weekdayCalc = dayNum === 0 ? 7 : dayNum // 1-7 (Isnin-Ahad)

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

        const rawClassName = String(row.class_name).toUpperCase();
        let cleanedClassName = rawClassName
          .replace(/\b(PM|PI|MORAL|AGAMA|ISLAM|PENDIDIKAN)\b/g, '')
          .replace(/\s+/g, ' ')
          .trim();
        
        cleanedClassName = cleanedClassName.replace(/^[/-]+|[/-]+$/g, '').trim();

        const classTokens = cleanedClassName.split(/[/,&+,]|\bDAN\b/).map(c => c.trim()).filter(Boolean);

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
    console.error('GAGAL MEMUATKAN DATA JADUAL KELAS:', err)
  }
}

// =================================================================
// ⭐️ Logik untuk membersihkan seluruh baris manual
const clearManualRow = async (pageIndex, slotIndex, sheetId) => {
  if (!window.confirm('ADAKAH ANDA PASTI MAHU MEMADAMKAN SEMUA KANDUNGAN DALAM BARIS INI (TERMASUK REKOD GURU GANTI)?')) {
    return
  }

  const prefix = sheetId ? `sheet_${sheetId}_${slotIndex}` : `page_${pageIndex}_${slotIndex}`
  const periods = currentPeriodTimes.value.length

  try {
    for (let p = 1; p <= periods; p++) {
      const virtualLeaveKey = `${prefix}_virtual_leave_${p}`
      const existingVirtualLeaveId = manualEntries.value[virtualLeaveKey]

      if (existingVirtualLeaveId) {
        await supabase.from('substitute_assignments').delete().eq('leave_request_id', existingVirtualLeaveId)
        await supabase.from('leave_requests').delete().eq('id', existingVirtualLeaveId)
        delete manualEntries.value[virtualLeaveKey]
      }

      manualEntries.value[`${prefix}-kelas-${p}`] = ''
      manualEntries.value[`${prefix}-ganti-${p}`] = ''
      manualEntries.value[`${prefix}-ttangan-${p}`] = ''
    }

    manualEntries.value[`${prefix}-name-0`] = ''

    await saveCustomSheetsToCloud()
    toast.success('KANDUNGAN BARIS TELAH DIKOSONGKAN!')
    
    fetchData()
  } catch (err) {
    toast.error('GAGAL DIKOSONGKAN: ' + err.message)
  }
}

// =================================================================
// Buka modal untuk mengekstrak subjek kelas baris keseluruhan
const openClassPicker = (pageIndex, slotIndex, sheetId) => {
  classPickerTarget.value = { pageIndex, slotIndex, sheetId }
  selectedClassToFill.value = ''
  showClassPickerModal.value = true
}

// Pengesahan ekstrak kelas ke baris KELAS
const confirmClassPicker = async () => {
  if (!selectedClassToFill.value) {
    toast.error('SILA PILIH SATU KELAS!')
    return
  }

  const { pageIndex, slotIndex, sheetId } = classPickerTarget.value
  const prefix = sheetId ? `sheet_${sheetId}_${slotIndex}` : `page_${pageIndex}_${slotIndex}`
  const className = selectedClassToFill.value.toUpperCase()

  manualEntries.value[`${prefix}-name-0`] = className

  for (let p = 1; p <= currentPeriodTimes.value.length; p++) {
    const subject = classSchedulesMap.value[className]?.[p] || ''
    const text = subject ? `${className} ${subject}`.toUpperCase() : '' 
    manualEntries.value[`${prefix}-kelas-${p}`] = text
  }

  await saveCustomSheetsToCloud()
  toast.success(`SEMUA SUBJEK UNTUK KELAS ${className} PADA HARI INI TELAH DIEKSTRAK!`)
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
    toast.error('TIADA REKOD CUTI UNTUK GURU INI PADA WAKTU INI!')
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
      existingSub.remark ? existingSub.remark.toUpperCase() : ''

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
            name: t.name ? t.name.toUpperCase() : '',
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

    recommendations.value = results.map(r => ({
      ...r,
      name: r.name ? r.name.toUpperCase() : ''
    }))
  } catch (err) {
    toast.error('GAGAL MEMUATKAN DATA JADUAL: ' + err.message)
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
        ? assignmentRemark.value.trim().toUpperCase()
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
        ? 'PERTUKARAN JADUAL BERJAYA DITETAPKAN!'
        : 'GURU GANTI BERJAYA DITETAPKAN!'
    )

    showModal.value = false

    fetchData()
  } catch (err) {
    toast.error('GAGAL MENETAPKAN: ' + err.message)
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

      toast.success('PENETAPAN TELAH DIBATALKAN')

      showModal.value = false

      fetchData()
    }
  } catch (err) {
    toast.error('OPERASI GAGAL: ' + err.message)
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
    toast.success('TIADA KELAS YANG PERLU DITETAPKAN UNTUK SESI INI!')
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

    toast.success(`BERJAYA! SEBANYAK ${successCount} KELAS TELAH DITETAPKAN SECARA AUTOMATIK.`)

    fetchData()
  } catch (err) {
    toast.error('RALAT SEMASA PENETAPAN AUTOMATIK: ' + err.message)
  } finally {
    isAutoAssigning.value = false
  }
}

// ================= Baris kosong & logik penetapan maya =================

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
        existingSub.remark ? existingSub.remark.toUpperCase() : ''

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
    return toast.error('SILA PILIH GURU GANTI JIKA INGIN MENGIRA BEBAN!')
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

    teacherName = t && t.name ? t.name.toUpperCase() : ''
  }
  
  let displayText = teacherName

  if (blankForm.value.remark) {
    const upperRemark = blankForm.value.remark.toUpperCase()
    displayText = teacherName
      ? `${teacherName} (${upperRemark})`
      : upperRemark
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
            'GURU INI SUDAH MEMPUNYAI REKOD CUTI SEBENAR PADA WAKTU INI, TIDAK BOLEH DITETAPKAN BERULANG KALI!'
          )
        }

        targetLeaveId =
          existingLeave.id

        await supabase
          .from('leave_requests')
          .update({
            reason:
              (blankForm.value.remark ? blankForm.value.remark.toUpperCase() : '') ||
              'TUGAS KHAS'
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
              (blankForm.value.remark ? blankForm.value.remark.toUpperCase() : '') ||
              'TUGAS KHAS',
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
              blankForm.value.remark ? blankForm.value.remark.toUpperCase() : null
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
              blankForm.value.remark ? blankForm.value.remark.toUpperCase() : null
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
      displayText.toUpperCase()

    await saveCustomSheetsToCloud()
    
    toast.success(
      blankForm.value.kiraBeban
        ? 'BERJAYA DITETAPKAN! DIREKOD DALAM STATISTIK BEBAN.'
        : 'TEKS BERJAYA DISIMPAN! TIDAK DIKIRA DALAM BEBAN.'
    )

    showBlankModal.value = false

    fetchData()

  } catch (err) {
    toast.error('GAGAL DISIMPAN: ' + err.message)
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
    
    toast.success('BERJAYA DIKOSONGKAN DAN BEBAN DIBATALKAN!')

    showBlankModal.value = false

    fetchData()

  } catch (err) {
    toast.error('GAGAL DIPADAM: ' + err.message)
  }
}

// =================================================================

watch(
  [targetDate, currentSession],
  () => {
    fetchData()
    loadClassSchedulesForTargetDate() // Memuatkan semula jadual apabila tarikh/sesi bertukar
  }
)

onMounted(async () => {
  await fetchSchoolIdentity()
  await fetchData()
  await loadClassSchedulesForTargetDate() // Memuatkan jadual waktu hari semasa pada permulaan
})

// ========================= Eksport PDF Langsung =========================

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
          `GAGAL MEMUAT TURUN FAIL FONT PDF: ${url} (HTTP ${response.status})`
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

      return String(value).trim().toUpperCase()
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
        (schoolName.value || 'SJK (C) LADANG GRISEK').toUpperCase(),
        PAGE_W / 2,
        y + 5,
        {
          align: 'center'
        }
      )

      doc.setFontSize(14)

      doc.text(
        `JADUAL GURU GANTI (${sessionText})`.toUpperCase(),
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
        'TARIKH :',
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
        'HARI :',
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
      ).toUpperCase()
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
      ).toUpperCase()
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
        'MASA',
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
          (teacher
            ? teacher.name
            : getManualValue(
                slot,
                'name',
                0,
                sheetId,
                pageIndex
              )).toUpperCase()

        const teacherReason =
          teacher
            ? (
                teacher.reason
                  ? `(${teacher.reason.toUpperCase()})`
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
          }`.toUpperCase(),
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
          'KELAS',
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
              (teacher
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
                  )).toUpperCase()

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
          'GURU GANTI',
          {
            fontSize: 6.2,
            bold: true
          }
        )

        currentPeriodTimes.value.forEach(
          (_, pIndex) => {
            const period =
              pIndex + 1

            const value =
              (teacher
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
                  )).toUpperCase()

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
          'T/TANGAN',
          {
            fontSize: 5.7,
            bold: true
          }
        )

        currentPeriodTimes.value.forEach(
          (_, pIndex) => {
            const period =
              pIndex + 1

            const value =
              (teacher
                ? ''
                : getManualValue(
                    slot,
                    'ttangan',
                    period,
                    sheetId,
                    pageIndex
                  )).toUpperCase()

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

      // ⭐️ PDF End: Render Catatan secara melintang ke sebelah kanan (UPPERCASE)
      // Tukar \t kepada jarak untuk elak ralat jsPDF
      const remarksListPdf = remarksList.value.map(s => s.replace(/\t/g, '    ').trim().toUpperCase()).filter(Boolean)

      if (remarksListPdf.length > 0) {
        y += 4
        doc.setFont('Georgia', 'bold')
        doc.setFontSize(8)
        doc.setTextColor(...BLACK)
        // Gunakan CATATAN: supaya serasi dengan PDF font
        doc.text('CATATAN:', M, y)
        
        y += 4
        const colCount = Math.min(remarksListPdf.length, 3) // Maksimum 3 kolum sebaris
        const colWidth = (CONTENT_W - (colCount - 1) * 4) / colCount
        
        let startX = M
        let maxBlockH = 0
        let currentY = y

        doc.setFont('Georgia', 'normal')
        doc.setFontSize(6.5)
        const lineHeight = 3.2 // mm
        
        remarksListPdf.forEach((rmkText, rIdx) => {
          const colIndex = rIdx % colCount
          if (colIndex === 0 && rIdx > 0) {
            currentY += maxBlockH + 3
            startX = M
            maxBlockH = 0
          }
          
          // Pisahkan baris untuk kekalkan format manual
          const rawLines = rmkText.split(/\r?\n/)
          let printLines = []
          rawLines.forEach(line => {
            const wrapped = doc.splitTextToSize(line, colWidth)
            printLines.push(...wrapped)
          })
          
          let lineY = currentY
          printLines.forEach(line => {
            doc.text(line, startX, lineY)
            lineY += lineHeight
          })
          
          const blockH = printLines.length * lineHeight
          if (blockH > maxBlockH) maxBlockH = blockH
          
          startX += colWidth + 4
        })
      }
    }

    const dateText =
      formattedDate.value

    const dayText =
      formattedDayName.value

    const sessionText =
      currentSession.value ===
      'morning'
        ? 'SESI PAGI'
        : 'SESI PETANG'

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
        ? 'SESI_PAGI'
        : 'SESI_PETANG'

    doc.save(
      `JADUAL_GURU_GANTI_${safeDate}_${sessionName}`.toUpperCase() + '.PDF'
    )

    toast.success(
      `PDF BERJAYA DIJANA, SEBANYAK ${
        pages.length +
        extraCustomSheets.value.length
      } MUKA SURAT.`
    )

  } catch (err) {
    console.error('PDF export failed:', err)
    toast.error(`GAGAL MENJANA PDF: ${err?.message || err}`.toUpperCase())
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