<template>
  <!-- 🌟 修复：使用最标准的 min-w-[1024px]，确保 Tailwind 能 100% 编译生效，卡片背景绝对能包裹所有内容 -->
  <div class="p-4 sm:p-8 mx-auto min-h-screen space-y-8 min-w-[1024px]">
    
    <!-- Header Section -->
    <div class="bg-white rounded-3xl p-6 sm:p-8 shadow-sm ring-1 ring-slate-900/5 flex flex-col md:flex-row justify-between items-start md:items-center gap-6">
      <div class="space-y-2 max-w-3xl">
        <h1 class="text-2xl sm:text-3xl font-extrabold tracking-tight text-transparent bg-clip-text bg-gradient-to-r from-slate-900 via-indigo-800 to-violet-800">
          TEACHER TIMETABLE MANAGEMENT
        </h1>
        <p class="text-slate-500 text-xs sm:text-sm font-medium leading-relaxed">
          SUPPORTS RAPID GRID EDITING AND BULK IMPORTING FOR ALL TEACHERS' MASTER TIMETABLES.
        </p>
      </div>
      
       <!-- Right Action Area: Upload & Download -->
      <div class="flex flex-col sm:flex-row items-center gap-3 shrink-0">
        <!-- Download Template Button -->
        <button @click="downloadTemplate" class="w-full sm:w-auto whitespace-nowrap inline-flex items-center justify-center px-4 h-11 text-xs font-bold text-indigo-700 bg-indigo-50 border border-indigo-100 rounded-2xl hover:bg-indigo-100 transition-all shadow-sm cursor-pointer">
          <svg class="w-4 h-4 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 16v1a3 3 0 003 3h10a3 3 0 003-3v-1m-4-4l-4 4m0 0l-4-4m4 4V4"></path></svg>
          DOWNLOAD TEMPLATE
        </button>
        
        <!-- Upload CSV Button -->
        <label class="w-full sm:w-auto whitespace-nowrap cursor-pointer inline-flex items-center justify-center px-4 h-11 text-xs font-bold text-white bg-slate-900 rounded-2xl hover:bg-slate-800 hover:shadow-md transition-all shadow-sm">
          <svg class="w-4 h-4 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 16v1a3 3 0 003 3h10a3 3 0 003-3v-1m-4-8l-4-4m0 0L8 8m4-4v12"></path></svg>
          BULK IMPORT
          <input type="file" accept=".csv" class="hidden" @change="handleCsvUpload" />
        </label>
      </div>
    </div>

    <!-- Session Switcher Tabs & Teacher Selector -->
    <div class="flex flex-col sm:flex-row justify-between items-stretch sm:items-center bg-white p-6 rounded-3xl shadow-sm ring-1 ring-slate-900/5 gap-4">
      <!-- Switcher Tabs -->
      <div class="bg-slate-100 p-1.5 rounded-2xl flex items-center shadow-inner shrink-0">
        <button 
          @click="currentSession = 'morning'; selectedTeacherId = ''; currentTimetable = []" 
          :class="currentSession === 'morning' ? 'bg-slate-900 text-white shadow-sm' : 'text-slate-600 hover:text-slate-900'"
          class="flex-1 sm:flex-none px-6 py-2.5 rounded-xl text-xs font-bold transition-all cursor-pointer"
        >
          ☀️ MORNING SESSION
        </button>
        <button 
          @click="currentSession = 'afternoon'; selectedTeacherId = ''; currentTimetable = []" 
          :class="currentSession === 'afternoon' ? 'bg-slate-900 text-white shadow-sm' : 'text-slate-600 hover:text-slate-900'"
          class="flex-1 sm:flex-none px-6 py-2.5 rounded-xl text-xs font-bold transition-all cursor-pointer"
        >
          🌙 AFTERNOON SESSION
        </button>
      </div>

      <!-- Teacher Selector -->
      <div class="relative flex items-center bg-slate-50 border border-slate-200 rounded-2xl px-4 h-11 shrink-0 w-[300px] sm:w-[320px] shadow-sm hover:border-slate-300 transition overflow-hidden">
        <div class="w-7 h-7 rounded-xl bg-white flex items-center justify-center text-base mr-3 shrink-0 shadow-xs">
          👩‍🏫
        </div>
        <select 
          v-model="selectedTeacherId" 
          @change="fetchTeacherTimetable"
          class="w-full bg-transparent border-none text-slate-800 font-bold focus:ring-0 cursor-pointer text-xs appearance-none outline-none pr-8 truncate"
        >
          <option value="" disabled>-- Please Select a Teacher --</option>
          <option v-for="teacher in filteredTeachersList" :key="teacher.id" :value="teacher.id">
            {{ teacher.name }}{{ teacher.subject ? ` (${teacher.subject})` : '' }}
          </option>
        </select>
        <div class="absolute right-3 pointer-events-none text-slate-500 bg-slate-50 pl-2">
          <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 9l-7 7-7-7"></path></svg>
        </div>
      </div>
    </div>

    <!-- Unselected Status Prompt -->
    <div v-if="!selectedTeacherId" class="bg-white rounded-3xl shadow-sm ring-1 ring-slate-900/5 p-16 text-center flex flex-col items-center justify-center">
      <div class="w-16 h-16 bg-slate-50 rounded-2xl flex items-center justify-center text-3xl mb-4 ring-1 ring-slate-100 shadow-inner">👆</div>
      <h3 class="text-base font-bold text-slate-900">PLEASE SELECT A TEACHER ABOVE</h3>
      <p class="text-slate-500 mt-1 text-xs font-medium">ONCE SELECTED, VIEW AND EDIT THE TEACHER'S TIMETABLE HERE</p>
    </div>

    <!-- Visual 2D Grid (Displayed after a teacher is selected) -->
    <div v-else class="bg-white rounded-3xl shadow-sm ring-1 ring-slate-900/5 overflow-hidden animate-in fade-in slide-in-from-bottom-4 duration-300">
      <div class="p-6">
        <table class="w-full text-center border-collapse">
          <thead>
            <tr>
              <th class="p-4 text-slate-400 font-bold text-xs w-32 border-b-2 border-slate-100 uppercase">SLOT \ DAY</th>
              <th v-for="day in config.daysPerWeek" :key="day" class="p-4 text-slate-800 font-extrabold text-xs w-40 border-b-2 border-slate-100">
                {{ dayNames[day - 1] }}
              </th>
            </tr>
          </thead>
          <tbody>
            <template v-for="(row, index) in tableRows" :key="index">
              <!-- Regular Class Period Row -->
              <tr v-if="row.type === 'period'" class="group">
                <td class="p-3 text-slate-600 font-bold border-r-2 border-slate-50 bg-slate-50/50 group-hover:bg-slate-50 transition-colors">
                  <div class="text-xs text-indigo-900 font-extrabold">PERIOD {{ row.period }}</div>
                  <div class="text-[10px] text-slate-400 font-semibold mt-0.5">{{ row.time }}</div>
                </td>
                
                <td v-for="day in config.daysPerWeek" :key="day" class="p-2 border border-slate-50">
                  <div 
                    @click="openEditModal(day, row.period)"
                    class="h-20 w-full rounded-2xl flex flex-col items-center justify-center cursor-pointer transition-all duration-200 border-2 relative"
                    :class="getCell(day, row.period) 
                      ? 'bg-indigo-50/50 border-indigo-100 hover:border-indigo-300 hover:shadow-sm hover:-translate-y-0.5' 
                      : 'bg-transparent border-dashed border-slate-200 hover:border-indigo-300 hover:bg-slate-50'"
                  >
                    <template v-if="getCell(day, row.period)">
                      <!-- Combined Class Badge -->
                      <span v-if="getCell(day, row.period).class_name.includes('/')" class="absolute top-1.5 right-1.5 px-1.5 py-0.5 bg-violet-100 text-violet-700 rounded text-[9px] font-bold">
                        COMBINED CLASS
                      </span>
                      <span class="font-bold text-slate-900 text-sm">{{ getCell(day, row.period).class_name }}</span>
                      <span class="text-xs text-indigo-600 font-semibold mt-1">{{ getCell(day, row.period).subject }}</span>
                    </template>
                    <template v-else>
                      <span class="text-slate-400 text-xs font-semibold opacity-0 group-hover:opacity-100 transition-opacity">
                        + CLICK TO ADD
                      </span>
                    </template>
                  </div>
                </td>
              </tr>

              <!-- Break Time Insertion Row (Rehat) -->
              <tr v-else-if="row.type === 'break'" class="bg-amber-50/75">
                <td class="p-3 text-amber-800 font-bold border-r-2 border-slate-100 bg-amber-100/60 text-xs">
                  ☕ RECESS
                </td>
                <td :colspan="config.daysPerWeek" class="p-3 text-center text-amber-900 font-bold text-xs tracking-wider uppercase">
                  {{ row.title }} <span class="ml-2 font-semibold text-amber-700">({{ row.time }})</span>
                </td>
              </tr>
            </template>
          </tbody>
        </table>
      </div>
    </div>

    <!-- Advanced Edit Modal -->
    <transition enter-active-class="transition duration-200 ease-out" enter-from-class="opacity-0 scale-95" enter-to-class="opacity-100 scale-100" leave-active-class="transition duration-150 ease-in" leave-from-class="opacity-100 scale-100" leave-to-class="opacity-0 scale-95">
      <div v-if="showModal" class="fixed inset-0 z-50 flex items-center justify-center p-4 bg-slate-900/40 backdrop-blur-sm">
        <div class="absolute inset-0" @click="closeModal"></div>
        <div class="relative bg-white rounded-3xl shadow-2xl w-full max-w-sm overflow-hidden ring-1 ring-slate-900/10 p-6 space-y-4">
          <div class="flex justify-between items-center mb-2">
            <div>
              <h2 class="text-base font-bold text-slate-900">EDIT TIMETABLE SLOT</h2>
              <p class="text-[11px] text-slate-500 mt-0.5 font-medium">{{ dayNames[editingData.weekday - 1] }} · PERIOD {{ editingData.period }} ({{ getPeriodTime(editingData.period) }})</p>
            </div>
            <button @click="closeModal" class="w-8 h-8 rounded-full bg-slate-100 flex items-center justify-center text-slate-400 hover:text-slate-600 transition cursor-pointer">×</button>
          </div>

          <div class="space-y-3">
            <div>
              <label class="block text-xs font-bold text-slate-500 mb-1.5 uppercase tracking-wider">CLASS (SLASH ALLOWED: 3A/3B)</label>
              <input v-model="editingData.class_name" type="text" placeholder="E.G. 3C OR 3A/3B" class="w-full px-4 py-2.5 bg-slate-50 border border-slate-200 rounded-xl text-xs font-semibold text-slate-800 focus:outline-none focus:ring-2 focus:ring-indigo-500/50 transition-all">
            </div>
            <div>
              <label class="block text-xs font-bold text-slate-500 mb-1.5 uppercase tracking-wider">SUBJECT</label>
              <input v-model="editingData.subject" type="text" placeholder="E.G. ENGLISH LANGUAGE" class="w-full px-4 py-2.5 bg-slate-50 border border-slate-200 rounded-xl text-xs font-semibold text-slate-800 focus:outline-none focus:ring-2 focus:ring-indigo-500/50 transition-all">
            </div>
          </div>

          <div class="mt-6 flex gap-3 pt-2">
            <button v-if="editingData.id" @click="deleteClass" class="flex-1 bg-red-50 hover:bg-red-100 text-red-600 font-bold py-2.5 rounded-xl text-xs transition-colors cursor-pointer">
              CLEAR
            </button>
            <button @click="saveClass" class="flex-[2] bg-slate-900 hover:bg-slate-800 text-white font-bold py-2.5 rounded-xl text-xs shadow-md transition-all hover:shadow-lg cursor-pointer">
              SAVE
            </button>
          </div>
        </div>
      </div>
    </transition>

    <!-- 📊 Dynamic Percentage Progress Bar Modal -->
    <Transition
      enter-active-class="transition duration-300 ease-out"
      enter-from-class="opacity-0 scale-95"
      enter-to-class="opacity-100 scale-100"
      leave-active-class="transition duration-200 ease-in"
      leave-from-class="opacity-100 scale-100"
      leave-to-class="opacity-0 scale-95"
    >
      <div v-if="uploadProgress.show" class="fixed inset-0 z-50 flex items-center justify-center bg-slate-900/60 backdrop-blur-sm p-4 select-none">
        <div class="bg-white rounded-3xl max-w-md w-full p-8 shadow-2xl border border-slate-100 text-center space-y-6">
          
          <div class="w-16 h-16 rounded-2xl mx-auto flex items-center justify-center text-3xl transition-all duration-300"
               :class="uploadProgress.percent === 100 ? 'bg-emerald-100 text-emerald-600' : 'bg-indigo-50 text-indigo-600 animate-bounce'">
            <span v-if="uploadProgress.percent < 100">📅</span>
            <span v-else>🎉</span>
          </div>

          <div>
            <h3 class="text-lg font-extrabold text-slate-900">
              {{ uploadProgress.percent === 100 ? 'SCHOOL-WIDE TIMETABLE IMPORTED SUCCESSFULLY!' : 'IMPORTING TIMETABLES IN BULK...' }}
            </h3>
            <p class="text-xs font-semibold text-slate-500 mt-1.5">
              {{ uploadProgress.statusText }}
            </p>
          </div>

          <div class="space-y-2">
            <div class="w-full h-3.5 bg-slate-100 rounded-full overflow-hidden p-0.5 border border-slate-200/80 shadow-inner">
              <div 
                class="h-full bg-gradient-to-r from-indigo-500 via-purple-500 to-blue-600 rounded-full transition-all duration-300 shadow-sm"
                :style="{ width: uploadProgress.percent + '%' }"
              ></div>
            </div>
            
            <div class="flex justify-between items-center text-xs font-bold px-1">
              <span class="text-slate-400">PROCESSING PROGRESS</span>
              <span class="text-indigo-600 font-black text-sm">{{ uploadProgress.percent }}%</span>
            </div>
          </div>

        </div>
      </div>
    </Transition>

  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import { supabase } from '../services/supabase'
