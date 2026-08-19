<template>
  <div class="p-8 max-w-7xl mx-auto min-h-screen space-y-8">
    
    <div class="bg-white rounded-3xl p-8 shadow-sm ring-1 ring-slate-900/5 space-y-2">
      <h1 class="text-2xl sm:text-3xl font-extrabold tracking-tight text-transparent bg-clip-text bg-gradient-to-r from-slate-900 via-indigo-800 to-violet-800 flex items-center gap-3">
        <Wrench class="w-8 h-8 text-indigo-700 shrink-0" />
        SYSTEM SETTINGS & OVERALL MAINTENANCE
      </h1>
      <p class="text-slate-500 text-xs sm:text-sm font-medium leading-relaxed">
        SCHOOL TIME PARAMETER CONFIGURATION, BASIC CLASS MANAGEMENT, MMI TARGET SETTINGS, SUPPORTING DATA CLEANING & RECOVERY BEFORE LAUNCH.
      </p>
    </div>

    <div class="bg-white rounded-3xl shadow-sm ring-1 ring-slate-900/5 p-8 transition-all duration-300 hover:shadow-md space-y-6">
      <h2 class="text-lg font-bold text-slate-900 mb-6 flex items-center gap-2">
        <span class="w-8 h-8 rounded-2xl bg-indigo-50 text-indigo-600 flex items-center justify-center font-bold text-xs"><UserCog class="w-4 h-4" /></span>
        ADMIN ACCOUNT MANAGEMENT
      </h2>
      
      <div class="grid grid-cols-1 lg:grid-cols-2 gap-8">
        <div class="bg-slate-50 p-6 rounded-2xl border border-slate-200 space-y-4">
          <h3 class="text-xs font-bold uppercase tracking-wider text-slate-900">ADD NEW ADMINISTRATOR</h3>
          <div>
            <label class="block text-xs font-bold text-slate-700 mb-1">NEW USER EMAIL</label>
            <input 
              v-model="newUserEmail" 
              type="email" 
              placeholder="teacher@school.edu.my" 
              class="w-full px-4 h-11 bg-white border border-slate-200 rounded-xl text-xs font-bold text-slate-800 focus:outline-none focus:ring-2 focus:ring-indigo-500/50"
            />
          </div>

          <div>
            <label class="block text-xs font-bold text-slate-700 mb-1">INITIAL PASSWORD</label>
            <input 
              v-model="newUserPassword" 
              type="password" 
              placeholder="At least 6 characters" 
              class="w-full px-4 h-11 bg-white border border-slate-200 rounded-xl text-xs font-bold text-slate-800 focus:outline-none focus:ring-2 focus:ring-indigo-500/50"
            />
          </div>

          <button 
            @click="handleCreateUser" 
            :disabled="creatingUser"
            class="w-full bg-indigo-600 hover:bg-indigo-700 text-white h-11 rounded-xl text-xs font-bold shadow-sm transition cursor-pointer disabled:opacity-50"
          >
            {{ creatingUser ? 'CREATING USER...' : 'CONFIRM & CREATE USER' }}
          </button>
        </div>

        <div class="bg-slate-50 p-6 rounded-2xl border border-slate-200 space-y-4 flex flex-col">
          <div class="flex items-center justify-between">
            <h3 class="text-xs font-bold uppercase tracking-wider text-slate-900 flex items-center gap-1.5">
              <ClipboardList class="w-4 h-4 text-indigo-600" /> EXISTING USER LIST
            </h3>
            <button @click="fetchUsers" class="text-xs text-indigo-600 font-bold hover:underline cursor-pointer">REFRESH</button>
          </div>

          <div class="flex-1 overflow-y-auto max-h-56 space-y-2 border border-slate-200 bg-white rounded-xl p-3">
            <div v-if="userList.length === 0" class="text-center text-xs text-slate-400 py-8">NO USERS FOUND OR LOADING...</div>
            <div v-for="u in userList" :key="u.id" class="flex items-center justify-between p-2.5 bg-slate-50 rounded-xl border border-slate-100 text-xs">
              <div class="truncate pr-2">
                <div class="font-bold text-slate-800 truncate">{{ u.email }}</div>
                <div class="text-[10px] text-slate-400">Created: {{ new Date(u.created_at).toLocaleDateString() }}</div>
              </div>
              <button 
                @click="handleDeleteUser(u.id, u.email)" 
                class="text-rose-500 hover:text-rose-700 bg-rose-50 hover:bg-rose-100 px-3 py-1.5 rounded-lg font-bold transition shrink-0 cursor-pointer"
              >
                DELETE
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>

    <div class="bg-white rounded-3xl shadow-sm ring-1 ring-slate-900/5 p-8 transition-all duration-300 hover:shadow-md space-y-6">
      <h2 class="text-lg font-bold text-slate-900 mb-6 flex items-center gap-2">
        <span class="w-8 h-8 rounded-2xl bg-cyan-50 text-cyan-600 flex items-center justify-center font-bold text-xs"><Image class="w-4 h-4" /></span>
        SCHOOL LOGO & DISPLAY SETTINGS
      </h2>
      <p class="text-slate-500 text-xs font-medium">UPDATE SCHOOL LOGO & NAME. ONCE SAVED, THE MENU BAR AND LOGIN PAGE WILL BE UPDATED IN REAL-TIME.</p>

      <div class="space-y-6">
        <div>
          <label class="block text-xs font-bold text-slate-700 mb-2 uppercase tracking-wider">CURRENT SCHOOL NAME</label>
          <input 
            v-model="schoolNameSetting" 
            type="text" 
            placeholder="ENTER SCHOOL NAME..." 
            class="w-full sm:w-96 px-4 h-11 bg-slate-50 border border-slate-200 rounded-2xl text-xs font-bold text-slate-800 focus:outline-none focus:ring-2 focus:ring-indigo-500/50 transition-all"
          />
        </div>

        <div>
          <label class="block text-xs font-bold text-slate-700 mb-2 uppercase tracking-wider">CHANGE SCHOOL LOGO (SUPPORTS LOCAL UPLOAD)</label>
          
          <div class="flex items-center gap-6">
            <div class="w-20 h-20 rounded-2xl border border-slate-200 bg-slate-50 flex items-center justify-center p-2 overflow-hidden shadow-inner shrink-0">
              <img :src="schoolLogoSetting || '/logo.png'" alt="Logo Preview" class="w-full h-full object-contain" />
            </div>

            <div class="flex-1 space-y-2">
              <label class="relative inline-flex cursor-pointer bg-slate-900 hover:bg-slate-800 text-white px-5 h-11 rounded-2xl text-xs font-bold transition shadow-sm items-center gap-2">
                <span>📂 CHOOSE NEW LOGO IMAGE</span>
                <input type="file" accept="image/*" @change="handleSettingsLogoUpload" class="absolute inset-0 opacity-0 cursor-pointer w-full h-full" />
              </label>
              <p class="text-[11px] text-slate-400 font-medium">SUPPORTS PNG, JPG FORMATS. SIZES UNDER 2MB ARE RECOMMENDED.</p>
            </div>
          </div>
        </div>
      </div>

      <div class="mt-10 pt-6 border-t border-slate-100 flex justify-end">
        <button 
          @click="saveSchoolIdentity" 
          class="bg-indigo-600 hover:bg-indigo-700 text-white px-6 h-11 rounded-2xl text-xs font-bold shadow-sm transition-all duration-200 cursor-pointer"
        >
          SAVE LOGO & NAME CHANGES
        </button>
      </div>
    </div>

    <div class="bg-white rounded-3xl shadow-sm ring-1 ring-slate-900/5 p-8 transition-all duration-300 hover:shadow-md space-y-6">
      <h2 class="text-lg font-bold text-slate-900 mb-6 flex items-center gap-2">
        <span class="w-8 h-8 rounded-2xl bg-indigo-50 text-indigo-600 flex items-center justify-center font-bold text-xs"><Clock class="w-4 h-4" /></span>
        SCHOOL HOURS CONFIGURATION
      </h2>
      
      <div class="space-y-6">
        <div>
          <label class="block text-xs font-bold text-slate-700 mb-2 uppercase tracking-wider">NUMBER OF SCHOOL DAYS PER WEEK (DAYS)</label>
          <div class="flex items-center gap-4">
            <input 
              v-model.number="config.daysPerWeek" 
              type="number" min="1" max="7" 
              class="w-full sm:w-64 px-4 h-11 bg-slate-50 border border-slate-200 rounded-2xl text-xs font-bold text-slate-800 focus:outline-none focus:ring-2 focus:ring-indigo-500/50 transition-all"
            />
            <span class="text-xs text-slate-400 font-medium">E.G. 5 (REPRESENTING MONDAY TO FRIDAY)</span>
          </div>
        </div>

        <div>
          <label class="block text-xs font-bold text-slate-700 mb-2 uppercase tracking-wider">NUMBER OF TIME SLOTS PER DAY (SLOTS)</label>
          <div class="flex items-center gap-4">
            <input 
              v-model.number="config.periodsPerDay" 
              type="number" min="1" max="15" 
              class="w-full sm:w-64 px-4 h-11 bg-slate-50 border border-slate-200 rounded-2xl text-xs font-bold text-slate-800 focus:outline-none focus:ring-2 focus:ring-indigo-500/50 transition-all"
            />
            <span class="text-xs text-slate-400 font-medium">E.G. 11 (TOTAL SLOTS FOR MORNING SESSION)</span>
          </div>
        </div>
      </div>

      <div class="mt-10 pt-6 border-t border-slate-100 flex justify-end">
        <button 
          @click="saveConfig" 
          class="bg-slate-900 hover:bg-slate-800 text-white px-6 h-11 rounded-2xl text-xs font-bold shadow-sm transition-all duration-200 cursor-pointer"
        >
          SAVE CONFIGURATION
        </button>
      </div>
    </div>

    <div class="bg-white rounded-3xl shadow-sm ring-1 ring-slate-900/5 p-8 transition-all duration-300 hover:shadow-md space-y-6">
      <h2 class="text-lg font-bold text-slate-900 mb-2 flex items-center gap-2">
        <span class="w-8 h-8 rounded-2xl bg-indigo-50 text-indigo-700 flex items-center justify-center font-bold text-xs"><Calendar class="w-4 h-4" /></span>
        SCHOOL CALENDAR & HOLIDAYS CONFIGURATION
      </h2>
      <p class="text-slate-500 text-xs font-medium mb-6">Enter school weeks and holidays throughout the year. The system will automatically exclude holiday periods to accurately calculate annual targets.</p>

      <div class="flex flex-col sm:flex-row items-start sm:items-center justify-between gap-4 p-5 bg-slate-50 rounded-2xl border border-slate-200 mb-6">
        <div>
          <h3 class="text-xs font-bold uppercase tracking-wider text-slate-950">BULK IMPORT / EXPORT SCHOOL CALENDAR</h3>
          <p class="text-slate-500 text-xs mt-1 font-medium">SUPPORTS DOWNLOADING STANDARD CSV TEMPLATE, FILLING AND BULK UPLOADING.</p>
        </div>
        <div class="flex flex-wrap items-center gap-3">
          <button @click="downloadWeekTemplate" class="bg-white hover:bg-slate-100 text-slate-700 border border-slate-200 px-4 h-11 rounded-2xl text-xs font-bold transition shadow-sm flex items-center gap-2 cursor-pointer">
            <Download class="w-4 h-4 text-indigo-600" /> DOWNLOAD CALENDAR TEMPLATE
          </button>
          
          <label class="relative cursor-pointer bg-indigo-600 hover:bg-indigo-700 text-white px-4 h-11 rounded-2xl text-xs font-bold transition shadow-sm flex items-center gap-2">
            <span>📂 CHOOSE CALENDAR FILE</span>
            <input type="file" ref="weekFileInput" accept=".csv" @change="handleWeekFileUpload" class="absolute inset-0 opacity-0 cursor-pointer w-full h-full" />
          </label>
        </div>
      </div>

      <div class="bg-slate-50 p-5 rounded-2xl border border-slate-200 mb-6 grid grid-cols-1 sm:grid-cols-5 gap-3 items-end">
        <div>
          <label class="block text-xs font-bold text-slate-700 mb-1">WEEK NO.</label>
          <input type="number" v-model.number="newWeek.week_number" placeholder="1" class="w-full bg-white border border-slate-200 px-3 h-11 rounded-xl text-xs font-bold text-slate-800" />
        </div>
        <div>
          <label class="block text-xs font-bold text-slate-700 mb-1">START DATE (YYYY-MM-DD)</label>
          <input type="date" v-model="newWeek.start_date" class="w-full bg-white border border-slate-200 px-3 h-11 rounded-xl text-xs font-bold text-slate-800 cursor-pointer" />
        </div>
        <div>
          <label class="block text-xs font-bold text-slate-700 mb-1">END DATE (YYYY-MM-DD)</label>
          <input type="date" v-model="newWeek.end_date" class="w-full bg-white border border-slate-200 px-3 h-11 rounded-xl text-xs font-bold text-slate-800 cursor-pointer" />
        </div>
        <div>
          <label class="block text-xs font-bold text-slate-700 mb-1">TYPE</label>
          <select v-model="newWeek.is_school_week" class="w-full bg-white border border-slate-200 px-3 h-11 rounded-xl text-xs font-bold text-slate-800 cursor-pointer">
            <option :value="true">🟢 SCHOOL WEEK</option>
            <option :value="false">🔴 HOLIDAY WEEK</option>
          </select>
        </div>
        <button @click="addSchoolWeek" class="bg-slate-900 hover:bg-slate-800 text-white h-11 rounded-xl text-xs font-bold shadow-md transition-all cursor-pointer">
          ➕ ADD ONE
        </button>
      </div>

      <div class="space-y-3">
        <div class="flex items-center justify-between mb-2">
          <h3 class="text-xs font-bold uppercase tracking-wider text-slate-500">SAVED SCHOOL CALENDAR LIST</h3>
          <span class="text-xs bg-indigo-50 text-indigo-700 px-2.5 py-1 rounded-full font-bold">TOTAL WEEKS {{ allSchoolWeeks.length }}</span>
        </div>

        <div class="divide-y divide-slate-100 border border-slate-200 rounded-2xl overflow-hidden max-h-64 overflow-y-auto">
          <div v-if="allSchoolWeeks.length === 0" class="p-8 text-center text-xs text-slate-400 font-medium">
            NO CALENDAR DATA ENTERED. PLEASE ADD OR IMPORT ABOVE.
          </div>
          <div v-for="w in allSchoolWeeks" :key="w.id" class="p-3.5 flex items-center justify-between bg-white hover:bg-slate-50 transition">
            <div class="flex items-center gap-4 text-xs font-bold text-slate-700">
              <span class="px-2.5 py-1 bg-indigo-50 text-indigo-700 rounded-xl font-bold">WEEK {{ w.week_number }}</span>
              <span class="text-slate-600 font-medium">{{ w.start_date }} ~ {{ w.end_date }}</span>
              <span :class="w.is_school_week ? 'text-emerald-700 bg-emerald-50 border-emerald-200' : 'text-red-700 bg-red-50 border-red-200'" class="px-2.5 py-1 rounded-full text-[10px] font-black border">
                {{ w.is_school_week ? '🟢 SCHOOL WEEK' : '🔴 HOLIDAY WEEK' }}
              </span>
            </div>
            <button @click="deleteSchoolWeek(w.id)" class="text-slate-400 hover:text-red-600 text-xs font-bold px-3 py-1.5 rounded-xl transition hover:bg-red-50 cursor-pointer">
              🗑️ DELETE
            </button>
          </div>
        </div>
      </div>
    </div>

    <div class="bg-white rounded-3xl shadow-sm ring-1 ring-slate-900/5 p-8 transition-all duration-300 hover:shadow-md space-y-6">
      <h2 class="text-lg font-bold text-slate-900 mb-6 flex items-center gap-2">
        <span class="w-8 h-8 rounded-2xl bg-violet-50 text-violet-600 flex items-center justify-center font-bold text-xs"><School class="w-4 h-4" /></span>
        SCHOOL BASIC CLASS MANAGEMENT
      </h2>
      <p class="text-slate-500 text-xs font-medium mb-6">MAINTAIN STANDARD SCHOOL CLASSES FOR TIMETABLES, LEAVES & MMI INTERRUPTION RECORDS.</p>

      <div class="bg-slate-50 p-5 rounded-2xl border border-slate-200 mb-8 flex flex-col sm:flex-row gap-4 items-end">
        <div class="w-full sm:w-1/3">
          <label class="block text-xs font-bold text-slate-700 mb-2 uppercase tracking-wider">GRADE:</label>
          <select v-model="newClassGrade" class="w-full bg-white border border-slate-200 px-4 h-11 rounded-2xl text-xs font-bold text-slate-800 cursor-pointer">
            <option v-for="g in [1, 2, 3, 4, 5, 6]" :key="g" :value="g">GRADE {{ g }}</option>
          </select>
        </div>

        <div class="w-full sm:w-1/2">
          <label class="block text-xs font-bold text-slate-700 mb-2 uppercase tracking-wider">CLASS NAME (E.G. 1A, 4C):</label>
          <input 
            type="text" 
            v-model="newClassName" 
            placeholder="ENTER STANDARD CLASS NAME..." 
            class="w-full bg-white border border-slate-200 px-4 h-11 rounded-2xl text-xs font-bold text-slate-800 focus:outline-none focus:ring-2 focus:ring-indigo-500"
          />
        </div>

        <button 
          @click="addClass" 
          class="w-full sm:w-auto bg-indigo-600 hover:bg-indigo-700 text-white px-6 h-11 rounded-2xl text-xs font-bold shadow-md transition-all shrink-0 cursor-pointer"
        >
          ➕ ADD CLASS
        </button>
      </div>

      <div class="space-y-6">
        <div class="flex items-center justify-between">
          <h3 class="text-xs font-bold uppercase tracking-wider text-slate-500">ENTERED SCHOOL CLASS LIST</h3>
          <button @click="toggleAllGrades" class="text-xs text-indigo-600 font-bold hover:underline cursor-pointer">
            {{ allExpanded ? 'COLLAPSE ALL' : 'EXPAND ALL' }}
          </button>
        </div>
        
        <div v-if="classList.length === 0" class="text-xs text-slate-400 font-medium py-6 text-center border border-dashed border-slate-200 rounded-2xl">
          NO CLASS DATA. PLEASE ADD ABOVE.
        </div>

        <div v-else class="space-y-4">
          <div v-for="(classListGroup, gradeKey) in groupedClasses" :key="gradeKey" class="space-y-2">
            
            <div v-if="classListGroup.length > 0" 
                 @click="toggleGrade(gradeKey)"
                 class="flex items-center gap-3 cursor-pointer group select-none">
              <span class="px-3 py-1 bg-violet-50 text-violet-700 rounded-xl text-xs font-black tracking-wider group-hover:bg-violet-100 transition flex items-center gap-2">
                GRADE {{ gradeKey }} 
                <span class="text-[10px] text-violet-400">{{ expandedGrades[gradeKey] ? '▼' : '▶' }}</span>
              </span>
              <div class="h-px bg-slate-100 flex-1"></div>
            </div>

            <div v-show="expandedGrades[gradeKey]" class="flex flex-wrap gap-3 pt-1">
              <div 
                v-for="c in classListGroup" 
                :key="c.id"
                class="p-3 bg-white border border-slate-200 rounded-2xl shadow-sm flex items-center justify-between group hover:border-indigo-300 transition-all min-w-[120px]"
              >
                <div>
                  <div class="text-xs font-bold text-slate-900">{{ c.class_name }}</div>
                  <div class="text-[10px] text-slate-400 font-medium">GRADE {{ c.grade }}</div>
                </div>
                <button 
                  @click.stop="deleteClass(c.id)" 
                  class="text-slate-300 hover:text-red-600 text-xs font-bold p-1 transition opacity-0 group-hover:opacity-100 ml-3 cursor-pointer"
                  title="DELETE"
                >
                  ✕
                </button>
              </div>
            </div>

          </div>
        </div>
      </div>
    </div>

    <div class="bg-white rounded-3xl shadow-sm ring-1 ring-slate-900/5 p-8 transition-all duration-300 hover:shadow-md space-y-6">
      <h2 class="text-lg font-bold text-slate-900 mb-6 flex items-center gap-2">
        <span class="w-8 h-8 rounded-2xl bg-emerald-50 text-emerald-600 flex items-center justify-center font-bold text-xs"><BarChart3 class="w-4 h-4" /></span>
        MMI SUBJECT TARGETS BULK CONFIGURATION
      </h2>
      <p class="text-slate-500 text-xs font-medium mb-6">DOWNLOAD STANDARD TEMPLATE, FILL IN SLOT TARGETS AND BASES FOR EACH GRADE AND SUBJECT, BULK IMPORT.</p>

      <div class="flex flex-col sm:flex-row items-start sm:items-center justify-between gap-4 p-5 bg-slate-50 rounded-2xl border border-slate-200">
        <div>
          <h3 class="text-xs font-bold uppercase tracking-wider text-slate-950">IMPORT / EXPORT SUBJECT TARGETS</h3>
          <p class="text-slate-500 text-xs mt-1 font-medium">SUPPORTS CSV TEMPLATE, AUTO-IMPORT TO UPDATE TARGET CONFIGURATIONS IN DATABASE.</p>
        </div>
        <div class="flex flex-wrap items-center gap-3">
          <button @click="downloadTemplate" class="bg-white hover:bg-slate-100 text-slate-700 border border-slate-200 px-4 h-11 rounded-2xl text-xs font-bold transition shadow-sm flex items-center gap-2 cursor-pointer">
            <Download class="w-4 h-4 text-indigo-600" /> DOWNLOAD TEMPLATE
          </button>
          
          <label class="relative cursor-pointer bg-indigo-600 hover:bg-indigo-700 text-white px-4 h-11 rounded-2xl text-xs font-bold transition shadow-sm flex items-center gap-2">
            <span>📂 CHOOSE SUBJECT FILE</span>
            <input type="file" ref="fileInput" accept=".csv" @change="handleFileUpload" class="absolute inset-0 opacity-0 cursor-pointer w-full h-full" />
          </label>
        </div>
      </div>
    </div>

    <div class="bg-white rounded-3xl shadow-sm ring-1 ring-slate-900/5 p-8 transition-all duration-300 hover:shadow-md space-y-6">
      <h2 class="text-lg font-bold text-slate-900 mb-6 flex items-center gap-2">
        <span class="w-8 h-8 rounded-2xl bg-blue-50 text-blue-600 flex items-center justify-center font-bold text-xs"><Database class="w-4 h-4" /></span>
        LOCAL DATA BACKUP & RECOVERY
      </h2>
      <p class="text-slate-500 text-xs font-medium mb-6">Create regular backups of all core school data and store them on your local computer to ensure data safety.</p>
      
      <div class="grid grid-cols-1 sm:grid-cols-2 gap-4">
        <div class="p-5 bg-slate-50 rounded-2xl border border-slate-200 flex flex-col justify-between space-y-4">
          <div>
            <h3 class="text-xs font-bold uppercase tracking-wider text-slate-950">EXPORT FULL SYSTEM BACKUP</h3>
            <p class="text-slate-500 text-xs mt-1 font-medium">DOWNLOAD A JSON BACKUP FILE CONTAINING MAIN SCHOOL DATA.</p>
          </div>
          <button @click="exportFullBackup" class="w-full bg-slate-900 hover:bg-slate-800 text-white h-11 rounded-2xl text-xs font-bold transition shadow-sm cursor-pointer flex items-center justify-center gap-2">
            <UploadCloud class="w-4 h-4 text-slate-300" /> EXPORT FULL SYSTEM BACKUP
          </button>
        </div>

        <div class="p-5 bg-slate-50 rounded-2xl border border-slate-200 flex flex-col justify-between space-y-4">
          <div>
            <h3 class="text-xs font-bold uppercase tracking-wider text-slate-950">RESTORE SYSTEM DATA</h3>
            <p class="text-slate-500 text-xs mt-1 font-medium">RESTORE SCHOOL DATA VIA PREVIOUS JSON BACKUP FILE.</p>
          </div>
          <label class="relative block w-full text-center cursor-pointer bg-indigo-600 hover:bg-indigo-700 text-white h-11 rounded-2xl text-xs font-bold transition shadow-sm flex items-center justify-center gap-2">
            <DownloadCloud class="w-4 h-4 text-indigo-200" /> <span>CHOOSE BACKUP FILE & RESTORE</span>
            <input type="file" accept=".json" @change="importFullBackup" class="absolute inset-0 opacity-0 cursor-pointer w-full h-full" />
          </label>
        </div>
      </div>
    </div>

    <div class="bg-white rounded-3xl shadow-sm ring-1 ring-slate-900/5 p-8 transition-all duration-300 hover:shadow-md space-y-6">
      <h2 class="text-lg font-bold text-slate-900 mb-6 flex items-center gap-2">
        <span class="w-8 h-8 rounded-2xl bg-red-50 text-red-600 flex items-center justify-center font-bold text-xs"><Wrench class="w-4 h-4" /></span>
        OFFICIAL LAUNCH DATA CLEANING & MAINTENANCE
      </h2>
      <p class="text-slate-500 text-xs font-medium mb-6">TO CLEAN TEST DATA BEFORE OFFICIAL LAUNCH, ONE-CLICK BACKUP IS SUPPORTED.</p>
      
      <div class="space-y-6">
        <div class="flex flex-col sm:flex-row justify-between items-start sm:items-center pb-6 border-b border-slate-100 gap-4">
          <div>
            <h3 class="text-xs font-bold uppercase tracking-wider text-slate-950">CLEAR ALL LEAVE & SUBSTITUTE RECORDS</h3>
            <p class="text-slate-500 text-xs mt-1 font-medium">DELETE ONLY LEAVE & SUBSTITUTE TEST DATA, KEEPING TEACHER PROFILES & TIMETABLES.</p>
          </div>
          <button @click="clearOnlyRecords" :disabled="loading" class="w-full sm:w-80 h-11 bg-amber-50 hover:bg-amber-100 text-amber-700 font-bold text-xs rounded-2xl transition shadow-sm border border-amber-200 disabled:opacity-50 shrink-0 cursor-pointer flex items-center justify-center">
            CLEAR LEAVE RECORDS
          </button>
        </div>

        <div class="flex flex-col sm:flex-row justify-between items-start sm:items-center pb-6 border-b border-slate-100 gap-4">
          <div>
            <h3 class="text-xs font-bold uppercase tracking-wider text-slate-950">RESET SCHOOL TIMETABLE</h3>
            <p class="text-slate-500 text-xs mt-1 font-medium">CLEAR ALL TIMETABLE DATA, MAKING IT EASIER TO IMPORT A NEW TIMETABLE.</p>
          </div>
          <button @click="clearOnlyTimetable" :disabled="loading" class="w-full sm:w-80 h-11 bg-orange-50 hover:bg-orange-100 text-orange-700 font-bold text-xs rounded-2xl transition shadow-sm border border-orange-200 disabled:opacity-50 shrink-0 cursor-pointer flex items-center justify-center">
            CLEAR SCHOOL TIMETABLE
          </button>
        </div>

        <div class="flex flex-col sm:flex-row justify-between items-start sm:items-center pb-6 border-b border-slate-100 gap-4">
          <div>
            <h3 class="text-xs font-bold uppercase tracking-wider text-indigo-900">BACKUP & CLEAR MMI INTERRUPTION HISTORY DATA</h3>
            <p class="text-slate-500 text-xs mt-1 font-medium">DOWNLOAD JSON BACKUP, THEN SAFELY DELETE ALL MMI INTERRUPTION LOGS.</p>
          </div>
          <button @click="backupAndClearMmi" :disabled="loading" class="w-full sm:w-80 h-11 bg-indigo-50 hover:bg-indigo-100 text-indigo-700 font-bold text-xs rounded-2xl transition shadow-sm border border-indigo-200 disabled:opacity-50 shrink-0 cursor-pointer flex items-center justify-center gap-2">
            <UploadCloud class="w-4 h-4 text-indigo-500" /> BACKUP & CLEAR MMI DATA
          </button>
        </div>

        <div class="flex flex-col sm:flex-row justify-between items-start sm:items-center gap-4">
          <div>
            <h3 class="text-xs font-bold uppercase tracking-wider text-red-600">DANGEROUS OPERATION: SYSTEM INITIALIZATION</h3>
            <p class="text-slate-500 text-xs mt-1 font-medium">CLEAR LEAVES, SUBSTITUTES, TIMETABLES, TEACHER LISTS, RESTORE TO INITIAL STATE.</p>
          </div>
          <button @click="clearEverything" :disabled="loading" class="w-full sm:w-80 h-11 bg-red-600 hover:bg-red-700 text-white font-bold text-xs rounded-2xl transition shadow-md disabled:opacity-50 shrink-0 cursor-pointer flex items-center justify-center">
            RESET SYSTEM COMPLETELY
          </button>
        </div>
      </div>
    </div>

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
            <span v-if="uploadProgress.percent < 100">📂</span>
            <span v-else>🎉</span>
          </div>

          <div>
            <h3 class="text-lg font-extrabold text-slate-900">
              {{ uploadProgress.percent === 100 ? 'TEMPLATE IMPORT SUCCESSFUL!' : 'IMPORTING TEMPLATE DATA...' }}
            </h3>
            <p class="text-xs font-semibold text-slate-500 mt-1.5">
              {{ uploadProgress.statusText }}
            </p>
          </div>

          <div class="space-y-2">
            <div class="w-full h-3.5 bg-slate-100 rounded-full overflow-hidden p-0.5 border border-slate-200/80 shadow-inner">
              <div 
                class="h-full bg-gradient-to-r from-indigo-500 via-purple-500 to-indigo-600 rounded-full transition-all duration-300 shadow-sm"
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
import { 
  Wrench, 
  UserCog, 
  Image, 
  Clock, 
  Calendar, 
  School, 
  BarChart3, 
  Database,
  ClipboardList,
  Download,
  UploadCloud,
  DownloadCloud
} from 'lucide-vue-next'

