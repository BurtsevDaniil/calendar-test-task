<script setup lang="ts">
import { computed, ref } from 'vue';
import LeftArrow from "../assets/left-arrow.svg"
import RightArrow from "../assets/right-arrow.svg"

interface CalendarDay {
    date?: Date;
    isToday: boolean;
    isSelected: boolean;
    isPlaceholder?: boolean
}

const today = new Date()
const locale = navigator.language
const currentMonth = ref(today.getMonth());
const currentYear = ref(today.getFullYear());
const selectedDate = ref<Date>();

//Получение названий месяцев по локали
function getMonthNames(locale: string) {
    const formatter = new Intl.DateTimeFormat(locale, { month: 'short' });
    return Array.from({ length: 12 }, (_, i) =>
        formatter.format(new Date(currentYear.value, i, 1))
    );
}

//Получение названий дней по локали
function getWeekDayNames(locale: string, weekStart: number = 0) {
    const formatter = new Intl.DateTimeFormat(locale, { weekday: 'short' });
    const todayWeekDay = today.getDay();
    const diff = todayWeekDay - weekStart;
    const weekStartDate = new Date();
    weekStartDate.setDate(today.getDate() - diff);
    const days = [];

    for (let i = 0; i < 7; i++) {
        const day = new Date(weekStartDate)
        day.setDate(weekStartDate.getDate() + ((i + weekStart) % 7));
        days.push(formatter.format(day));
    }
    return days;
}

const weekDays = getWeekDayNames(locale)
const monthNames = getMonthNames(locale)

const calendarDays = computed(() => generateCalendar());

//Вспомогательная функция првоерки, являются ли даты одинаковыми (нужно для сверки текущей даты для отображения)
const checkToday = (date1: Date, date2: Date) => {
    return date1.getFullYear() === date2.getFullYear() &&
        date1.getMonth() === date2.getMonth() &&
        date1.getDate() === date2.getDate()
}

const generateCalendar = () => {
    const days: CalendarDay[] = []
    const firstDayOfMonth = new Date(currentYear.value, currentMonth.value, 1);
    const lastDayOfMonth = new Date(currentYear.value, currentMonth.value + 1, 0);

    //Нужно для правильного отображения первого дня недели
    for (let i = 0; i < firstDayOfMonth.getDay(); i++) {
        days.push({
            date: undefined,
            isToday: false,
            isSelected: false,
            isPlaceholder: true,
        });
    }

    for (let i = 1; i < lastDayOfMonth.getDate() + 1; i++) {
        const date = new Date(currentYear.value, currentMonth.value, i)
        days.push({
            date: date,
            isToday: checkToday(date, today),
            isSelected: !!selectedDate.value && checkToday(selectedDate.value, date),
        })
    }
    return days
}

const selectDate = (day: CalendarDay) => {
    selectedDate.value = day.date
}

const prevMonth = () => {
    const isPrevYear = currentMonth.value - 1 < 0 ? true : false
    currentMonth.value = isPrevYear ? 11 : currentMonth.value - 1
    if (isPrevYear) {
        currentYear.value -= 1
    }
}

const nextMonth = () => {
    const isNextYear = currentMonth.value + 1 > 11 ? true : false
    currentMonth.value = isNextYear ? 0 : currentMonth.value + 1
    if (isNextYear) {
        currentYear.value += 1
    }
}
</script>
<template>
    <main>
        <div class="calendar-body">
            <div class="calendar-header">
                <button class="custom-button prev" @click="prevMonth()"
                    :style="{ backgroundImage: `url(${LeftArrow})` }"></button>
                <span>{{ monthNames[currentMonth] }} {{ currentYear }}</span>
                <button class="custom-button next" @click="nextMonth()"></button>
            </div>
            <div class="calendar-weekdays">
                <div v-for="wday in weekDays" :key="wday">{{ wday }}</div>
            </div>
            <div class="calendar-dates">
                <div class="calendar-date" v-for="(day, i) in calendarDays" :key="i" :class="{
                    today: day.isToday,
                    selected: day.isSelected
                }" @click="selectDate(day)">
                    {{ day.date?.getDate() }}
                </div>
            </div>

        </div>
    </main>
</template>
<style scoped>
.calendar-body {
    width: 100%;
    max-width: 300px;
    height: 100%;
    max-height: 300px;
    display: flex;
    flex-direction: column;
    gap: 10px;
    padding: 5px;
    border: 1px solid black;
}

.calendar-header {
    display: flex;
    flex-direction: row;
    justify-content: space-between;
}

.calendar-weekdays {
    display: grid;
    grid-template-columns: repeat(7, 1fr);
    text-align: center;
}

.calendar-dates {
    display: grid;
    grid-template-columns: repeat(7, 1fr);
    align-items: center;
}

.calendar-date {
    box-sizing: border-box;
    text-align: center;
    cursor: pointer;
    padding: 5px;
    position: relative;
}

.today::after {
    box-sizing: border-box;
    content: "";
    position: absolute;
    width: 100%;
    height: 100%;
    top: 0;
    left: 0;
    border: lightblue 1px solid;
}

.selected {
    border: none;
    background-color: lightcoral;
}

.selected::after {
    display: none;
}

.custom-button {
    background-color: transparent;
    border: none;
    background-size: contain;
    background-repeat: no-repeat;
    background-position: center;
    padding: 0;
    width: 20px;
    height: 20px;
    cursor: pointer;
}

.custom-button.prev {
    background-image: url("@/assets/left-arrow.svg");
}

.custom-button.next {
    background-image: url("@/assets/right-arrow.svg");
}
</style>