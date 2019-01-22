<template>
  <div class="calendar">
    <input
      type="text"
      v-model="date"
      @focus="focus"
      @blur="blur"
      placeholder="Pick a date"
    >
    <div
      class="calendar__layout"
      v-if="active"
      @mousedown.prevent
    >
      <div class="calendar__header">
        <div class="calendar__header-title">
          <i
            class="material-icons"
            @click="setPreviousMonth()"
          >
            chevron_left
          </i>
          <div class="calendar__title">
            {{ monthYearFormatted }}
          </div>
          <i
            class="material-icons"
            @click="setNextMonth()"
          >
            chevron_right
          </i>
        </div>
        <div class="calendar__header-days">
          <div
            class="calendar__day-name"
            v-for="day in days"
            :key="day"
          >
            {{ day | trimThreeChar }}
          </div>
        </div>
      </div>
      <div class="calendar__body">
        <div
          class="calendar__row"
          v-bind:class="{ 'swipe-left': swipeLeft, 'swipe-right': swipeRight }"
          v-for="(row, index) in calendarDays"
          :key="index"
        >
          <div
            class="calendar__day"
            :class="{'is-disabled': isDisabled(cell), 'is-selected': isSelected(cell)}"
            v-for="(cell, index) in row"
            :key="index"
            @click="selectDay(cell)"
          >
            {{ cell.day }}
          </div>
        </div>
      </div>
      <div class="calendar__footer">
      </div>
    </div>
  </div>
