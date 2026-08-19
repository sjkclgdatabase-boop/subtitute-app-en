<template>
  <!-- 🌟 终极修复：min-w-max w-full 保证外框永远跟最宽的表格一样宽，绝不裁切 -->
  <div class="p-4 sm:p-8 min-h-screen space-y-8 min-w-max w-full">
    
    <!-- Header Section -->
    <div class="bg-white rounded-3xl p-6 sm:p-8 shadow-sm ring-1 ring-slate-900/5 flex justify-between items-center gap-6">
      <div class="space-y-2 max-w-3xl">
        <h1 class="text-xl sm:text-2xl md:text-3xl font-extrabold tracking-tight text-transparent bg-clip-text bg-gradient-to-r from-slate-900 via-indigo-800 to-violet-800 flex items-center gap-3">
          <UsersRound class="w-8 h-8 text-indigo-700 shrink-0" />
          TEACHER PROFILE MANAGEMENT
        </h1>
        <p class="text-slate-500 text-xs sm:text-sm font-medium leading-relaxed">
          MANAGE TEACHER INFORMATION, SESSION ASSIGNMENTS, MAXIMUM SUBSTITUTE LIMITS, AND BULK CSV IMPORT.
        </p>
      </div>

      <button @click="showModal = true" class="bg-slate-900 hover:bg-slate-800 text-white px-6 h-11 rounded-2xl text-xs font-bold shadow-sm transition cursor-pointer shrink-0 flex items-center justify-center gap-2">
        <Plus class="w-4 h-4" />
        ADD TEACHER
      </button>
    </div>
    
    <!-- Top Action Bar -->
    <div class="bg-white p-6 rounded-3xl shadow-sm ring-1 ring-slate-900/5 flex justify-between items-center gap-4">
      <div class="flex items-center gap-4">
        <!-- Morning / Afternoon switcher -->
        <div class="bg-slate-100 p-1.5 rounded-2xl flex items-center shadow-inner shrink-0">
          <button 
            @click="currentSession = 'morning'" 
            class="py-2.5 px-6 rounded-xl text-xs font-bold transition-all flex items-center justify-center gap-2 cursor-pointer"
            :class="currentSession === 'morning' 
              ? 'bg-slate-900 text-white shadow-sm' 
              : 'text-slate-500 hover:text-slate-900'"
          >
            <Sun class="w-4 h-4 text-amber-500" /> MORNING SESSION
          </button>
          <button 
            @click="currentSession = 'afternoon'" 
            class="py-2.5 px-6 rounded-xl text-xs font-bold transition-all flex items-center justify-center gap-2 cursor-pointer"
            :class="currentSession === 'afternoon' 
              ? 'bg-slate-900 text-white shadow-sm' 
              : 'text-slate-500 hover:text-slate-900'"
          >
            <Moon class="w-4 h-4 text-indigo-400" /> AFTERNOON SESSION
          </button>
        </div>

        <!-- Bulk Import & Template Button Group -->
        <div class="flex items-center gap-3">
          <button @click="downloadTemplate" class="bg-emerald-600 text-white px-4 h-11 rounded-2xl hover:bg-emerald-700 text-xs font-bold shadow-sm transition cursor-pointer shrink-0 flex items-center gap-2">
            <Download class="w-4 h-4" />
            DOWNLOAD IMPORT TEMPLATE
          </button>
          <label class="bg-blue-600 text-white px-4 h-11 rounded-2xl hover:bg-blue-700 cursor-pointer text-xs font-bold shadow-sm transition flex items-center justify-center shrink-0 gap-2">
            <Upload class="w-4 h-4" />
            BULK IMPORT
            <input type="file" accept=".csv" @change="handleCsvUpload" class="hidden" />
          </label>
        </div>
      </div>
    </div>

    <!-- Teacher List Table Area -->
    <!-- 🌟 保留 overflow-hidden 让圆角生效，现在卡片宽度充足，再也不会切掉内容了 -->
    <div class="bg-white rounded-3xl shadow-sm ring-1 ring-slate-900/5 overflow-hidden">
      <table class="w-full text-left border-collapse whitespace-nowrap">
        <thead>
          <tr class="bg-slate-50/50 border-b border-slate-100 text-slate-500 text-xs uppercase tracking-widest font-semibold select-none">
            <th @click="sortBy('name')" class="py-4 pl-8 pr-4 cursor-pointer hover:text-indigo-600 transition">
              <div class="flex items-center gap-1 font-bold">
                TEACHER NAME <span class="text-xs text-slate-400">{{ getSortIcon('name') }}</span>
              </div>
            </th>
            <th @click="sortBy('subject')" class="p-4 cursor-pointer hover:text-indigo-600 transition">
              <div class="flex items-center gap-1 font-bold">
                SUBJECT <span class="text-xs text-slate-400">{{ getSortIcon('subject') }}</span>
              </div>
            </th>
            <th @click="sortBy('session')" class="p-4 cursor-pointer hover:text-indigo-600 transition">
              <div class="flex items-center gap-1 font-bold">
                SESSION <span class="text-xs text-slate-400">{{ getSortIcon('session') }}</span>
              </div>
            </th>
            <th @click="sortBy('max_substitute_per_week')" class="p-4 cursor-pointer hover:text-indigo-600 transition">
              <div class="flex items-center gap-1 font-bold">
                MAX WEEKLY SUBSTITUTES <span class="text-xs text-slate-400">{{ getSortIcon('max_substitute_per_week') }}</span>
              </div>
            </th>
            <th @click="sortBy('is_active')" class="p-4 cursor-pointer hover:text-indigo-600 transition">
              <div class="flex items-center gap-1 font-bold">
                STATUS <span class="text-xs text-slate-400">{{ getSortIcon('is_active') }}</span>
              </div>
            </th>
            <th class="py-4 pr-8 pl-4 font-bold text-right">ACTIONS</th>
          </tr>
        </thead>
        <tbody class="divide-y divide-slate-100 text-xs font-medium text-slate-800">
          <tr v-if="filteredTeachers.length === 0">
            <td colspan="6" class="p-12 text-center text-slate-400 text-xs font-medium">
              NO TEACHER DATA FOUND FOR THIS SESSION
            </td>
          </tr>
          <tr v-for="t in filteredTeachers" :key="t.id" class="hover:bg-slate-50/80 transition">
            <td class="py-4 pl-8 pr-4 font-bold text-slate-900">
              <div class="flex items-center gap-2">
                <span>{{ t.name }}</span>
                <span v-if="t.is_admin" class="px-2 py-0.5 bg-rose-100 text-rose-700 rounded-md text-[10px] font-bold tracking-widest shrink-0">
                  ADMIN
                </span>
                <span v-if="t.is_counselor" class="px-2 py-0.5 bg-emerald-100 text-emerald-700 rounded-md text-[10px] font-bold tracking-widest shrink-0">
                  COUNSELOR
                </span>
              </div>
            </td>
            <td class="p-4 font-semibold text-slate-700">{{ t.subject || '-' }}</td>
            <td class="p-4">
              <span :class="t.session === 'afternoon' ? 'bg-orange-100 text-orange-800' : 'bg-blue-100 text-blue-800'" class="px-2.5 py-1 rounded-full text-xs font-bold whitespace-nowrap inline-block">
                {{ t.session === 'afternoon' ? 'AFTERNOON SESSION' : 'MORNING SESSION' }}
              </span>
            </td>
            <td class="p-4 font-bold text-slate-700 whitespace-nowrap">{{ t.max_substitute_per_week }} SLOTS</td>
            <td class="p-4">
              <span :class="t.is_active ? 'bg-emerald-50 text-emerald-700' : 'bg-red-50 text-red-700'" class="px-2.5 py-1 rounded-lg text-xs font-bold whitespace-nowrap inline-block">
                {{ t.is_active ? 'ACTIVE' : 'INACTIVE' }}
              </span>
            </td>
            <td class="py-4 pr-8 pl-4 text-right">
              <button @click="deleteTeacher(t.id)" class="text-xs text-red-600 hover:text-red-800 font-bold px-3 py-1.5 bg-red-50 hover:bg-red-100 rounded-lg cursor-pointer transition whitespace-nowrap">
                DELETE
              </button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <!-- Add Teacher Modal -->
    <div v-if="showModal" class="fixed inset-0 bg-slate-900/40 backdrop-blur-sm flex items-center justify-center z-50 p-4">
      <div class="bg-white p-6 rounded-3xl w-full max-w-sm shadow-2xl ring-1 ring-slate-900/10 max-h-[90vh] overflow-y-auto space-y-4">
        <h2 class="text-base font-bold text-slate-900 mb-2">ADD NEW TEACHER</h2>
        
        <div class="space-y-3">
          <div>
            <label class="block text-xs font-bold text-slate-500 mb-1">NAME</label>
            <input v-model="form.name" placeholder="E.G. AHMAD" class="w-full px-4 py-2.5 bg-slate-50 border border-slate-200 rounded-xl text-xs font-semibold text-slate-800 focus:outline-none focus:ring-2 focus:ring-indigo-500/50" />
          </div>
          <div>
            <label class="block text-xs font-bold text-slate-500 mb-1">SUBJECT</label>
            <input v-model="form.subject" placeholder="E.G. CHINESE LANGUAGE" class="w-full px-4 py-2.5 bg-slate-50 border border-slate-200 rounded-xl text-xs font-semibold text-slate-800 focus:outline-none focus:ring-2 focus:ring-indigo-500/50" />
          </div>
          <div>
            <label class="block text-xs font-bold text-slate-500 mb-1">SESSION ASSIGNMENT</label>
            <select v-model="form.session" class="w-full px-4 py-2.5 bg-slate-50 border border-slate-200 rounded-xl text-xs font-semibold text-slate-800 focus:outline-none focus:ring-2 focus:ring-indigo-500/50 cursor-pointer">
              <option value="morning">MORNING SESSION</option>
              <option value="afternoon">AFTERNOON SESSION</option>
            </select>
          </div>
          <div>
            <label class="block text-xs font-bold text-slate-500 mb-1">MAX SUBSTITUTE LIMIT (SLOTS)</label>
            <input v-model.number="form.max_substitute_per_week" type="number" class="w-full px-4 py-2.5 bg-slate-50 border border-slate-200 rounded-xl text-xs font-semibold text-slate-800 focus:outline-none focus:ring-2 focus:ring-indigo-500/50" />
          </div>

          <!-- Admin Toggle -->
          <div class="flex items-center justify-between p-3 bg-slate-50 border border-slate-200 rounded-xl mt-2 gap-3">
            <div>
              <label class="block text-xs font-bold text-slate-700">SET AS ADMINISTRATOR</label>
              <p class="text-[10px] text-slate-500 mt-0.5">LOWER SCHEDULING PRIORITY</p>
            </div>
            <label class="relative inline-flex items-center cursor-pointer shrink-0">
              <input type="checkbox" v-model="form.is_admin" class="sr-only peer">
              <div class="w-9 h-5 bg-slate-300 rounded-full peer peer-checked:after:translate-x-full peer-checked:after:border-white after:content-[''] after:absolute after:top-[2px] after:left-[2px] after:bg-white after:border-gray-300 after:border after:rounded-full after:h-4 after:w-4 after:transition-all peer-checked:bg-rose-500"></div>
            </label>
          </div>

          <!-- Counselor Toggle -->
          <div class="flex items-center justify-between p-3 bg-slate-50 border border-slate-200 rounded-xl mt-2 gap-3">
            <div>
              <label class="block text-xs font-bold text-slate-700">SET AS COUNSELOR</label>
              <p class="text-[10px] text-slate-500 mt-0.5">EXPAND DAILY LIMIT TO 4 SLOTS</p>
            </div>
            <label class="relative inline-flex items-center cursor-pointer shrink-0">
              <input type="checkbox" v-model="form.is_counselor" class="sr-only peer">
              <div class="w-9 h-5 bg-slate-300 rounded-full peer peer-checked:after:translate-x-full peer-checked:after:border-white after:content-[''] after:absolute after:top-[2px] after:left-[2px] after:bg-white after:border-gray-300 after:border after:rounded-full after:h-4 after:w-4 after:transition-all peer-checked:bg-emerald-500"></div>
            </label>
          </div>
        </div>

        <div class="mt-6 flex gap-3">
          <button @click="showModal = false" class="flex-1 bg-slate-100 hover:bg-slate-200 text-slate-700 py-2.5 rounded-xl text-xs font-bold transition cursor-pointer">CANCEL</button>
          <button @click="saveTeacher" class="flex-1 bg-slate-900 hover:bg-slate-800 text-white py-2.5 rounded-xl text-xs font-bold shadow-md transition cursor-pointer">SAVE</button>
        </div>
      </div>
    </div>

    <!-- Dynamic Percentage Progress Bar Modal -->
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
          <div class="w-16 h-16 rounded-2xl mx-auto flex items-center justify-center transition-all duration-300"
               :class="uploadProgress.percent === 100 ? 'bg-emerald-100 text-emerald-600' : 'bg-indigo-50 text-indigo-600 animate-bounce'">
            <GraduationCap v-if="uploadProgress.percent < 100" class="w-8 h-8" />
            <Sparkles v-else class="w-8 h-8" />
          </div>

          <div>
            <h3 class="text-lg font-extrabold text-slate-900">
              {{ uploadProgress.percent === 100 ? 'TEACHER DATA IMPORTED SUCCESSFULLY!' : 'IMPORTING TEACHERS IN BULK...' }}
            </h3>
            <p class="text-xs font-semibold text-slate-500 mt-1.5">
              {{ uploadProgress.statusText }}
            </p>
          </div>

          <div class="space-y-2">
            <div class="w-full h-3.5 bg-slate-100 rounded-full overflow-hidden p-0.5 border border-slate-200/80 shadow-inner">
              <div 
                class="h-full bg-gradient-to-r from-blue-500 via-indigo-500 to-purple-600 rounded-full transition-all duration-300 shadow-sm"
                :style="{ width: uploadProgress.percent + '%' }"
              ></div>
            </div>
            
            <div class="flex justify-between items-center text-xs font-bold px-1">
              <span class="text-slate-400">PROCESSING PROGRESS</span>
              <span class="text-indigo-600 font-black text-sm shrink-0">{{ uploadProgress.percent }}%</span>
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
import { parseCSV } from '../utils/importCsv'
import { useToast } from '../utils/toast'
import { 
  UsersRound, 
  Plus, 
  Sun, 
  Moon, 
  Download, 
  Upload, 
  GraduationCap, 
  Sparkles 
} from 'lucide-vue-next'

