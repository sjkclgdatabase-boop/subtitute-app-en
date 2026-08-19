<template>
  <!-- 🌟 核心：给最外层一个定死的最小宽度 (min-w-[1024px]) -->
  <div class="p-4 sm:p-8 mx-auto min-h-screen space-y-8 min-w-[1024px]">
    
    <!-- Header Section -->
    <div class="bg-white rounded-3xl p-6 sm:p-8 shadow-sm ring-1 ring-slate-900/5 space-y-2">
      <h1 class="text-xl sm:text-2xl md:text-3xl font-extrabold tracking-tight text-transparent bg-clip-text bg-gradient-to-r from-slate-900 via-indigo-800 to-violet-800 flex items-center gap-3">
        <CalendarCheck2 class="w-8 h-8 text-indigo-700 shrink-0" />
        TEACHER ABSENCE & SUBSTITUTE REGISTRATION
      </h1>
      <p class="text-slate-500 text-xs sm:text-sm font-medium leading-relaxed">
        ADAPTED FOR DUAL-SESSION OPERATIONS. SELECT SESSION, TEACHER, AND TASK CATEGORY, THEN CHOOSE THE PERIODS THAT REQUIRE A SUBSTITUTE TO GENERATE TASKS.
      </p>
    </div>

    <!-- Step 1: Basic Information Selection -->
    <div class="bg-white rounded-3xl shadow-sm ring-1 ring-slate-900/5 p-6 sm:p-8">
      <h2 class="text-lg font-bold text-slate-900 mb-6 flex items-center gap-2">
        <span class="w-8 h-8 rounded-xl bg-indigo-50 text-indigo-600 flex items-center justify-center font-bold text-xs shrink-0">1</span>
        <span>BASIC ABSENCE INFORMATION</span>
      </h2>
      
      <!-- 🌟 优化：改为 lg:grid-cols-3，让三项完美并排并防堆叠 -->
      <div class="grid grid-cols-1 lg:grid-cols-3 gap-6 items-center">
        
        <!-- 1. Session Selection -->
        <div>
          <label class="block text-xs font-bold text-slate-700 mb-2 uppercase tracking-wider">SESSION SELECTION</label>
          <!-- 锁定高度 h-14 -->
          <div class="bg-slate-100 p-1.5 rounded-2xl flex items-center shadow-inner h-14">
            <button 
              @click="currentSession = 'morning'; selectedTeacherId = ''; dailyClasses = []" 
              class="flex-1 h-full rounded-xl text-xs font-bold transition-all flex items-center justify-center gap-2 cursor-pointer"
              :class="currentSession === 'morning' 
                ? 'bg-white text-indigo-600 shadow-sm ring-2 ring-indigo-600' 
                : 'text-slate-500 hover:text-slate-900'"
            >
              <Sun class="w-4 h-4 shrink-0 text-amber-500" /> <span class="truncate">MORNING SESSION</span>
            </button>
            <button 
              @click="currentSession = 'afternoon'; selectedTeacherId = ''; dailyClasses = []" 
              class="flex-1 h-full rounded-xl text-xs font-bold transition-all flex items-center justify-center gap-2 cursor-pointer"
              :class="currentSession === 'afternoon' 
                ? 'bg-white text-orange-600 shadow-sm ring-2 ring-orange-500' 
                : 'text-slate-500 hover:text-slate-900'"
            >
              <Moon class="w-4 h-4 shrink-0 text-indigo-400" /> <span class="truncate">AFTERNOON SESSION</span>
            </button>
          </div>
        </div>

        <!-- 2. Teacher Selector -->
        <div>
          <label class="block text-xs font-bold text-slate-700 mb-2 uppercase tracking-wider">ABSENT TEACHER</label>
          <!-- 锁定高度 h-14，移除原有的 py-3 -->
          <div class="relative flex items-center bg-slate-50 border border-slate-200 rounded-2xl px-4 h-14 shadow-sm hover:border-slate-300 transition w-full">
            <div class="w-8 h-8 rounded-xl bg-white flex items-center justify-center mr-3 shrink-0 shadow-xs text-indigo-600">
              <GraduationCap class="w-4 h-4" />
            </div>
            <!-- 下拉菜单占满 h-full -->
            <select 
              v-model="selectedTeacherId" 
              @change="fetchDailyTimetable"
              class="w-full h-full bg-transparent border-none text-slate-800 font-semibold focus:ring-0 cursor-pointer text-xs appearance-none outline-none pr-8 truncate"
            >
              <option value="" disabled>-- SELECT {{ currentSession === 'morning' ? 'MORNING' : 'AFTERNOON' }} TEACHER --</option>
              <option v-for="teacher in filteredTeachersList" :key="teacher.id" :value="teacher.id">
                {{ teacher.name }}{{ teacher.subject ? ` (${teacher.subject})` : '' }}
              </option>
            </select>
            <div class="absolute right-4 pointer-events-none text-slate-400 shrink-0">
              <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 9l-7 7-7-7"></path></svg>
            </div>
          </div>
        </div>

        <!-- 3. Date Selection -->
        <div>
          <label class="block text-xs font-bold text-slate-700 mb-2 uppercase tracking-wider">ABSENCE DATE</label>
          <!-- 锁定高度 h-14，移除原有的 py-3 -->
          <div class="relative flex items-center">
            <input 
              type="date" 
              v-model="leaveDate"
              @change="fetchDailyTimetable"
              class="w-full px-4 h-14 bg-slate-50 border border-slate-200 rounded-2xl text-xs focus:outline-none focus:ring-2 focus:ring-indigo-500/50 transition-all font-semibold text-slate-800 cursor-pointer"
            />
          </div>
        </div>

      </div>

      <!-- 🌟 Task Category Selection -->
      <div class="mt-8">
        <label class="block text-xs font-bold text-slate-700 mb-3 uppercase tracking-wider">ABSENCE TYPE / TASK NATURE (REQUIRED)</label>
        <div class="grid grid-cols-1 sm:grid-cols-3 gap-4">
          <label v-for="cat in leaveCategories" :key="cat.value" class="cursor-pointer relative">
            <input type="radio" v-model="leaveCategory" :value="cat.value" class="peer sr-only" />
            <div class="p-4 rounded-2xl border-2 border-slate-100 bg-slate-50 hover:bg-slate-100 transition-all peer-checked:border-indigo-600 peer-checked:bg-indigo-50 flex flex-col gap-1.5 shadow-sm">
              <div class="flex items-center gap-2.5">
                <component :is="cat.iconComponent" class="w-5 h-5 text-indigo-600 shrink-0" />
                <span class="text-sm font-bold text-slate-800">{{ cat.label }}</span>
              </div>
              <span class="text-[11px] text-slate-500 font-medium ml-7">{{ cat.desc }}</span>
            </div>
          </label>
        </div>
      </div>

      <!-- 🌟 Specific Reason & Auto-Capitalization -->
      <div class="mt-6">
        <label class="block text-xs font-bold text-slate-700 mb-2 uppercase tracking-wider">SPECIFIC REASON (OPTIONAL, AUTO-UPPERCASE)</label>
        <input 
          type="text" 
          v-model="leaveReason"
          :placeholder="currentPlaceholder"
          class="w-full px-4 py-3 bg-slate-50 border border-slate-200 rounded-2xl text-xs focus:outline-none focus:ring-2 focus:ring-indigo-500/50 transition-all font-semibold text-slate-800 uppercase"
        />
        <p class="text-[10px] text-slate-400 mt-2 ml-1">Upon submission, reports will automatically categorize this as: <strong class="text-indigo-600">[{{ leaveCategory }}] {{ leaveReason ? leaveReason.toUpperCase() : 'UNSPECIFIED' }}</strong></p>
      </div>

    </div>

    <!-- Step 2: Select Time Slots to Substitute Preview -->
    <transition enter-active-class="transition duration-300 ease-out" enter-from-class="opacity-0 translate-y-4" enter-to-class="opacity-100 translate-y-0">
      <div v-if="selectedTeacherId && leaveDate" class="bg-white rounded-3xl shadow-sm ring-1 ring-slate-900/5 p-6 sm:p-8">
        <div class="flex flex-col sm:flex-row justify-between items-start sm:items-center mb-6 gap-4 border-b border-slate-100 pb-4">
          <div>
            <h2 class="text-lg font-bold text-slate-900 flex items-center gap-2">
              <span class="w-8 h-8 rounded-xl bg-emerald-50 text-emerald-600 flex items-center justify-center font-bold text-xs shrink-0">2</span>
              <span>SELECT SUBSTITUTE TIME SLOTS</span>
            </h2>
            <p class="text-xs text-slate-500 mt-1 font-medium">CHECK THE CARDS TO GENERATE SUBSTITUTE TASKS. UNCHECKED SLOTS WILL NOT BE PROCESSED.</p>
          </div>

          <div class="flex flex-wrap items-center gap-3 shrink-0">
            <!-- Select All / Deselect All Shortcuts -->
            <button @click="selectAll(true)" class="px-3 py-2 bg-slate-100 hover:bg-slate-200 rounded-xl text-xs font-bold text-slate-700 cursor-pointer transition shrink-0">
              SELECT ALL
            </button>
            <button @click="selectAll(false)" class="px-3 py-2 bg-slate-100 hover:bg-slate-200 rounded-xl text-xs font-bold text-slate-700 cursor-pointer transition shrink-0">
              DESELECT ALL
            </button>
            <span class="px-3 py-1.5 bg-indigo-50 text-indigo-700 rounded-full text-xs font-bold shrink-0 flex items-center gap-1">
              <CalendarDays class="w-3.5 h-3.5" />
              {{ computedWeekdayName }}
            </span>
          </div>
        </div>

        <!-- Timetable Display Area -->
        <div v-if="loading" class="py-12 text-center">
          <div class="w-8 h-8 border-4 border-indigo-500/30 border-t-indigo-600 rounded-full animate-spin mx-auto"></div>
          <p class="text-slate-500 text-xs font-bold mt-4">FETCHING TIMETABLE FROM DATABASE...</p>
        </div>

        <div v-else-if="dailyClasses.length === 0" class="bg-slate-50 rounded-2xl p-8 text-center border border-slate-100 space-y-2">
          <div class="w-12 h-12 rounded-2xl bg-emerald-100 text-emerald-600 flex items-center justify-center mx-auto mb-3">
            <CheckCircle2 class="w-6 h-6" />
          </div>
          <p class="text-slate-900 font-bold text-sm">THIS TEACHER HAS NO CLASSES SCHEDULED ON THIS DATE</p>
          <p class="text-slate-500 text-xs font-medium">NO SUBSTITUTE TEACHER NEEDED, ADMINISTRATORS CAN DIRECTLY APPROVE THE LEAVE.</p>
        </div>

        <div v-else class="space-y-3">
          <!-- Selection Cards Area -->
          <div 
            v-for="cls in dailyClasses" 
            :key="cls.period"
            @click="toggleClassSelection(cls)"
            :class="cls.selected 
              ? 'border-indigo-600 bg-indigo-50/40 shadow-sm' 
              : 'border-slate-200 bg-slate-50/50 opacity-60 hover:opacity-100'"
            class="p-4 border-2 rounded-2xl transition-all cursor-pointer flex items-center justify-between select-none gap-4"
          >
            <div class="flex items-center gap-4">
              <!-- Checkbox -->
              <input 
                type="checkbox" 
                :checked="cls.selected"
                @click.stop="toggleClassSelection(cls)"
                class="w-5 h-5 text-indigo-600 rounded border-slate-300 focus:ring-indigo-500 cursor-pointer shrink-0"
              />

              <!-- Period Number Box -->
              <div class="w-16 h-14 rounded-2xl bg-white text-indigo-700 flex flex-col items-center justify-center font-bold shadow-sm ring-1 ring-slate-900/5 shrink-0 px-1">
                <span class="text-sm leading-none font-black" v-html="formatOrdinalNumberHtml(cls.period)"></span>
                <span class="text-[9px] text-slate-400 font-bold uppercase mt-1 tracking-wider">PERIOD</span>
              </div>

              <!-- Class & Subject -->
              <div>
                <div class="flex items-center gap-2">
                  <p class="font-bold text-slate-900 text-sm truncate">{{ cls.class_name }}</p>
                  <span v-if="cls.class_name.includes('/')" class="px-2 py-0.5 bg-violet-100 text-violet-700 rounded text-[10px] font-bold shrink-0">COMBINED CLASS</span>
                </div>
                <p class="text-xs text-indigo-600 font-semibold mt-0.5 truncate">{{ cls.subject }}</p>
              </div>
            </div>

            <!-- Selection Status Badge -->
            <div class="shrink-0">
              <span 
                :class="cls.selected ? 'bg-indigo-600 text-white' : 'bg-slate-200 text-slate-600'" 
                class="text-xs font-bold px-3 py-1.5 rounded-full transition whitespace-nowrap block"
              >
                {{ cls.selected ? 'SELECTED FOR SUBSTITUTE' : 'NO SUBSTITUTE' }}
              </span>
            </div>
          </div>

          <!-- Submit Button & Statistics -->
          <div class="mt-8 pt-6 border-t border-slate-100 flex flex-col sm:flex-row items-start sm:items-center justify-between gap-4">
            <div class="text-xs text-slate-500 font-bold truncate">
              SELECTED <strong class="text-indigo-600 text-sm font-black">{{ selectedClassesCount }}</strong> SUBSTITUTE TASKS
            </div>

            <button 
              @click="submitLeaveRequests" 
              :disabled="isSubmitting || selectedClassesCount === 0"
              class="group flex items-center justify-center px-6 py-3 text-xs font-bold text-white bg-slate-900 rounded-2xl hover:bg-slate-800 hover:shadow-lg hover:-translate-y-0.5 transition-all shadow-md disabled:opacity-50 disabled:cursor-not-allowed cursor-pointer shrink-0 w-full sm:w-auto"
            >
              <span v-if="!isSubmitting" class="truncate">GENERATE SUBSTITUTE TASKS ({{ selectedClassesCount }})</span>
              <span v-else>GENERATING...</span>
              <ArrowRight v-if="!isSubmitting" class="w-4 h-4 ml-2 group-hover:translate-x-1 transition-transform shrink-0" />
            </button>
          </div>
        </div>
      </div>
    </transition>

  </div>
