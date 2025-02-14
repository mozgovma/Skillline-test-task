<template>
  <div class="custom-datepicker">
    <input 
      type="text" 
      v-model="selectedRange" 
      @focus="showCalendar = true" 
      @click="showCalendar = true" 
      placeholder="Выберите диапазон дат"
      readonly
    />
    <div 
      class="calendar-popup" 
      :class="{ show: showCalendar }"
      @click.stop
    >
      <div class="calendar">
        <div class="select-period">Выбрать период</div>
        <div class="calendar-header">
          <span class="prev" @click="changeMonth(-1)">&#8249;</span>
          <span>{{ monthNames[month] }} {{ year }}</span>
          <span class="next" @click="changeMonth(1)">&#8250;</span>
        </div>
        <div class="calendar-days-of-week">
          <div v-for="(day, index) in weekDays" :key="index" class="calendar-day-of-week">
            {{ day }}
          </div>
        </div>
        <div class="calendar-days">
          <div v-for="day in daysInMonth" :key="day" 
               class="calendar-day"
               :class="{
                 'selected': isSelectedDay(day),
                 'in-range': isInRange(day),
                 'start-range': isStartDay(day),
                 'end-range': isEndDay(day)
               }"
               @click="selectDate(day)">
            {{ day }}
          </div>
        </div>
        <div class="calendar-footer">
          <button class="cancel-btn" @click="cancelSelection">Отмена</button>
          <button @click="saveSelection">Сохранить</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, computed } from 'vue';

const selectedRange = ref('');
const showCalendar = ref(false);
const currentDate = new Date();
const month = ref(currentDate.getMonth());
const year = ref(currentDate.getFullYear());

const monthNames = [
  'Январь', 'Февраль', 'Март', 'Апрель', 'Май', 'Июнь',
  'Июль', 'Август', 'Сентябрь', 'Октябрь', 'Ноябрь', 'Декабрь'
];

const weekDays = ['Пн', 'Вт', 'Ср', 'Чт', 'Пт', 'Сб', 'Вс'];

const startDate = ref(null);
const endDate = ref(null);

const previousMonthDays = ref([]);
const nextMonthDays = ref([]);
const daysInMonth = ref([]);

const getDaysInMonth = (month, year) => new Date(year, month + 1, 0).getDate();

const getFirstDayOfWeek = (month, year) => {
  const firstDay = new Date(year, month, 1).getDay();
  return firstDay === 0 ? 7 : firstDay;
};

const updateCalendar = () => {
  const daysInCurrentMonth = Array.from({ length: getDaysInMonth(month.value, year.value) }, (_, i) => i + 1);
  const firstDayOfMonth = getFirstDayOfWeek(month.value, year.value);

  const prevMonthDaysCount = firstDayOfMonth - 1;
  const prevMonthLastDay = new Date(year.value, month.value, 0).getDate();
  previousMonthDays.value = Array.from({ length: prevMonthDaysCount }, (_, i) => prevMonthLastDay - prevMonthDaysCount + i + 1);

  const totalDays = daysInCurrentMonth.length + prevMonthDaysCount;
  const nextMonthDaysCount = totalDays % 7 === 0 ? 0 : 7 - totalDays % 7;
  nextMonthDays.value = Array.from({ length: nextMonthDaysCount }, (_, i) => i + 1);

  daysInMonth.value = daysInCurrentMonth;
};

const changeMonth = (direction) => {
  month.value = (month.value + direction + 12) % 12;
  year.value += direction === -1 && month.value === 11 ? -1 : direction === 1 && month.value === 0 ? 1 : 0;
  updateCalendar();
};

const formatDate = (date) => {
  if (!date) return '';
  const { day, month, year } = date;
  return `${String(day).padStart(2, '0')} ${monthNames[month]} ${year}`;
};

const isValidDay = (day) => day > 0 && day <= getDaysInMonth(month.value, year.value);

const selectDate = (day) => {
  const selectedDate = { day, month: month.value, year: year.value };


  if (!startDate.value) {
    startDate.value = selectedDate;
  } else if (!endDate.value) {

    endDate.value = selectedDate;


    const start = new Date(startDate.value.year, startDate.value.month, startDate.value.day);
    const end = new Date(endDate.value.year, endDate.value.month, endDate.value.day);
    if (end < start) {
      [startDate.value, endDate.value] = [endDate.value, startDate.value];
    }
  } else {

    startDate.value = selectedDate;
    endDate.value = null;
  }

  logSelection();
  saveToLocalStorage();
};

const logSelection = () => console.log("Выбранный диапазон дат:", selectedRange.value);

const saveToLocalStorage = () => {
  localStorage.setItem('selectedRange', selectedRange.value);
  localStorage.setItem('startDate', JSON.stringify(startDate.value));
  localStorage.setItem('endDate', JSON.stringify(endDate.value));
};

const cancelSelection = () => {
  startDate.value = null;
  endDate.value = null;
  showCalendar.value = false;
  selectedRange.value = null;
  logSelection();
  saveToLocalStorage();
};

const saveSelection = () => {
  if (startDate.value && endDate.value) {
    selectedRange.value = `${formatDate(startDate.value)} - ${formatDate(endDate.value)}`;
    showCalendar.value = false;
    logSelection();
    saveToLocalStorage();
  } else {
    alert('Пожалуйста, выберите диапазон дат');
  }
};