</template>
<script>
export default {
  props: {
    colors: {
      type: [],
      default: null
    },
    start_month: {
      type: Number,
      default: 1
    },
    start_year: {
      type: Number,
      default: 2019
    },
    start_day: {
      type: Number,
      default: null
    },
    value: {
      type: Number,
      default: null
    }
  },
  data() {
    return {
      date: this.value,
      activeDate: "201901",
      activeDay: "",
      active: false,
      days: [
        "Monday",
        "Tuesday",
        "Wednesday",
        "Thursday",
        "Friday",
        "Saturday",
        "Sunday"
      ],
      months: {
        "01": "January",
        "02": "February",
        "03": "March",
        "04": "April",
        "05": "May",
        "06": "June",
        "07": "July",
        "08": "August",
        "09": "September",
        "10": "October",
        "11": "November",
        "12": "December"
      },
      calendarDays: [],
      swipeLeft: false,
      swipeRight: false
    };
  },
  filters: {
    trimThreeChar(item) {
      return item.substr(0, 3);
    }
  },
  computed: {
    monthYearFormatted() {
      // Returns the current month name followed by the current year
      return (
        this.months[this.activeDate.substr(4, 2)] +
        " " +
        this.activeDate.substr(0, 4)
      );
    }
  },
  methods: {
    focus() {
      this.active = true;
      this.cellHeight = this.$el.querySelector("input").offsetWidth / 7;
    },
    blur() {
      this.active = false;
    },
    getCurrentDate() {
      // Returns today's date with format YYYY/MM/DD
      return new Date()
        .toJSON()
        .slice(0, 10)
        .replace(/-/g, "/");
    },
    getCurrentDay() {
      // Returns current day within range 1-31
      return new Date().getDate();
    },
    getCurrentMonth() {
      // Returns the current month as a 2 digits number
      return ("0" + (new Date().getMonth() + 1)).slice(-2);
    },
    getPreviousMonth(month) {
      if (parseInt(month) == 1) return "12";
      else return ("0" + (parseInt(month) - 1)).slice(-2);
    },
    getNextMonth(month) {
      if (parseInt(month) == 12) return "01";
      else return ("0" + (parseInt(month) + 1)).slice(-2);
    },
    getCurrentYear() {
      // Returns the current year as a 4 digits number
      return new Date().getFullYear();
    },
    getNumberOfDaysInMonth(year, month) {
      return new Date(year, month, 0).getDate();
    },
    getNumberOfDaysInPreviousMonth(year, month) {
      if (month == "01")
        return new Date(year - 1, this.getPreviousMonth(month), 0).getDate();
      else return new Date(year, this.getPreviousMonth(month), 0).getDate();
    },
    getNumberOfDaysInNextMonth() {
      if (this.getNextMonth() == "12")
        return new Date(
          this.getCurrentYear() + 1,
          this.getNextMonth(),
          0
        ).getDate();
      else
        return new Date(
          this.getCurrentYear(),
          this.getNextMonth(),
          0
        ).getDate();
    },
    getFirstDayOfMonth(year, month) {
      if (new Date(year, parseInt(month) - 1, 1).getDay() - 1 == -1) return 6;
      else return new Date(year, parseInt(month) - 1, 1).getDay() - 1;
    },
    getLastDayOfMonth(year, month) {
      return new Date(year, parseInt(month) + 1, 0).getDay();
    },
    populateCalendar(year, month) {
      this.calendarDays = [];
      const firstDayOfMonth = this.getFirstDayOfMonth(year, month);
      const numberOfRows = Math.ceil(
        this.getNumberOfDaysInMonth(year, month) / 7
      );
      let dayNumber = 1;
      // Number of rows to display
      let i = 1;
      for (let x = 0; x <= numberOfRows; x++) {
        let d = 0;
        let row = [];
        // let endOfMonth = false;
        // Fills the first row with previous month ending days
        if (x == 0) {
          for (let i = 0; i < firstDayOfMonth; i++) {
            if (parseInt(month) == 1)
              row.push({
                day: this.getNumberOfDaysInPreviousMonth(year, month) - d,
                month: this.getPreviousMonth(month),
                year: parseInt(year) - 1
              });
            else
              row.push({
                day: this.getNumberOfDaysInPreviousMonth(year, month) - d,
                month: this.getPreviousMonth(month),
                year: parseInt(year)
              });
            d++;
          }
          row.reverse();
        }
        // Fills the rest of the rows
        for (
          let y = row.length;
          y < 7 && y < this.getNumberOfDaysInMonth(year, month);
          y++
        ) {
          // While current month days are not excedeed
          if (dayNumber <= this.getNumberOfDaysInMonth(year, month)) {
            row.push({ day: dayNumber, month: month, year: year });
            dayNumber++;
          } else {
            // Fills space left
            for (let j = row.length; j < 7; j++) {
              if (parseInt(month) == 12)
                row.push({
                  day: i,
                  month: this.getNextMonth(month),
                  year: parseInt(year) + 1
                });
              else
                row.push({
                  day: i,
                  month: this.getNextMonth(month),
                  year: year
                });
              i++;
              // let t = row;
            }
          }
        }
        this.calendarDays.push(row);
      }
    },
    setPreviousMonth() {
      let activeYear = this.activeDate.substr(0, 4);
      let activeMonth = this.activeDate.substr(4, 2);
      if (parseInt(activeMonth) == 1) {
        activeYear = parseInt(activeYear) - 1;
        activeMonth = "12";
      } else {
        activeMonth = ("0" + (parseInt(activeMonth) - 1)).slice(-2);
      }
      this.activeDate = activeYear + activeMonth;
      this.populateCalendar(activeYear, activeMonth);
      this.animeSwipeRight();
    },
    setNextMonth() {
      let activeYear = this.activeDate.substr(0, 4);
      let activeMonth = this.activeDate.substr(4, 2);
      if (parseInt(activeMonth) == 12) {
        activeYear = parseInt(activeYear) + 1;
        activeMonth = "01";
      } else {
        activeMonth = ("0" + (parseInt(activeMonth) + 1)).slice(-2);
      }
      this.activeDate = activeYear + activeMonth;
      this.populateCalendar(activeYear, activeMonth);
      this.animeSwipeLeft();
    },
    selectDay(cell) {
      if (cell.month == this.getPreviousMonth(this.activeDate.substr(4, 2))) {
        this.setPreviousMonth();
      } else if (
        cell.month == this.getNextMonth(this.activeDate.substr(4, 2))
      ) {
        this.setNextMonth();
      }
      this.activeDay = ("0" + cell.day).slice(-2) + cell.month + cell.year;
      this.date =
        ("0" + cell.day).slice(-2) + "/" + cell.month + "/" + cell.year;
    },
    isSelected(cell) {
      if (("0" + cell.day).slice(-2) + cell.month + cell.year == this.activeDay)
        return true;
    },
    isDisabled(cell) {
      if (parseInt(cell.month) !== parseInt(this.activeDate.substr(4, 2)))
        return true;
    },
    animeSwipeLeft() {
      this.swipeLeft = true;
      setTimeout(() => {
        this.swipeLeft = false;
      }, 300);
    },
    animeSwipeRight() {
      this.swipeRight = true;
      setTimeout(() => {
        this.swipeRight = false;
      }, 300);
    },
    setDefaults() {
      // if (this.$props.color) {
      // }
      // if (this.$props.start) {
      // }
    }
  },
  watch: {
    date: function() {
      this.$emit("input", this.date);
    }
  },
  mounted() {
    this.setDefaults();
    this.populateCalendar(2017, 1);
    this.activeDate = this.getCurrentYear() + this.getCurrentMonth();
  }
};
</script>