</template>

<script setup>
import { ref, onMounted, computed } from 'vue'
import { supabase } from '../services/supabase'
import { useRouter } from 'vue-router'
import { useToast } from '../utils/toast'
import { 
  CalendarCheck2, 
  Sun, 
  Moon, 
  GraduationCap, 
  CalendarDays, 
  UserCheck, 
  BriefcaseBusiness, 
  Building2, 
  CheckCircle2, 
  ArrowRight 
} from 'lucide-vue-next'

const router = useRouter()
const toast = useToast()

const teachersList = ref([])
const currentSession = ref('morning')
const selectedTeacherId = ref('')
const leaveDate = ref('')
const leaveReason = ref('')

// 🌟 Added: Category Data (English Version - Lucide Icons Configured)
const leaveCategory = ref('PERSONAL LEAVE')
const leaveCategories = [
  { value: 'PERSONAL LEAVE', iconComponent: UserCheck, label: 'PERSONAL LEAVE', desc: 'SICK/EMERGENCY/MATERNITY', placeholder: 'E.G.: MC, CRK, MATERNITY LEAVE' },
  { value: 'OFFICIAL DUTY', iconComponent: BriefcaseBusiness, label: 'OFFICIAL DUTY', desc: 'MEETINGS/COURSES / WORKSHOPS', placeholder: 'E.G.: PPD MEETING, COURSE, PKL' },
  { value: 'INTERNAL TASK', iconComponent: Building2, label: 'INTERNAL TASK', desc: 'SCHOOL DUTIES/EVENTS', placeholder: 'E.G.: INTERNAL MEETING, EVENT SUPERVISION' }
]

