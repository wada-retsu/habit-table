<template>
  <div class="all-container">
    <div class="mode-toggle">
      <button @click="toggleMode">{{ isDailyMode ? '週' : '日' }}</button>
    </div>
    <div v-if="isDailyMode" class="daily-mode-container">
      <div class="category-column" v-for="(habits, cat) in dailyHabitsByCategory" :key="cat">
        <h3>{{ cat }}（合計: {{ dailyCategoryTotals[cat] }}pt）</h3>
        <div v-for="habit in habits" :key="habit.name">
          <span>{{ habit.name }}（{{ habit.point }}pt）</span>
          <button
            @click="toggleCheck(habit.name)"
            :class="['habit-cell', { checked: isChecked(habit.name) }]"
          >
            {{ isChecked(habit.name) ? habit.point : '' }}
          </button>
        </div>
      </div>
    </div>
    <div v-if="!isDailyMode" class="weekly-mode-container">
      <div v-if="!isDailyMode" class="week-nav">
        <button @click="prevWeek">←</button>
        <span>{{ format(currentWeekStart, 'yyyy年M月d日') }}週</span>
        <button @click="nextWeek">→</button>
      </div>
      <div v-if="!isDailyMode" class="month-nav">
        <button @click="prevMonth">←</button>
        <button @click="nextMonth">→</button>
      </div>
      <div style="min-width: 800px;">
        <table border="1">
          <thead>
            <tr>
              <th>習慣</th>
              <th>ポイント</th>
              <th v-for="day in weekDays" :key="day">
                {{ day }}
              </th>
            </tr>
            <tr>
              <th></th>
              <th></th>
              <th v-for="(date, i) in displayDates" :key="formattedDates[i]" class="date-cell" @click="selectDate(formattedDates[i])">
                {{ date }}
              </th>
            </tr>
          </thead>
          <tbody>
            <!-- 合計行 -->
            <tr>
              <td colspan="2"><strong>合計</strong></td>
              <td v-for="(total, i) in columnTotals" :key="i"><strong>{{ total }}</strong></td>
            </tr>
            <tr v-for="(habit, index) in habits" :key="index">
              <td>{{ habit.name }}</td>
              <td>{{ habit.point }}pt</td>
              <td v-for="(date, i) in formattedDates" :key="i">
                <button
                  @click="toggleCheck(habit.name, date)"
                  :class="['habit-cell', { checked: isChecked(habit.name, date) }]"
                >
                  {{ isChecked(habit.name, date) ? habit.point : '' }}
                </button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>
</template>

<script setup>
import { reactive, ref, computed, onMounted } from 'vue'
import { startOfWeek, addWeeks, addDays, format, eachDayOfInterval } from 'date-fns'
import ja from 'date-fns/locale/ja'

onMounted(() => {
  loadHabitsFromLocalStorage()
})

const currentWeekStart = ref(startOfWeek(new Date(), { weekStartsOn: 1 })) // 月曜始まり

const weekDates = computed(() =>
  eachDayOfInterval({
    start: currentWeekStart.value,
    end: addDays(currentWeekStart.value, 6)
  })
)

const formattedDates = computed(() =>
  weekDates.value.map(date => format(date, 'yyyy-MM-dd')) // ← Habitデータと同じ形式にする！
)

const weekDays = computed(() =>
  weekDates.value.map(date => format(date, 'EEE', { locale: ja }))
)

const prevWeek = () => {
  currentWeekStart.value = addWeeks(currentWeekStart.value, -1)
}
const nextWeek = () => {
  currentWeekStart.value = addWeeks(currentWeekStart.value, 1)
}
const prevMonth = () => {
  currentWeekStart.value = addWeeks(currentWeekStart.value, -4)
}
const nextMonth = () => {
  currentWeekStart.value = addWeeks(currentWeekStart.value, 4)
}

const habits = reactive([
  { name: 'カーテン', point: 3, checks: {}, category: '朝' },
  { name: 'お湯沸かす', point: 1, checks: {}, category: '朝' },
  { name: 'トイレ', point: 2, checks: {}, category: '朝' },
  { name: '体重測定', point: 3, checks: {}, category: '朝' },
  { name: '着替え', point: 3, checks: {}, category: '朝' },
  { name: '白湯', point: 2, checks: {}, category: '朝' },
  { name: '目標確認', point: 5, checks: {}, category: '朝' },
  { name: '瞑想', point: 5, checks: {}, category: '朝' },
  { name: '洗顔', point: 3, checks: {}, category: '朝' },
  { name: '朝食', point: 5, checks: {}, category: '朝' },
  { name: '朝薬', point: 3, checks: {}, category: '朝' },
  { name: '朝歯磨き', point: 3, checks: {}, category: '朝' },
  { name: 'タゲ友', point: 8, checks: {}, category: '朝' },
  { name: 'Duolingo', point: 8, checks: {}, category: '朝' },

  { name: '料理', point: 5, checks: {}, category: '夜' },
  { name: '夕食', point: 5, checks: {}, category: '夜' },
  { name: '夜薬', point: 3, checks: {}, category: '夜' },
  { name: '洗い物', point: 8, checks: {}, category: '夜' },
  { name: 'お風呂', point: 3, checks: {}, category: '夜' },
  { name: '肌ケア', point: 3, checks: {}, category: '夜' },
  { name: '夜歯磨き', point: 3, checks: {}, category: '夜' },
  { name: 'ピアノ', point: 8, checks: {}, category: '夜' },
  { name: '日記', point: 8, checks: {}, category: '夜' },

  { name: '昼食', point: 5, checks: {}, category: 'その他' },
  { name: '昼薬', point: 5, checks: {}, category: 'その他' },
  { name: '勉強', point: 20, checks: {}, category: 'その他' },
  { name: '読書', point: 15, checks: {}, category: 'その他' },
  { name: '作業', point: 20, checks: {}, category: 'その他' },
  { name: '筋トレ', point: 15, checks: {}, category: 'その他' },
  { name: '睡眠', point: 20, checks: {}, category: 'その他' },
  { name: '床', point: 25, checks: {}, category: 'その他' },
  { name: '机・棚', point: 25, checks: {}, category: 'その他' },
  { name: '水回り', point: 40, checks: {}, category: 'その他' },
  { name: '収納', point: 30, checks: {}, category: 'その他' },
  { name: '洗濯', point: 15, checks: {}, category: 'その他' },
  { name: 'ゴミ捨て', point: 15, checks: {}, category: 'その他' },
  { name: '買い物', point: 15, checks: {}, category: 'その他' },
  { name: '散歩', point: 20, checks: {}, category: 'その他' },
])

