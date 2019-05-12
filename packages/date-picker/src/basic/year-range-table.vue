<template>
  <table @click="handleYearTableClick" @mousemove="handleMouseMove" class="el-month-table">
    <tbody>
      <tr v-for="(row, i) in rows" :key="i">
        <td :class="getCellStyle(cell, startYear + key+(4*i))" v-for="(cell, key) in row" :key="key">
          <div>
            <a class="cell">{{ startYear + key+(4*i) }}</a>
          </div>
        </td>
      </tr>
    </tbody>
  </table>
</template>

<script type="text/babel">
  import { hasClass } from 'element-ui/src/utils/dom';
  import Locale from 'element-ui/src/mixins/locale';
  import { isDate, range, nextDate, getDayCountOfYear } from 'element-ui/src/utils/date-util';
  import { arrayFindIndex, coerceTruthyValueToArray, arrayFind } from 'element-ui/src/utils/util';

  const datesInYear = year => {
    const numOfDays = getDayCountOfYear(year);
    const firstDay = new Date(year, 0, 1);
    return range(numOfDays).map(n => nextDate(firstDay, n));
  };
  const clearDate = (date) => {
    return new Date(date.getFullYear(), 1);
  };
  const getMonthTimestamp = function(time) {
    if (typeof time === 'number' || typeof time === 'string') {
      return clearDate(new Date(time)).getTime();
    } else if (time instanceof Date) {
      return clearDate(time).getTime();
    } else {
      return NaN;
    }
  };
  export default {
    props: {
      disabledDate: {},
      value: {},
      selectionMode: {
        default: 'year'
      },
      minDate: {},
      maxDate: {},
      defaultValue: {
        validator(val) {
          // null or valid Date Object
          return val === null || (val instanceof Date && isDate(val));
        }
      },
      date: {},
      rangeState: {
        default() {
          return {
            endDate: null,
            selecting: false
          };
        }
      }
    },
    mixins: [Locale],
    watch: {
      'rangeState.endDate'(newVal) {
        this.markRange(this.minDate, newVal);
      },
      minDate(newVal, oldVal) {
        if (getMonthTimestamp(newVal) !== getMonthTimestamp(oldVal)) {
          this.markRange(this.minDate, this.maxDate);
        }
      },
      maxDate(newVal, oldVal) {
        if (getMonthTimestamp(newVal) !== getMonthTimestamp(oldVal)) {
          this.markRange(this.minDate, this.maxDate);
        }
      }
    },

    data() {
      return {
        tableRows: [ [], [], [] ],
        lastRow: null,
        lastColumn: null
      };
    },

    computed: {
      startYear() {
        return Math.floor(this.date.getFullYear() / 10) * 10;
      },
      rows() {
        // TODO: refactory rows / getCellClasses
        const rows = this.tableRows;
        const disabledDate = this.disabledDate;
        const selectedDate = [];
        const now = new Date(Math.floor(this.date.getFullYear() / 10) * 10, 1).getTime();
        for (let i = 0; i < 3; i++) {
          const row = rows[i];
          if (i === 2) {
            for (let j = 0; j < 2; j++) {
              let cell = row[j];
              if (!cell) {
                cell = { row: i, column: j, type: 'normal', inRange: false, start: false, end: false };
              }
              cell.type = 'normal';
              let index = i * 4 + j;
              let yearArr = Math.floor(this.date.getFullYear() / 10) * 10 + index;
              let time = new Date(yearArr, 1).getTime();
              cell.inRange = time >= getMonthTimestamp(this.minDate) && time <= getMonthTimestamp(this.maxDate);
              cell.start = this.minDate && time === getMonthTimestamp(this.minDate);
              cell.end = this.maxDate && time === getMonthTimestamp(this.maxDate);
              const isToday = time === now;

              if (isToday) {
                cell.type = 'today';
              }
              cell.text = index;
              let cellDate = new Date(time);
              cell.disabled = typeof disabledDate === 'function' && disabledDate(cellDate);
              cell.selected = arrayFind(selectedDate, date => date.getTime() === cellDate.getTime());
              this.$set(row, j, cell);
            }
          } else {
            for (let j = 0; j < 4; j++) {
              let cell = row[j];
              if (!cell) {
                cell = { row: i, column: j, type: 'normal', inRange: false, start: false, end: false };
              }
              cell.type = 'normal';
              let index = i * 4 + j;
              let yearArr = Math.floor(this.date.getFullYear() / 10) * 10 + index;
              let time = new Date(yearArr, 1).getTime();
              cell.inRange = time >= getMonthTimestamp(this.minDate) && time <= getMonthTimestamp(this.maxDate);
              cell.start = this.minDate && time === getMonthTimestamp(this.minDate);
              cell.end = this.maxDate && time === getMonthTimestamp(this.maxDate);
              const isToday = time === now;
              if (isToday) {
                cell.type = 'today';
              }
              cell.text = index;
              let cellDate = new Date(time);
              cell.disabled = typeof disabledDate === 'function' && disabledDate(cellDate);
              cell.selected = arrayFind(selectedDate, date => date.getTime() === cellDate.getTime());

              this.$set(row, j, cell);
            }
          }
        }
        return rows;
      }
    },

    methods: {
      getCellStyle(cell, year) {
        const style = {};
        const today = new Date();
        style.disabled = typeof this.disabledDate === 'function'
          ? datesInYear(year).every(this.disabledDate)
          : false;
        style.current = arrayFindIndex(coerceTruthyValueToArray(this.value), date => date.getFullYear() === year) >= 0;
        style.today = today.getFullYear() === year;
        style.default = this.defaultValue && this.defaultValue.getFullYear() === year;
        if (cell.inRange) {
          style['in-range'] = true;

          if (cell.start) {
            style['start-date'] = true;
          }

          if (cell.end) {
            style['end-date'] = true;
          }
        }
        return style;
      },
      markRange(minDate, maxDate) {
        minDate = getMonthTimestamp(minDate);
        maxDate = getMonthTimestamp(maxDate) || minDate;
        [minDate, maxDate] = [Math.min(minDate, maxDate), Math.max(minDate, maxDate)];
        const rows = this.rows;
        for (let i = 0, k = rows.length; i < k; i++) {
          const row = rows[i];
          for (let j = 0, l = row.length; j < l; j++) {

            const cell = row[j];
            const index = i * 4 + j;
            const yearArr = Math.floor(this.date.getFullYear() / 10) * 10 + index;
            const time = new Date(yearArr, 1).getTime();
            cell.inRange = minDate && time >= minDate && time <= maxDate;
            cell.start = minDate && time === minDate;
            cell.end = maxDate && time === maxDate;
          }
        }
      },
      getMonthOfCell(month) {
        const year = Math.floor(this.date.getFullYear() / 10) * 10 + month;
        return new Date(year, 1);
      },
      handleMouseMove(event) {
        if (!this.rangeState.selecting) return;
        let target = event.target;
        if (target.tagName === 'A') {
          target = target.parentNode.parentNode;
        }
        if (target.tagName === 'DIV') {
          target = target.parentNode;
        }
        if (target.tagName !== 'TD') return;

        const row = target.parentNode.rowIndex;
        const column = target.cellIndex;
        // can not select disabled date
        // if (this.rows[row][column].disabled) return;

        // only update rangeState when mouse moves to a new cell
        // this avoids frequent Date object creation and improves performance
        if (row !== this.lastRow || column !== this.lastColumn) {
          this.lastRow = row;
          this.lastColumn = column;
          this.$emit('changerange', {
            minDate: this.minDate,
            maxDate: this.maxDate,
            rangeState: {
              selecting: true,
              endDate: this.getMonthOfCell(row * 4 + column)
            }
          });
        }
      },
      handleYearTableClick(event) {
        let target = event.target;
        if (target.tagName === 'A') {
          target = target.parentNode.parentNode;
        }
        if (target.tagName === 'DIV') {
          target = target.parentNode;
        }
        if (target.tagName !== 'TD') return;
        if (hasClass(target, 'disabled')) return;
        const year = target.textContent || target.innerText;
        if (this.selectionMode === 'range') {
          if (!this.rangeState.selecting) {
            this.$emit('pick', {minDate: year, maxDate: null});
            this.rangeState.selecting = true;
          } else {
            if (year >= this.minDate) {
              this.$emit('pick', {minDate: this.minDate, maxDate: year});
            } else {
              this.$emit('pick', {minDate: year, maxDate: this.minDate});
            }
            this.rangeState.selecting = false;
          }
        } else {
          this.$emit('pick', Number(year));
        }
      }
    }
  };
</script>