// Dynamic Placeholder
const currentPlaceholder = computed(() => {
  const cat = leaveCategories.find(c => c.value === leaveCategory.value)
  return cat ? cat.placeholder : 'PLEASE ENTER SPECIFIC REASON...'
})

const dailyClasses = ref([])
const loading = ref(false)
const isSubmitting = ref(false)

const dayNames = ['SUNDAY', 'MONDAY', 'TUESDAY', 'WEDNESDAY', 'THURSDAY', 'FRIDAY', 'SATURDAY']

// Format Ordinal Number HTML
const formatOrdinalNumberHtml = (p) => {
  const num = Number(p)
  if (isNaN(num)) return `${p}`
  
  let suffix = 'th'
  if (num % 100 < 11 || num % 100 > 13) {
    switch (num % 10) {
      case 1: suffix = 'st'; break;
      case 2: suffix = 'nd'; break;
      case 3: suffix = 'rd'; break;
    }
  }
  return `${num}<sup style="font-size: 9px; vertical-align: super;">${suffix}</sup>`
}

onMounted(async () => {
  const { data } = await supabase.from('teachers').select('id, name, subject, session')
  if (data) teachersList.value = data
})

const filteredTeachersList = computed(() => {
  return teachersList.value.filter(t => (t.session || 'morning') === currentSession.value)
})