const isChecked = (habitName, date = selectedDate.value) => {
  const habit = habits.find(h => h.name === habitName)
  return habit.checks[date] || false
}

const toggleCheck = (habitName, date = selectedDate.value) => {
  const habit = habits.find(h => h.name === habitName)
  habit.checks[date] = !habit.checks[date]
  saveHabitsToLocalStorage()
}

const columnTotals = computed(() => {
  return formattedDates.value.map(date => {
    return habits.reduce((sum, habit) => {
      return sum + (habit.checks[date] ? habit.point : 0)
    }, 0)
  })
})

const isDailyMode = ref(false)
const selectedDate = ref(format(new Date(), 'yyyy-MM-dd'))

const toggleMode = () => {
  isDailyMode.value = !isDailyMode.value
}

const categoryTotal = (category) => {
  return habits
    .filter(h => h.category === category)
    .reduce((sum, h) => {
      return sum + (h.checks[selectedDate.value] ? h.point : 0)
    }, 0)
}

const today = computed(() => format(new Date(), 'yyyy-MM-dd'))

const dailyHabitsByCategory = computed(() => {
  const categories = ['朝', '夜', 'その他']
  const result = {}
  categories.forEach(cat => {
    result[cat] = habits.filter(h => h.category === cat)
  })
  return result
})

const dailyCategoryTotals = computed(() => {
  const result = {}
  Object.entries(dailyHabitsByCategory.value).forEach(([cat, habits]) => {
    result[cat] = habits.reduce((sum, h) => sum + (h.checks[selectedDate.value] ? h.point : 0), 0)
  })
  return result
})


const saveHabitsToLocalStorage = () => {
  localStorage.setItem('habitData', JSON.stringify(habits))
}

const loadHabitsFromLocalStorage = () => {
  const stored = localStorage.getItem('habitData')
  if (stored) {
    const parsed = JSON.parse(stored)
    habits.forEach((habit, i) => {
      habit.checks = parsed[i]?.checks || {}
    })
  }
}

const selectDate = (date) => {
  selectedDate.value = date
  isDailyMode.value = true
}

const displayDates = computed(() =>
  weekDates.value.map(date => format(date, 'M/d')) // 表示用
)

</script>

<style scoped>

.habit-cell {
  width: 100px;   /* ← ここ追加 */
  height: 40px;
  background: #222;
  color: white;
  border: none;
  cursor: pointer;
  padding: 0.5rem;
  box-sizing: border-box;
}

.habit-cell.checked {
  background: #0a0;
  color: white;
}

.week-nav,
.month-nav {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 1rem;
  margin-top: 1rem;
  max-width: 1200px;  /* ← ここも合わせる */
  margin-left: auto;
  margin-right: auto;
  flex-wrap: wrap;    /* ← 折り返し許可（狭い画面でも崩れない） */
}

.date-cell {
  padding: 0.5rem;
  width: 100px;        /* ← 固定幅を設定 */
  text-align: center;
  white-space: nowrap; /* ← 折り返し防止（任意） */
}

.daily-mode-container {
  display: flex;
  flex-direction: row;
  justify-content: flex-start; /* 左詰めにして自然な並び */
  align-items: flex-start;
  gap: 2rem;
  width: 100vw;
  height: 100vh;
  overflow-x: auto;
  padding: 1rem;
  box-sizing: border-box;
}

.category-column {
  min-width: 280px;
  padding: 1rem;
  border: 1px solid #ccc;
  border-radius: 12px;
  box-shadow: 0 2px 6px rgba(0,0,0,0.1);
}

.category-column h3 {
  margin-bottom: 0.5rem;
}

.category-column span {
  display: inline-block;
  width: 160px;
}

.weekly-mode-container {
  width: 100vw;
  height: 100vh;
  overflow-x: auto;
  overflow-y: auto; /* ← ここ重要 */
  display: flex;
  flex-direction: column; /* ← column に変更 */
}

.weekly-content {
  flex: 1;
  display: flex;
  flex-direction: column;
  padding: 1rem;
  box-sizing: border-box;
  min-width: 900px;
}

.all-container {
  padding: 0px;
}

button {
  background: transparent;
  color: #ccc;
}

</style>