import { useToast } from '../utils/toast'
import { parseCSV } from '../utils/importCsv'

const toast = useToast()
const dayNames = ['MONDAY', 'TUESDAY', 'WEDNESDAY', 'THURSDAY', 'FRIDAY', 'SATURDAY', 'SUNDAY']
const config = ref({ daysPerWeek: 5, periodsPerDay: 11 })
const teachersList = ref([])
const currentSession = ref('morning')
const selectedTeacherId = ref('')
const currentTimetable = ref([])
const showModal = ref(false)
const editingData = ref({ id: null, weekday: 1, period: 1, class_name: '', subject: '' })

const uploadProgress = ref({
  show: false,
  percent: 0,
  statusText: ''
})

const sleep = (ms) => new Promise(resolve => setTimeout(resolve, ms))

const startProgress = (initialText = 'READING TIMETABLE CSV FILE...') => {
  uploadProgress.value = {
    show: true,
    percent: 10,
    statusText: initialText
  }
}

const updateProgress = (percent, text) => {
  uploadProgress.value.percent = percent
  if (text) uploadProgress.value.statusText = text
}

const finishProgress = async (successMsg = 'IMPORT SUCCESSFUL') => {
  uploadProgress.value.percent = 100
  uploadProgress.value.statusText = successMsg
  await sleep(600)
  uploadProgress.value.show = false
}