const toast = useToast()
const config = ref({ daysPerWeek: 5, periodsPerDay: 8 })
const loading = ref(false)
const fileInput = ref(null)
const weekFileInput = ref(null)

// --- User Management State ---
const newUserEmail = ref('')
const newUserPassword = ref('')
const creatingUser = ref(false)
const userList = ref([])

const fetchUsers = async () => {
  try {
    const { data, error } = await supabase.functions.invoke('create-user', {
      body: { action: 'list' }
    })
    if (error || data?.error) throw new Error(error?.message || data?.error)
    if (data?.users) {
      userList.value = data.users
    }
  } catch (err) {
    console.error("获取用户列表失败:", err.message)
  }
}

const handleCreateUser = async () => {
  if (!newUserEmail.value.trim() || !newUserPassword.value) {
    return toast.error("请完整填写邮箱与初始密码！")
  }

  creatingUser.value = true
  try {
    const { data, error } = await supabase.functions.invoke('create-user', {
      body: { 
        action: 'create',
        email: newUserEmail.value.trim(), 
        password: newUserPassword.value 
      }
    })

    if (error || data?.error) {
      throw new Error(error?.message || data?.error)
    }

    toast.success("新用户创建成功！")
    newUserEmail.value = ''
    newUserPassword.value = ''
    fetchUsers()
  } catch (err) {
    toast.error("创建用户失败: " + err.message)
  } finally {
    creatingUser.value = false
  }
}