<style lang="scss">
.calendar {
  --datepicker-accent: red;
  --datepicker-background: blue;
  --datepicker-body-color: black;
  --datepicker-header-color: white;
  position: relative;
  box-sizing: border-box;
  &__layout {
    position: absolute;
    z-index: 1;
    width: 100%;
    display: flex;
    flex-direction: column;
    animation: appear 0.3s ease forwards;
    border-radius: 0.25rem;
    overflow: hidden;
  }
  &__header {
    width: 100%;
    background-color: var(--datepicker-accent);
    display: flex;
    flex-direction: column;
    align-items: center;
    color: #fff;
    &-title {
      background-color: rgba(0, 0, 0, 0.1);
      width: 100%;
      display: flex;
      flex-direction: row;
      justify-content: space-between;
      align-items: center;
      padding: 0.5rem;
    }
    &-days {
      display: flex;
      justify-content: space-between;
      width: 100%;
      font-size: 14px;
      padding: 0.5rem;
    }
  }
  &__day-name {
    max-width: calc(100% / 7);
    padding: 0.5rem;
    text-align: center;
    font-weight: bold;
    overflow: hidden;
  }
  &__body {
    display: flex;
    flex-direction: column;
    background-color: var(--datepicker-background, white);
    overflow: hidden;
  }
  &__row {
    display: flex;
    flex-direction: row;
    transition: all 0.15s ease;
  }
  &__day {
    width: calc(100% / 7);
    padding: 0.5rem;
    display: flex;
    justify-content: center;
    align-items: center;
    font-size: 14px;
    color: rgba(0, 0, 0, 0.75);
    position: relative;
    cursor: pointer;
    transition: all 0.3s ease;
    &.is-disabled {
      background-color: rgba(0, 0, 0, 0.1);
      color: rgba(0, 0, 0, 0.3);
    }
    &.is-selected {
      color: #ff4364;
      &:after {
        width: 100%;
      }
    }
    &:after {
      content: "";
      height: 3px;
      width: 100%;
      background-color: var(--datepicker-accent);
      position: absolute;
      bottom: 0;
      left: 0;
      transform: scale(0, 1);
      transform-origin: 0 0;
      transition: transform 0.3s ease;
    }
    &:hover {
      color: #ff4364;
    }
    &:hover:after {
      transform: scale(1, 1);
    }
  }
  &.swipe-left {
    opacity: 0;
    transform: translateX(10px);
  }
  &.swipe-right {
    opacity: 0;
    transform: translateX(-10px);
  }
  &__footer {
  }
}

@keyframes appear {
  from {
    transform: translateY(-10px);
    opacity: 0;
  }
  to {
    transform: translateY(5px);
    opacity: 1;
  }
}
</style>