const tableRows = computed(() => {
  if (currentSession.value === 'morning') {
    return [
      { type: 'period', period: 1, time: '7.00-7.30' },
      { type: 'period', period: 2, time: '7.30-8.00' },
      { type: 'period', period: 3, time: '8.00-8.30' },
      { type: 'period', period: 4, time: '8.30-9.00' },
      { type: 'period', period: 5, time: '9.00-9.30' },
      { type: 'break', title: 'RECESS / BREAK TIME', time: '9.30 - 10.00' },
      { type: 'period', period: 6, time: '10.00-10.30' },
      { type: 'period', period: 7, time: '10.30-11.00' },
      { type: 'period', period: 8, time: '11.00-11.30' },
      { type: 'period', period: 9, time: '11.30-12.00' },
      { type: 'period', period: 10, time: '12.00-12.30' },
      { type: 'period', period: 11, time: '12.30-1.00' }
    ]
  } else {
    return [
      { type: 'period', period: 1, time: '1.00-1.30' },
      { type: 'period', period: 2, time: '1.30-2.00' },
      { type: 'period', period: 3, time: '2.00-2.30' },
      { type: 'period', period: 4, time: '2.30-3.00' },
      { type: 'period', period: 5, time: '3.00-3.30' },
      { type: 'break', title: 'RECESS / BREAK TIME', time: '3.30 - 3.50' },
      { type: 'period', period: 6, time: '3.50-4.20' },
      { type: 'period', period: 7, time: '4.20-4.50' },
      { type: 'period', period: 8, time: '4.50-5.20' },
      { type: 'period', period: 9, time: '5.20-5.50' },
      { type: 'period', period: 10, time: '5.50-6.20' }
    ]
  }
})