const handleDeleteUser = async (userId, email) => {
  if (!confirm(`⚠️ 确定要删除管理员账号 ${email} 吗？`)) return

  try {
    const { data, error } = await supabase.functions.invoke('create-user', {
      body: { 
        action: 'delete',
        userId 
      }
    })

    if (error || data?.error) {
      throw new Error(error?.message || data?.error)
    }

    toast.success("用户已成功删除")
    fetchUsers()
  } catch (err) {
    toast.error("删除用户失败: " + err.message)
  }
}

// 📊 Upload percentage progress bar state
const uploadProgress = ref({
  show: false,
  percent: 0,
  statusText: ''
})

const sleep = (ms) => new Promise(resolve => setTimeout(resolve, ms))

const startProgress = (initialText = 'READING TEMPLATE FILE...') => {
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

// Class management state
const classList = ref([])
const newClassGrade = ref(1)
const newClassName = ref('')

const expandedGrades = ref({ 1: true, 2: true, 3: true, 4: true, 5: true, 6: true })
const allExpanded = computed(() => Object.values(expandedGrades.value).every(v => v))

// School weeks state
const allSchoolWeeks = ref([])
const newWeek = ref({ week_number: 1, start_date: '', end_date: '', is_school_week: true })

const toggleGrade = (grade) => {
  expandedGrades.value[grade] = !expandedGrades.value[grade]
}

const toggleAllGrades = () => {
  const target = !allExpanded.value
  Object.keys(expandedGrades.value).forEach(k => {
    expandedGrades.value[k] = target
  })
}

const groupedClasses = computed(() => {
  const groups = { 1: [], 2: [], 3: [], 4: [], 5: [], 6: [] }
  classList.value.forEach(c => {
    const g = c.grade || Number(c.class_name[0]) || 1
    if (groups[g]) {
      groups[g].push(c)
    } else {
      if (!groups['Other']) groups['Other'] = []
      groups['Other'].push(c)
    }
  })
  return groups
})

onMounted(() => {
  const saved = localStorage.getItem('school_config')
  if (saved) {
    config.value = JSON.parse(saved)
  }
  fetchClasses()
  fetchSchoolWeeks()
  fetchSchoolIdentity()
  fetchUsers() // Automatically load user list
})

const saveConfig = () => {
  if (config.value.daysPerWeek < 1 || config.value.daysPerWeek > 7) return toast.error("NUMBER OF DAYS MUST BE BETWEEN 1 AND 7")
  if (config.value.periodsPerDay < 1 || config.value.periodsPerDay > 15) return toast.error("NUMBER OF TIME SLOTS MUST BE BETWEEN 1 AND 15")
  
  localStorage.setItem('school_config', JSON.stringify(config.value))
  toast.success("SYSTEM CONFIGURATION SAVED SUCCESSFULLY!")
}

const fetchClasses = async () => {
  const { data, error } = await supabase
    .from('classes')
    .select('*')
    .order('grade', { ascending: true })
    .order('class_name', { ascending: true })
  
  if (data) classList.value = data
  if (error) console.error(error)
}

const fetchSchoolWeeks = async () => {
  const { data, error } = await supabase
    .from('school_weeks')
    .select('*')
    .order('week_number', { ascending: true })
  
  if (data) allSchoolWeeks.value = data
  if (error) console.error(error)
}

const addClass = async () => {
  if (!newClassName.value.trim()) {
    return toast.error("PLEASE ENTER A CLASS NAME!")
  }

  try {
    const { error } = await supabase.from('classes').insert({
      grade: newClassGrade.value,
      class_name: newClassName.value.trim().toUpperCase()
    })

    if (error) throw error

    toast.success("CLASS ADDED SUCCESSFULLY!")
    newClassName.value = ''
    fetchClasses()
  } catch (err) {
    toast.error("FAILED TO ADD (CLASS NAME MAY ALREADY EXIST): " + err.message)
  }
}

const deleteClass = async (id) => {
  try {
    const { error } = await supabase.from('classes').delete().eq('id', id)
    if (error) throw error
    toast.success("CLASS DELETED")
    fetchClasses()
  } catch (err) {
    toast.error("FAILED TO DELETE: " + err.message)
  }
}

const addSchoolWeek = async () => {
  if (!newWeek.value.start_date || !newWeek.value.end_date) {
    return toast.error("PLEASE FILL IN START AND END DATES COMPLETELY!")
  }
  try {
    const { error } = await supabase.from('school_weeks').insert([newWeek.value])
    if (error) throw error
    toast.success("SCHOOL CALENDAR ADDED SUCCESSFULLY!")
    newWeek.value.week_number += 1 
    newWeek.value.start_date = ''
    newWeek.value.end_date = ''
    fetchSchoolWeeks()
  } catch (err) {
    toast.error("FAILED TO ADD: " + err.message)
  }
}

const deleteSchoolWeek = async (id) => {
  try {
    const { error } = await supabase.from('school_weeks').delete().eq('id', id)
    if (error) throw error
    toast.success("THIS WEEK'S RECORD HAS BEEN DELETED")
    fetchSchoolWeeks()
  } catch (err) {
    toast.error("FAILED TO DELETE: " + err.message)
  }
}

const downloadWeekTemplate = () => {
  const csvContent = "week_number,start_date,end_date,is_school_week\n" +
                     "1,12/01/2026,16/01/2026,TRUE\n" +
                     "2,19/01/2026,23/01/2026,TRUE\n" +
                     "3,26/01/2026,30/01/2026,FALSE"
  
  const blob = new Blob(["\uFEFF" + csvContent], { type: 'text/csv;charset=utf-8;' })
  const url = URL.createObjectURL(blob)
  const link = document.createElement('a')
  link.setAttribute('href', url)
  link.setAttribute('download', 'SCHOOL_CALENDAR_TEMPLATE.csv')
  document.body.appendChild(link)
  link.click()
  document.body.removeChild(link)
  toast.success("CALENDAR TEMPLATE DOWNLOADED SUCCESSFULLY!")
}

const handleWeekFileUpload = async (event) => {
  const file = event.target.files[0]
  if (!file) return

  startProgress('READING CALENDAR TEMPLATE CSV FILE...')

  const reader = new FileReader()
  reader.onload = async (e) => {
    try {
      await sleep(250)
      updateProgress(35, 'VALIDATING DATE FORMATS AND DATA VALIDITY...')

      const text = e.target.result
      const lines = text.split('\n')
      const rows = []

      const parseDate = (dateStr) => {
        dateStr = dateStr.trim()
        if (dateStr.includes('/')) {
          const parts = dateStr.split('/')
          if (parts.length === 3) {
            return `${parts[2]}-${parts[1].padStart(2, '0')}-${parts[0].padStart(2, '0')}`
          }
        }
        return dateStr
      }

      for (let i = 1; i < lines.length; i++) {
        const line = lines[i].trim()
        if (!line) continue
        const cols = line.split(',')
        if (cols.length >= 4) {
          const rawBool = cols[3].trim().toUpperCase()
          rows.push({
            week_number: Number(cols[0].trim()),
            start_date: parseDate(cols[1]),
            end_date: parseDate(cols[2]),
            is_school_week: rawBool === 'TRUE' || rawBool === '1' || rawBool === 'YES'
          })
        }
      }

      if (rows.length === 0) {
        uploadProgress.value.show = false
        toast.error("FILE IS EMPTY OR FORMAT IS INCORRECT!")
        return
      }

      await sleep(250)
      updateProgress(70, `WRITING ${rows.length} CALENDAR RECORDS TO DATABASE...`)

      const { error } = await supabase.from('school_weeks').insert(rows)
      if (error) throw error

      await finishProgress(`SUCCESSFULLY IMPORTED ${rows.length} CALENDAR WEEKS!`)

      toast.success(`BULK IMPORTED ${rows.length} CALENDAR RECORDS SUCCESSFULLY!`)
      if (weekFileInput.value) weekFileInput.value.value = ''
      fetchSchoolWeeks()
    } catch (err) {
      uploadProgress.value.show = false
      toast.error("CALENDAR IMPORT FAILED: " + err.message)
    }
  }
  reader.readAsText(file)
}

const downloadTemplate = () => {
  const csvContent = "grade,subject_name,planned_periods,kpm_min_hours\n" +
                     "3,BAHASA MELAYU,80,60\n" +
                     "3,MATEMATIK,75,60\n" +
                     "4,SAINS,40,40"
  
  const blob = new Blob(["\uFEFF" + csvContent], { type: 'text/csv;charset=utf-8;' })
  const url = URL.createObjectURL(blob)
  const link = document.createElement('a')
  link.setAttribute('href', url)
  link.setAttribute('download', 'SUBJECT_TARGETS_TEMPLATE.csv')
  document.body.appendChild(link)
  link.click()
  document.body.removeChild(link)
  toast.success("TARGETS TEMPLATE DOWNLOADED SUCCESSFULLY!")
}

const handleFileUpload = async (event) => {
  const file = event.target.files[0]
  if (!file) return

  startProgress('READING SUBJECT TARGETS TEMPLATE CSV FILE...')

  const reader = new FileReader()
  reader.onload = async (e) => {
    try {
      await sleep(250)
      updateProgress(40, 'PARSING GRADE SUBJECT TARGETS...')

      const text = e.target.result
      const lines = text.split('\n')
      const rows = []

      for (let i = 1; i < lines.length; i++) {
        const line = lines[i].trim()
        if (!line) continue
        const cols = line.split(',')
        if (cols.length >= 4) {
          rows.push({
            grade: Number(cols[0].trim()),
            subject_name: cols[1].trim(),
            planned_periods: Number(cols[2].trim()),
            kpm_min_hours: Number(cols[3].trim())
          })
        }
      }

      if (rows.length === 0) {
        uploadProgress.value.show = false
        toast.error("FILE IS EMPTY OR FORMAT IS INCORRECT!")
        return
      }

      await sleep(250)
      updateProgress(75, `WRITING ${rows.length} SUBJECT TARGETS TO DATABASE...`)

      const { error } = await supabase.from('subject_targets').insert(rows)
      if (error) throw error

      await finishProgress(`SUCCESSFULLY IMPORTED ${rows.length} SUBJECT TARGETS!`)

      toast.success(`BULK IMPORTED ${rows.length} SUBJECT TARGET RECORDS SUCCESSFULLY!`)
      if (fileInput.value) fileInput.value.value = ''
    } catch (err) {
      uploadProgress.value.show = false
      toast.error("IMPORT FAILED: " + err.message)
    }
  }
  reader.readAsText(file)
}

const clearOnlyRecords = async () => {
  if (!confirm("⚠️ ARE YOU SURE YOU WANT TO CLEAR ALL LEAVE & SUBSTITUTE RECORDS? THIS ACTION CANNOT BE UNDONE!")) return

  loading.value = true
  try {
    const { error: err1 } = await supabase.from('substitute_assignments').delete().not('id', 'is', null)
    if (err1) throw err1

    const { error: err2 } = await supabase.from('leave_requests').delete().not('id', 'is', null)
    if (err2) throw err2

    toast.success("ALL LEAVE & SUBSTITUTE RECORDS CLEARED SUCCESSFULLY!")
  } catch (error) {
    toast.error("CLEARING FAILED: " + error.message)
  } finally {
    loading.value = false
  }
}

const clearOnlyTimetable = async () => {
  if (!confirm("⚠️ ARE YOU SURE YOU WANT TO CLEAR THE SCHOOL TIMETABLE? TEACHER LISTS WILL BE PRESERVED.")) return

  loading.value = true
  try {
    const { error } = await supabase.from('timetable').delete().not('id', 'is', null)
    if (error) throw error

    toast.success("SCHOOL TIMETABLE HAS BEEN RESET!")
  } catch (error) {
    toast.error("TIMETABLE CLEARING FAILED: " + error.message)
  } finally {
    loading.value = false
  }
}

const backupAndClearMmi = async () => {
  if (!confirm("📥 ARE YOU SURE YOU WANT TO BACKUP AND CLEAR ALL MMI INTERRUPTION HISTORY DATA?")) return

  loading.value = true
  try {
    const { data: mmiData, error: fetchErr } = await supabase.from('mmi_interruptions').select('*')
    if (fetchErr) throw fetchErr

    if (mmiData && mmiData.length > 0) {
      const dataStr = "data:text/json;charset=utf-8," + encodeURIComponent(JSON.stringify(mmiData, null, 2))
      const downloadAnchor = document.createElement('a')
      downloadAnchor.setAttribute("href", dataStr)
      downloadAnchor.setAttribute("download", `MMI_Backup_${new Date().toISOString().slice(0, 10)}.json`)
      document.body.appendChild(downloadAnchor)
      downloadAnchor.click()
      downloadAnchor.remove()
    } else {
      toast.success("NO MMI INTERRUPTION DATA TO BACKUP. PROCEEDING TO CLEAR.")
    }

    const { error: deleteErr } = await supabase.from('mmi_interruptions').delete().not('id', 'is', null)
    if (deleteErr) throw deleteErr

    toast.success("MMI INTERRUPTION DATA BACKED UP AND CLEARED FROM DATABASE SUCCESSFULLY!")
  } catch (error) {
    toast.error("MMI BACKUP OR CLEAR FAILED: " + error.message)
  } finally {
    loading.value = false
  }
}

const clearEverything = async () => {
  if (!confirm("🚨 WARNING: THIS ACTION WILL DELETE ALL TEACHERS, TIMETABLES, CLASSES AND LEAVE RECORDS IN THE SYSTEM! ARE YOU SURE YOU WANT TO RESET TO THE INITIAL STATE?")) return

  loading.value = true
  try {
    await supabase.from('substitute_assignments').delete().not('id', 'is', null)
    await supabase.from('leave_requests').delete().not('id', 'is', null)
    await supabase.from('timetable').delete().not('id', 'is', null)
    await supabase.from('mmi_interruptions').delete().not('id', 'is', null)
    const { error } = await supabase.from('teachers').delete().not('id', 'is', null)
    if (error) throw error

    toast.success("SYSTEM HAS BEEN COMPLETELY RESET TO ITS INITIAL STATE!")
    fetchClasses()
    fetchSchoolWeeks()
  } catch (error) {
    toast.error("RESET FAILED: " + error.message)
  } finally {
    loading.value = false
  }
}

// --- School Name & Logo Settings State ---
const schoolNameSetting = ref('')
const schoolLogoSetting = ref('')

const fetchSchoolIdentity = async () => {
  try {
    const { data } = await supabase.from('school_settings').select('*').eq('id', 1).single()
    if (data) {
      schoolNameSetting.value = data.school_name || ''
      schoolLogoSetting.value = data.logo_url || ''
    }
  } catch (err) {
    schoolNameSetting.value = localStorage.getItem('school_name') || ''
    schoolLogoSetting.value = localStorage.getItem('school_logo') || ''
  }
}

const handleSettingsLogoUpload = (event) => {
  const file = event.target.files[0]
  if (!file) return

  if (file.size > 2 * 1024 * 1024) {
    return toast.error("LOGO IMAGE SIZE IS TOO LARGE, PLEASE CHOOSE UNDER 2MB")
  }

  const reader = new FileReader()
  reader.onload = (e) => {
    schoolLogoSetting.value = e.target.result
    toast.success("NEW LOGO READ SUCCESSFULLY, PLEASE CLICK SAVE BELOW!")
  }
  reader.readAsDataURL(file)
}

const saveSchoolIdentity = async () => {
  try {
    const { error } = await supabase
      .from('school_settings')
      .update({
        school_name: schoolNameSetting.value.trim(),
        logo_url: schoolLogoSetting.value
      })
      .eq('id', 1)

    if (error) throw error

    localStorage.setItem('school_logo', schoolLogoSetting.value)
    localStorage.setItem('school_name', schoolNameSetting.value.trim())

    toast.success("SCHOOL LOGO & NAME UPDATED SUCCESSFULLY! SYSTEM HAS BEEN REFRESHED.")

    setTimeout(() => {
      window.location.reload()
    }, 600)
  } catch (err) {
    toast.error("FAILED TO SAVE: " + err.message)
  }
}

// 📦 Export Full System Backup (Supabase Cloud Version)
const exportFullBackup = async () => {
  try {
    toast.success("PREPARING CLOUD DATA BACKUP...")
    
    const [teachers, classes, schoolWeeks, subjectTargets, timetable, leaveRequests, substituteAssignments, schoolSettings] = await Promise.all([
      supabase.from('teachers').select('*'),
      supabase.from('classes').select('*'),
      supabase.from('school_weeks').select('*'),
      supabase.from('subject_targets').select('*'),
      supabase.from('timetable').select('*'),
      supabase.from('leave_requests').select('*'),
      supabase.from('substitute_assignments').select('*'),
      supabase.from('school_settings').select('*')
    ])

    const backupPackage = {
      version: "2.0-supabase",
      exportDate: new Date().toISOString(),
      data: {
        teachers: teachers.data || [],
        classes: classes.data || [],
        schoolWeeks: schoolWeeks.data || [],
        subjectTargets: subjectTargets.data || [],
        timetable: timetable.data || [],
        leaveRequests: leaveRequests.data || [],
        substituteAssignments: substituteAssignments.data || [],
        schoolSettings: schoolSettings.data || []
      }
    }

    const dataStr = "data:text/json;charset=utf-8," + encodeURIComponent(JSON.stringify(backupPackage, null, 2))
    const downloadAnchor = document.createElement('a')
    downloadAnchor.setAttribute("href", dataStr)
    downloadAnchor.setAttribute("download", `SJKC_Supabase_Backup_${new Date().toISOString().slice(0, 10)}.json`)
    document.body.appendChild(downloadAnchor)
    downloadAnchor.click()
    downloadAnchor.remove()

    toast.success("EXPORT FULL SYSTEM BACKUP SUCCESSFUL!")
  } catch (err) {
    toast.error("BACKUP EXPORT FAILED: " + err.message)
  }
}

// 📥 Restore Full System Backup (WebApp Cloud Version - Compatible with Local & Cloud JSON)
const importFullBackup = async (event) => {
  const file = event.target.files[0]
  if (!file) return

  if (!confirm("⚠️ WARNING: RESTORING BACKUP WILL OVERWRITE CURRENT DATA IN THE CLOUD DATABASE! ARE YOU SURE?")) {
    event.target.value = ''
    return
  }

  toast.info("READING AND VALIDATING BACKUP FILE...")
  const reader = new FileReader()
  
  reader.onload = async (e) => {
    try {
      const raw = JSON.parse(e.target.result)
      let d = {}

      if (raw.data) {
        d = raw.data 
      } else if (typeof raw === 'object' && !Array.isArray(raw)) {
        d = raw      
      } else {
        throw new Error("UNSUPPORTED FILE FORMAT")
      }
      
      if (Array.isArray(d.schoolSettings) && d.schoolSettings.length > 0) {
        await supabase.from('school_settings').upsert(d.schoolSettings)
      }
      
      if (Array.isArray(d.classes) && d.classes.length > 0) {
        await supabase.from('classes').upsert(d.classes)
      }

      if (Array.isArray(d.teachers) && d.teachers.length > 0) {
        const cleanTeachers = d.teachers.map(t => ({
          id: t.id,
          name: t.name,
          max_substitute_per_week: t.max_substitute_per_week || t.max_lessons_per_day || 6,
          is_active: t.is_active ?? true,
          session: t.session || 'morning'
        }))
        await supabase.from('teachers').upsert(cleanTeachers)
      }

      if (Array.isArray(d.schoolWeeks) && d.schoolWeeks.length > 0) {
        const cleanWeeks = d.schoolWeeks.map(w => ({
          ...w,
          is_school_week: (w.is_school_week === true || w.is_school_week === 1 || w.is_school_week === 'true')
        }))
        await supabase.from('school_weeks').upsert(cleanWeeks)
      }

      if (Array.isArray(d.subjectTargets) && d.subjectTargets.length > 0) {
        await supabase.from('subject_targets').upsert(d.subjectTargets)
      }

      if (Array.isArray(d.timetable) && d.timetable.length > 0) {
        const cleanTimetable = d.timetable.map(tt => ({
          id: tt.id,
          teacher_id: tt.teacher_id,
          class_name: tt.class_name || tt.class_id, 
          subject: tt.subject || tt.subject_name,
          weekday: tt.weekday || tt.day_of_week,
          period: tt.period || tt.period_number
        }))
        await supabase.from('timetable').upsert(cleanTimetable)
      }

      if (Array.isArray(d.leaveRequests) && d.leaveRequests.length > 0) {
        await supabase.from('leave_requests').upsert(d.leaveRequests)
      }
      if (Array.isArray(d.substituteAssignments) && d.substituteAssignments.length > 0) {
        await supabase.from('substitute_assignments').upsert(d.substituteAssignments)
      }
      if (Array.isArray(d.mmiInterruptions) && d.mmiInterruptions.length > 0) {
        await supabase.from('mmi_interruptions').upsert(d.mmiInterruptions)
      }

      toast.success("SYSTEM DATA RESTORED SUCCESSFULLY! PAGE WILL REFRESH...")
      setTimeout(() => {
        window.location.reload()
      }, 1000)
      
    } catch (err) {
      console.error("IMPORT ERROR:", err)
      toast.error("RESTORE FAILED: " + err.message)
    } finally {
      event.target.value = '' 
    }
  }
  
  reader.readAsText(file)
}
</script>