onMounted(() => {
  updateCalendar();

  const storedRange = localStorage.getItem('selectedRange');
  const storedStartDate = localStorage.getItem('startDate');
  const storedEndDate = localStorage.getItem('endDate');

  if (storedRange) selectedRange.value = storedRange;
  if (storedStartDate) startDate.value = JSON.parse(storedStartDate);
  if (storedEndDate) endDate.value = JSON.parse(storedEndDate);

  logSelection();
});

const isSelectedDay = (day) => {
  if (!startDate.value && !endDate.value) return false;

  const currentDate = new Date(year.value, month.value, day);

  if (startDate.value) {
    const start = new Date(startDate.value.year, startDate.value.month, startDate.value.day);
    if (currentDate.getTime() === start.getTime()) return true;
  }

  if (endDate.value) {
    const end = new Date(endDate.value.year, endDate.value.month, endDate.value.day);
    if (currentDate.getTime() === end.getTime()) return true;
  }

  return false;
};

const isInRange = (day) => {
  if (!startDate.value || !endDate.value) return false;

  const currentDate = new Date(year.value, month.value, day);
  const start = new Date(startDate.value.year, startDate.value.month, startDate.value.day);
  const end = new Date(endDate.value.year, endDate.value.month, endDate.value.day);

  return currentDate >= start && currentDate <= end;
};

const isStartDay = (day) => day === startDate.value?.day;

const isEndDay = (day) => day === endDate.value?.day;
</script>




<style lang="scss" scoped>
@use '@/assets/scss/_fonts.scss' as *;

@media (max-width:1024px){
  input {
    width: 100% !important;
    height: 56px;
    padding-left: 15px;
    border-radius: 10px;
    border: 1px solid #D3D3DE;
    font-size: 16px;
    font-family: Gotham Pro;
    font-weight: 400;
    font-size: 16px;
    line-height: 20.8px;
    letter-spacing: 0px;

  }
  .calendar-popup.show {
    display: block;
    opacity: 1;
    visibility: visible;
    width: 100% !important;
    border-radius: 16px;
  }
}

@media (max-width: 1624px) {
  
  input {
    width: 300px !important;
    height: 56px;
    padding-left: 15px;
    border-radius: 10px;
    border: 1px solid #D3D3DE;
    font-size: 16px;
    font-family: Gotham Pro;
    font-weight: 400;
    font-size: 16px;
    line-height: 20.8px;
    letter-spacing: 0px;

  }
  .calendar-popup.show {
    display: block;
    opacity: 1;
    visibility: visible;
    width: 100% !important;
    border-radius: 16px;
  }
 
}

.custom-datepicker {
  position: relative;
  font-family: Gotham Pro;
  font-weight: 400;
  font-size: 16px;
  line-height: 20.8px;
  letter-spacing: 0px;

  

  input {
        width: 517px;
        height: 56px;
        padding-left: 15px;
        border-radius: 10px;
        border: 1px solid #D3D3DE;
        font-size: 16px;
        font-family: Gotham Pro;
        font-weight: 400;
        font-size: 16px;
        line-height: 20.8px;
        letter-spacing: 0px;

  }

  .calendar-popup {
    position: absolute;
    left: 0;
    background-color: white;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
    border-radius: 5px;
    width: 100%;
    display: none;
    z-index: 999;
    transition: opacity 0.3s ease, visibility 0.3s ease;
  }

  .calendar-popup.show {
    display: block;
    opacity: 1;
    visibility: visible;
    width: 70%;
    border-radius: 16px;
  }

  .calendar {
    width: 100%;
    text-align: center;
  }

  .calendar-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 10px;
    font-size: 18px;
    margin: 20px;
    font-family: 'Gotham Pro';
  }

  .calendar-days-of-week {
    display: grid;
    grid-template-columns: repeat(7, 1fr);
    gap: 5px;
    font-size: 14px;
    margin-bottom: 10px;
  }

  .calendar-day-of-week {
    font-weight: bold;
    color: #888;
    text-align: center;
    font-family: 'Gotham Pro';
  }

  .calendar-days {
    display: grid;
    grid-template-columns: repeat(7, 1fr);
    gap: 5px;
    font-size: 14px;
    font-family: 'Gotham Pro';
  }

  .calendar-day {
    padding: 10px;
    cursor: pointer;
    transition: background-color 0.3s;
  }

  .calendar-day:hover {
    background-color: #f0f0f0;
  }

  .calendar-day.selected {
    background-color: #33D35E;
    color: white;
  }

  .calendar-day.selected.in-range.start-range {
    background-color: #33D35E;
    color: black;
  }

  .calendar-day.selected.in-range.end-range {
    background-color: #33D35E;
    color: black;
  }

  .calendar-day.in-range {
    background-color: #C3FCD2;
    color: black;
  }

  .calendar-footer {
    margin-top: 15px;
    display: flex;
    justify-content: center;
    gap: 40px;
    margin: 10px;

    button {
      padding: 8px 15px;
      font-size: 14px;
      border: none;
      border-radius: 5px;
      cursor: pointer;
      background-color: #000000;
      color: white;
      font-family: 'Gotham Pro';
      width: 148px;
      height: 53px;
    }

    .cancel-btn{
      background-color: transparent !important;
      color: black;
      border: 1px solid black;
    }
  }

  .prev, .next {
    cursor: pointer;
    font-size: 20px;
  }
}
.select-period {
  font-size: 16px;
  font-weight: bold;
  margin: 30px;
  text-align: left;
  font-family: 'Gotham Pro';
}
</style>