const getPeriodTime = (periodNum) => {
  const found = tableRows.value.find(r => r.type === 'period' && r.period === periodNum)
  return found ? found.time : ''
}

const filteredTeachersList = computed(() => {
  const list = teachersList.value.filter(t => (t.session || 'morning') === currentSession.value)
  return [...list].sort((a, b) => {
    let valA = a.name ? a.name.trim().toUpperCase() : ''
    let valB = b.name ? b.name.trim().toUpperCase() : ''
    return valA.localeCompare(valB, 'en', { numeric: true })
  })
})

onMounted(async () => {
  const savedConfig = localStorage.getItem('school_config')
  if (savedConfig) config.value = JSON.parse(savedConfig)

  const { data } = await supabase.from('teachers').select('id, name, subject, session')
  if (data) teachersList.value = data
})

const fetchTeacherTimetable = async () => {
  if (!selectedTeacherId.value) return
  const { data } = await supabase
    .from('timetable')
    .select('*')
    .eq('teacher_id', selectedTeacherId.value)
  if (data) currentTimetable.value = data
}

const getCell = (weekday, period) => {
  return currentTimetable.value.find(c => c.weekday === weekday && c.period === period)
}

const openEditModal = (weekday, period) => {
  const existingClass = getCell(weekday, period)
  if (existingClass) {
    editingData.value = { ...existingClass }
  } else {
    editingData.value = { id: null, weekday, period, class_name: '', subject: '' }
  }
  showModal.value = true
}