const toast = useToast()
const teachers = ref([])
const showModal = ref(false)
const currentSession = ref('morning')

const form = ref({ name: '', subject: '', max_substitute_per_week: 5, session: 'morning', is_active: true, is_admin: false, is_counselor: false })

const uploadProgress = ref({
  show: false,
  percent: 0,
  statusText: ''
})

const sleep = (ms) => new Promise(resolve => setTimeout(resolve, ms))

const startProgress = (initialText = 'READING TEACHER CSV FILE...') => {
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

// Sorting state management
const sortKey = ref('name') 
const sortOrder = ref('asc') 

const sortBy = (key) => {
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

const filteredTeachers = computed(() => {
  const list = teachers.value.filter(t => (t.session || 'morning') === currentSession.value)
  
  return [...list].sort((a, b) => {
    let valA = a[sortKey.value] ? String(a[sortKey.value]).trim().toUpperCase() : ''
    let valB = b[sortKey.value] ? String(b[sortKey.value]).trim().toUpperCase() : ''

    const res = valA.localeCompare(valB, 'en', { numeric: true })
    return sortOrder.value === 'asc' ? res : -res
  })
})

const fetchTeachers = async () => {
  const { data } = await supabase.from('teachers').select('*')
  if (data) teachers.value = data
}

const downloadTemplate = () => {
  const csvContent = "name,subject,max_substitute_per_week,session,is_admin,is_counselor\nAHMAD,CHINESE LANGUAGE,5,morning,false,false\nLEE(ADMIN),MATHEMATICS,8,afternoon,true,false\nWONG(COUNSELOR),GUIDANCE,10,morning,false,true"
  const blob = new Blob(['\ufeff', csvContent], { type: 'text/csv;charset=utf-8;' })
  const url = URL.createObjectURL(blob)
  
  const link = document.createElement("a")
  link.setAttribute("href", url)
  link.setAttribute("download", "TEACHER_IMPORT_TEMPLATE.csv")
  document.body.appendChild(link)
  link.click()
  document.body.removeChild(link)
  toast.success("TEMPLATE DOWNLOADED SUCCESSFULLY!")
}

const saveTeacher = async () => {
  const { error } = await supabase.from('teachers').insert([form.value])
  if (error) toast.error("FAILED TO SAVE: " + error.message)
  else {
    toast.success("ADDED SUCCESSFULLY")
    showModal.value = false
    form.value = { name: '', subject: '', max_substitute_per_week: 5, session: 'morning', is_active: true, is_admin: false, is_counselor: false }
    fetchTeachers()
  }
}

const deleteTeacher = async (id) => {
  const { error } = await supabase.from('teachers').delete().eq('id', id)
  if (!error) { toast.success("DELETED SUCCESSFULLY"); fetchTeachers(); }
}

const handleCsvUpload = async (e) => {
  const file = e.target.files[0]
  if (!file) return
  
  startProgress('READING TEACHER CSV FILE...')

  try {
    await sleep(200)
    updateProgress(30, 'PARSING AND VALIDATING TEACHER DATA...')

    const data = await parseCSV(file)
    if (!data || data.length === 0) {
      uploadProgress.value.show = false
      toast.error("CSV FILE IS EMPTY OR FORMAT IS INVALID")
      return
    }

    let updateCount = 0
    let insertCount = 0
    const totalRows = data.length

    await sleep(200)
    updateProgress(50, `READY TO PROCESS ${totalRows} TEACHER RECORDS...`)

    for (let index = 0; index < totalRows; index++) {
      const row = data[index]
      if (!row.name) continue
      const trimmedName = String(row.name).trim()

      let rawSession = row.session ? String(row.session).trim().toLowerCase() : 'morning'
      if (rawSession.includes('afternoon') || rawSession.includes('下') || rawSession.includes('petang')) {
        rawSession = 'afternoon'
      } else {
        rawSession = 'morning'
      }

      const teacherPayload = {
        name: trimmedName,
        subject: row.subject ? String(row.subject).trim() : '',
        max_substitute_per_week: parseInt(row.max_substitute_per_week || 5),
        session: rawSession,
        is_active: true,
        is_admin: row.is_admin ? (String(row.is_admin).trim().toLowerCase() === 'true' || String(row.is_admin).trim() === '1' || row.is_admin === '是') : false,
        is_counselor: row.is_counselor ? (String(row.is_counselor).trim().toLowerCase() === 'true' || String(row.is_counselor).trim() === '1' || row.is_counselor === '是') : false
      }

      const existingTeacher = teachers.value.find(t => t.name.trim() === trimmedName)

      if (existingTeacher) {
        const { error } = await supabase
          .from('teachers')
          .update(teacherPayload)
          .eq('id', existingTeacher.id)
        
        if (error) throw error
        updateCount++
      } else {
        const { error } = await supabase
          .from('teachers')
          .insert([teacherPayload])
        
        if (error) throw error
        insertCount++
      }

      const currentPercent = 50 + Math.floor(((index + 1) / totalRows) * 40)
      updateProgress(currentPercent, `WRITING TO DATABASE (${index + 1}/${totalRows})...`)
    }

    await finishProgress(`COMPLETED: ADDED ${insertCount}, UPDATED ${updateCount} TEACHERS`)
    toast.success(`IMPORT SUCCESSFUL! UPDATED ${updateCount}, ADDED ${insertCount} TEACHER RECORDS.`)
    fetchTeachers()
  } catch (err) {
    uploadProgress.value.show = false
    toast.error("IMPORT FAILED: " + err.message)
  } finally {
    e.target.value = ''
  }
}

onMounted(fetchTeachers)
</script>