const computedWeekdayNumber = computed(() => {
  if (!leaveDate.value) return null
  const dateObj = new Date(leaveDate.value)
  let day = dateObj.getDay()
  return day === 0 ? 7 : day
})

const computedWeekdayName = computed(() => {
  if (!leaveDate.value) return ''
  const dateObj = new Date(leaveDate.value)
  return dayNames[dateObj.getDay()]
})

// Count selected periods
const selectedClassesCount = computed(() => {
  return dailyClasses.value.filter(cls => cls.selected).length
})

// Toggle selection state
const toggleClassSelection = (cls) => {
  cls.selected = !cls.selected
}

// Quick select / deselect all
const selectAll = (status) => {
  dailyClasses.value.forEach(cls => {
    cls.selected = status
  })
}

// Fetch timetable and initialize selected: true state for each period
const fetchDailyTimetable = async () => {
  if (!selectedTeacherId.value || !leaveDate.value) {
    dailyClasses.value = []
    return
  }

  loading.value = true
  try {
    const { data, error } = await supabase
      .from('timetable')
      .select('*')
      .eq('teacher_id', selectedTeacherId.value)
      .eq('weekday', computedWeekdayNumber.value)
      .order('period', { ascending: true })

    if (error) throw error

    const periodMap = new Map()

    ;(data || []).forEach(cls => {
      if (!periodMap.has(cls.period)) {
        periodMap.set(cls.period, {
          ...cls,
          is_combined: false,
          selected: true
        })
      } else {
        const existing = periodMap.get(cls.period)
        if (!existing.class_name.includes(cls.class_name)) {
          existing.class_name = `${existing.class_name}/${cls.class_name}`
          existing.is_combined = true
        }
      }
    })

    dailyClasses.value = Array.from(periodMap.values())
  } catch (error) {
    toast.error("FAILED TO FETCH TIMETABLE: " + error.message)
  } finally {
    loading.value = false
  }
}