const closeModal = () => showModal.value = false

const saveClass = async () => {
  if (!editingData.value.class_name || !editingData.value.subject) return toast.error('CLASS AND SUBJECT CANNOT BE EMPTY')
  const payload = {
    teacher_id: selectedTeacherId.value,
    weekday: editingData.value.weekday,
    period: editingData.value.period,
    class_name: editingData.value.class_name.trim().toUpperCase(),
    subject: editingData.value.subject.trim().toUpperCase()
  }
  const res = editingData.value.id 
    ? await supabase.from('timetable').update(payload).eq('id', editingData.value.id)
    : await supabase.from('timetable').insert(payload)
  
  if (res.error) return toast.error('FAILED TO SAVE: ' + res.error.message)
  toast.success('TIMETABLE UPDATED SUCCESSFULLY')
  fetchTeacherTimetable()
  closeModal()
}

const deleteClass = async () => {
  const { error } = await supabase.from('timetable').delete().eq('id', editingData.value.id)
  if (error) return toast.error('FAILED TO DELETE: ' + error.message)
  toast.success('TIMETABLE SLOT CLEARED')
  fetchTeacherTimetable()
  closeModal()
}

const downloadTemplate = () => {
  const csvContent = "\uFEFFteacher_name,weekday,period,class_name,subject\n(EXAMPLE) AHMAD,1,3,3A/3B,MORAL\n(EXAMPLE) LEE,2,4,4A,ENGLISH";
  const blob = new Blob([csvContent], { type: 'text/csv;charset=utf-8;' });
  const link = document.createElement("a");
  link.href = URL.createObjectURL(blob);
  link.download = "TIMETABLE_IMPORT_TEMPLATE.csv";
  link.style.visibility = 'hidden';
  document.body.appendChild(link);
  link.click();
  document.body.removeChild(link);
  toast.success("TEMPLATE DOWNLOADED SUCCESSFULLY!");
}