// 🚀 Core optimization: Conflict-free & Auto-Uppercase Submission Logic (with precise MMI control)
const submitLeaveRequests = async () => {
  const selectedList = dailyClasses.value.filter(cls => cls.selected)
  if (selectedList.length === 0) {
    return toast.error("PLEASE SELECT AT LEAST ONE SUBJECT FOR SUBSTITUTE TEACHING!")
  }

  isSubmitting.value = true
  try {
    const currentTeacher = teachersList.value.find(t => t.id === selectedTeacherId.value)
    const teacherName = currentTeacher ? currentTeacher.name : 'UNKNOWN TEACHER'

    // 🌟 Merge category and reason, force uppercase
    const rawReason = leaveReason.value.trim()
    const formattedReason = rawReason 
      ? `[${leaveCategory.value}] ${rawReason.toUpperCase()}`
      : `[${leaveCategory.value}] UNSPECIFIED DETAILS`

    // 1. Check existing leave requests in database for this teacher on this date to prevent conflicts
    const { data: existingLeaves } = await supabase
      .from('leave_requests')
      .select('period, status')
      .eq('teacher_id', selectedTeacherId.value)
      .eq('leave_date', leaveDate.value)

    const existingPeriodsMap = new Map()
    if (existingLeaves) {
      existingLeaves.forEach(req => {
        existingPeriodsMap.set(Number(req.period), req.status)
      })
    }

    // 2. Only pending records can be deleted and overwritten
    await supabase
      .from('leave_requests')
      .delete()
      .eq('teacher_id', selectedTeacherId.value)
      .eq('leave_date', leaveDate.value)
      .eq('status', 'pending')

    // 3. Build data to insert
    const requests = []
    const periodsForMMI = []

    selectedList.forEach(cls => {
      const p = Number(cls.period)
      
      if (existingPeriodsMap.get(p) === 'assigned') {
        return
      }

      requests.push({
        teacher_id: selectedTeacherId.value,
        leave_date: leaveDate.value,
        weekday: cls.weekday,
        period: cls.period,
        class_name: cls.class_name,
        subject: cls.subject,
        reason: formattedReason, // 🌟 Save formatted reason
        status: 'pending'
      })
      periodsForMMI.push(p)
    })

    if (requests.length === 0) {
      toast.info("THE SELECTED SLOTS ALREADY HAVE SUBSTITUTE TASKS ASSIGNED, NO NEED TO RE-GENERATE.")
      isSubmitting.value = false
      return
    }

    // 4. Insert newly filtered non-conflicting tasks
    const { error: leaveError } = await supabase.from('leave_requests').insert(requests)
    if (leaveError) throw leaveError

    // 5. 🌟 Precise MMI Control: Ensure single period selections don't incorrectly span across gaps
    if (periodsForMMI.length > 0) {
      periodsForMMI.sort((a, b) => a - b)
      
      const startP = periodsForMMI[0]
      const endP = periodsForMMI.length === 1 ? periodsForMMI[0] : periodsForMMI[periodsForMMI.length - 1]

      const mmiLogPayload = {
        interruption_date: leaveDate.value,
        type: 'teacher',
        start_period: startP,
        end_period: endP,
        reason: formattedReason, // 🌟 Save formatted reason to MMI log
        target_display: `TEACHER: ${teacherName}`,
        remarks: `(INVOLVING SLOTS: ${periodsForMMI.join(', ')} | SUBJECTS: ${requests.map(c => `${c.class_name}(${c.subject})`).join(', ')})`
      }

      const { error: mmiError } = await supabase.from('mmi_interruptions').insert([mmiLogPayload])
      if (mmiError) {
        console.error("FAILED TO SYNCHRONIZE MMI RECORD:", mmiError.message)
      }
    }

    toast.success(`SUBSTITUTE TASKS SUCCESSFULLY GENERATED AND RECORDED IN MMI HISTORY!`)
    router.push('/')
  } catch (error) {
    toast.error("GENERATION FAILED: " + error.message)
  } finally {
    isSubmitting.value = false
  }
}
</script>