const handleCsvUpload = async (e) => {
  const file = e.target.files[0]
  if (!file) return
  
  startProgress('READING TIMETABLE CSV FILE...')

  try {
    await sleep(200)
    updateProgress(25, 'VALIDATING TIMETABLE DATA...')

    const data = await parseCSV(file)
    const filteredData = data.filter(row => row.teacher_name && !row.teacher_name.includes('EXAMPLE'))
    
    if (filteredData.length === 0) {
      uploadProgress.value.show = false
      throw new Error("NO VALID DATA FOUND, PLEASE CHECK FORMAT.")
    }

    let successCount = 0
    const totalRows = filteredData.length

    await sleep(200)
    updateProgress(40, `READ SUCCESSFUL! READY TO IMPORT ${totalRows} TIMETABLE RECORDS...`)

    for (let index = 0; index < totalRows; index++) {
      const row = filteredData[index]
      const inputName = row.teacher_name.trim()
      const matchedTeacher = teachersList.value.find(t => t.name.trim() === inputName)
      
      if (!matchedTeacher) {
        uploadProgress.value.show = false
        throw new Error(`ERROR AT ROW ${index + 2}: TEACHER "${inputName}" NOT FOUND IN SYSTEM, PLEASE ADD THEM IN TEACHER PROFILES!`)
      }

      const weekdayNum = parseInt(row.weekday)
      const periodNum = parseInt(row.period)

      await supabase
        .from('timetable')
        .delete()
        .eq('teacher_id', matchedTeacher.id)
        .eq('weekday', weekdayNum)
        .eq('period', periodNum)

      const { error: insertErr } = await supabase.from('timetable').insert({
        teacher_id: matchedTeacher.id,
        weekday: weekdayNum,
        period: periodNum,
        class_name: row.class_name.trim().toUpperCase(),
        subject: row.subject.trim().toUpperCase()
      })

      if (insertErr) throw insertErr
      successCount++

      const currentPercent = 40 + Math.floor(((index + 1) / totalRows) * 50)
      updateProgress(currentPercent, `WRITING TIMETABLE TO DATABASE (${index + 1}/${totalRows})...`)
    }

    await finishProgress(`SUCCESSFULLY IMPORTED AND UPDATED ${successCount} TIMETABLE RECORDS!`)
    toast.success(`SUCCESSFULLY IMPORTED AND UPDATED ${successCount} TIMETABLE RECORDS!`)
    
    if (selectedTeacherId.value) {
      fetchTeacherTimetable()
    }
    
  } catch (err) {
    uploadProgress.value.show = false
    toast.error("IMPORT FAILED: " + err.message)
  } finally {
    e.target.value = ''
  }
}
</script>