<template>
  <transition name="el-zoom-in-top" @after-enter="handleEnter" @after-leave="handleLeave">
    <div
        v-show="visible"
        class="el-picker-panel el-date-picker el-popper"
        :class="[{
        'has-sidebar': $slots.sidebar || shortcuts,
        'has-time': showTime
      }, popperClass]">
      <div class="el-picker-panel__body-wrapper">
        <slot name="sidebar" class="el-picker-panel__sidebar"></slot>
        <div class="el-picker-panel__sidebar" v-if="shortcuts">
          <button
              type="button"
              class="el-picker-panel__shortcut"
              v-for="(shortcut, key) in shortcuts"
              :key="key"
              @click="handleShortcutClick(shortcut)">{{ shortcut.text }}</button>
        </div>
        <div class="el-picker-panel__body">
          <!--日期-->
<!--          <div class="el-date-picker__time-header" v-if="showTime" style="display: none">-->
<!--            <span class="el-date-picker__editor-wrap">-->
<!--              <el-input-->
<!--                  :placeholder="t('el.datepicker.selectDate')"-->
<!--                  :value="visibleDate"-->
<!--                  size="small"-->
<!--                  @input="val => userInputDate = val"-->
<!--                  @change="handleVisibleDateChange" />-->
<!--            </span>-->
<!--          </div>-->

          <div
              class="el-date-picker__header"
              :class="{ 'el-date-picker__header--bordered': currentView === 'year' || currentView === 'month' }"
              v-show="currentView !== 'time'">
            <button
                type="button"
                @click="prevYear"
                :aria-label="t(`el.datepicker.prevYear`)"
                class="el-picker-panel__icon-btn el-date-picker__prev-btn el-icon-d-arrow-left">
            </button>
            <button
                type="button"
                @click="prevMonth"
                v-show="currentView === 'date'"
                :aria-label="t(`el.datepicker.prevMonth`)"
                class="el-picker-panel__icon-btn el-date-picker__prev-btn el-icon-arrow-left">
            </button>
            <span
                @click="showYearPicker"
                role="button"
                class="el-date-picker__header-label">{{ yearLabel }}</span>
            <span
                @click="showMonthPicker"
                v-show="currentView === 'date'"
                role="button"
                class="el-date-picker__header-label"
                :class="{ active: currentView === 'month' }">{{t(`el.datepicker.month${ month + 1 }`)}}</span>
            <button
                type="button"
                @click="nextYear"
                :aria-label="t(`el.datepicker.nextYear`)"
                class="el-picker-panel__icon-btn el-date-picker__next-btn el-icon-d-arrow-right">
            </button>
            <button
                type="button"
                @click="nextMonth"
                v-show="currentView === 'date'"
                :aria-label="t(`el.datepicker.nextMonth`)"
                class="el-picker-panel__icon-btn el-date-picker__next-btn el-icon-arrow-right">
            </button>
          </div>

          <div class="el-picker-panel__content">
            <date-table
                v-show="currentView === 'date'"
                @pick="handleDatePick"
                :selection-mode="selectionMode"
                :first-day-of-week="firstDayOfWeek"
                :value="value"
                :default-value="defaultValue ? new Date(defaultValue) : null"
                :date="date"
                :cell-class-name="cellClassName"
                :disabled-date="disabledDate">
            </date-table>
            <year-table
                v-show="currentView === 'year'"
                @pick="handleYearPick"
                :value="value"
                :default-value="defaultValue ? new Date(defaultValue) : null"
                :date="date"
                :disabled-date="disabledDate">
            </year-table>
            <month-table
                v-show="currentView === 'month'"
                @pick="handleMonthPick"
                :value="value"
                :default-value="defaultValue ? new Date(defaultValue) : null"
                :date="date"
                :disabled-date="disabledDate">
            </month-table>
          </div>

          <!--时间-->
          <div class="el-date-picker__time-wrap-cisdi" v-if="showTime" v-clickoutside="handleTimePickClose">
            <span>时间：</span>
            <el-input
                ref="input"
                placeholder="时"
                :value="visibleTimeH"
                size="small"
                :maxlength="2"
                :disabled="!hasTimeH"
                @input="val => userInputTimeH = val"
                @change="handleVisibleTimeChange($event, 'H')" />
            <span>：</span>
            <el-input
                ref="inputM"
                placeholder="分"
                :value="visibleTimeM"
                size="small"
                :maxlength="2"
                :disabled="!hasTimeM"
                @input="val => userInputTimeM = val"
                @change="handleVisibleTimeChange($event, 'M')" />
            <span>：</span>
            <el-input
                ref="inputS"
                placeholder="秒"
                :value="visibleTimeS"
                size="small"
                :maxlength="2"
                :disabled="!hasTimeS"
                @input="val => userInputTimeS = val"
                @change="handleVisibleTimeChange($event, 'S')" />
            <time-picker
                ref="timepicker"
                :time-arrow-control="arrowControl"
                @pick="handleTimePick"
                :visible="timePickerVisible"
                @mounted="proxyTimePickerDataProperties">
            </time-picker>
          </div>
        </div>
      </div>

      <div
          class="el-picker-panel__footer el-picker-panel__footer-cisdi"
          v-show="footerVisible && currentView === 'date'">
        <el-button
            size="mini"
            type="text"
            class="el-picker-panel__link-btn el-picker-panel__link-btn-clear"
            @click="changeToClear"
            v-show="selectionMode !== 'dates'">
          清除
        </el-button>
        <el-button
            size="mini"
            type="text"
            class="el-picker-panel__link-btn el-picker-panel__link-btn-now"
            @click="changeToNow"
            v-show="selectionMode !== 'dates'">
          现在
        </el-button>
        <el-button
            type="primary"
            size="mini"
            class="el-picker-panel__link-btn"
            @click="confirm">
          {{ t('el.datepicker.confirm') }}
        </el-button>
      </div>
    </div>
  </transition>
</template>

<script type="text/babel">
import {
  formatDate,
  parseDate,
  getWeekNumber,
  isDate,
  modifyDate,
  modifyTime,
  modifyWithTimeString,
  clearMilliseconds,
  clearTime,
  prevYear,
  nextYear,
  prevMonth,
  nextMonth,
  changeYearMonthAndClampDate,
  extractDateFormat,
  extractTimeFormat,
  timeWithinRange,
  toDate
} from 'element-ui/src/utils/date-util';
import Clickoutside from 'element-ui/src/utils/clickoutside';
import Locale from 'element-ui/src/mixins/locale';
import ElInput from 'element-ui/packages/input';
import ElButton from 'element-ui/packages/button';
import TimePicker from './time';
import YearTable from '../basic/year-table';
import MonthTable from '../basic/month-table';
import DateTable from '../basic/date-table';

export default {
  mixins: [Locale],

  directives: { Clickoutside },

  watch: {
    showTime(val) {
      /* istanbul ignore if */
      if (!val) return;
      this.$nextTick(_ => {
        const inputElm = this.$refs.input.$el;
        if (inputElm) {
          this.pickerWidth = inputElm.getBoundingClientRect().width + 10;
        }
      });
    },

    value(val) {
      if (this.selectionMode === 'dates' && this.value) return;
      if (isDate(val)) {
        this.date = new Date(val);
      } else {
        this.date = this.getDefaultValue();
      }
    },

    defaultValue(val) {
      if (!isDate(this.value)) {
        this.date = val ? new Date(val) : new Date();
      }
    },

    timePickerVisible(val) {
      if (val) this.$nextTick(() => this.$refs.timepicker.adjustSpinners());
    },

    selectionMode(newVal) {
      if (newVal === 'month') {
        /* istanbul ignore next */
        if (this.currentView !== 'year' || this.currentView !== 'month') {
          this.currentView = 'month';
        }
      } else if (newVal === 'dates') {
        this.currentView = 'date';
      }
    }
  },

  methods: {
    proxyTimePickerDataProperties() {
      const format = timeFormat => {this.$refs.timepicker.format = timeFormat;};
      const value = value => {this.$refs.timepicker.value = value;};
      const date = date => {this.$refs.timepicker.date = date;};
      const selectableRange = selectableRange => {this.$refs.timepicker.selectableRange = selectableRange;};

      this.$watch('value', value);
      this.$watch('date', date);
      this.$watch('selectableRange', selectableRange);

      format(this.timeFormat);
      value(this.value);
      date(this.date);
      selectableRange(this.selectableRange);
    },

    handleClear() {
      this.date = this.getDefaultValue();
      this.$emit('pick', null);
    },

    emit(value, ...args) {
      if (!value) {
        this.$emit('pick', value, ...args);
      } else if (Array.isArray(value)) {
        const dates = value.map(date => this.showTime ? clearMilliseconds(date) : clearTime(date));
        this.$emit('pick', dates, ...args);
      } else {
        this.$emit('pick', this.showTime ? clearMilliseconds(value) : clearTime(value), ...args);
      }
      this.userInputDate = null;
      this.userInputTimeH = null;
      this.userInputTimeM = null;
      this.userInputTimeS = null;
    },

    // resetDate() {
    //   this.date = new Date(this.date);
    // },

    showMonthPicker() {
      this.currentView = 'month';
    },

    showYearPicker() {
      this.currentView = 'year';
    },

    // XXX: 没用到
    // handleLabelClick() {
    //   if (this.currentView === 'date') {
    //     this.showMonthPicker();
    //   } else if (this.currentView === 'month') {
    //     this.showYearPicker();
    //   }
    // },

    prevMonth() {
      this.date = prevMonth(this.date);
    },

    nextMonth() {
      this.date = nextMonth(this.date);
    },

    prevYear() {
      if (this.currentView === 'year') {
        this.date = prevYear(this.date, 10);
      } else {
        this.date = prevYear(this.date);
      }
    },

    nextYear() {
      if (this.currentView === 'year') {
        this.date = nextYear(this.date, 10);
      } else {
        this.date = nextYear(this.date);
      }
    },

    handleShortcutClick(shortcut) {
      if (shortcut.onClick) {
        shortcut.onClick(this);
      }
    },

    handleTimePick(value, visible, first) {
      if (isDate(value)) {
        const newDate = this.value
          ? modifyTime(this.value, value.getHours(), value.getMinutes(), value.getSeconds())
          : modifyWithTimeString(this.getDefaultValue(), this.defaultTime);
        this.date = newDate;
        this.emit(this.date, true);
      } else {
        this.emit(value, true);
      }
      if (!first) {
        this.timePickerVisible = visible;
      }
    },

    handleTimePickClose() {
      this.timePickerVisible = false;
    },

    handleMonthPick(month) {
      if (this.selectionMode === 'month') {
        this.date = modifyDate(this.date, this.year, month, 1);
        this.emit(this.date);
      } else {
        this.date = changeYearMonthAndClampDate(this.date, this.year, month);
        // TODO: should emit intermediate value ??
        // this.emit(this.date);
        this.currentView = 'date';
      }
    },

    handleDatePick(value) {
      if (this.selectionMode === 'day') {
        let newDate = this.value
          ? modifyDate(this.value, value.getFullYear(), value.getMonth(), value.getDate())
          : modifyWithTimeString(value, this.defaultTime);
        // change default time while out of selectableRange
        if (!this.checkDateWithinRange(newDate)) {
          newDate = modifyDate(this.selectableRange[0][0], value.getFullYear(), value.getMonth(), value.getDate());
        }
        this.date = newDate;
        this.emit(this.date, this.showTime);
      } else if (this.selectionMode === 'week') {
        this.emit(value.date);
      } else if (this.selectionMode === 'dates') {
        this.emit(value, true); // set false to keep panel open
      }
    },

    handleYearPick(year) {
      if (this.selectionMode === 'year') {
        this.date = modifyDate(this.date, year, 0, 1);
        this.emit(this.date);
      } else {
        this.date = changeYearMonthAndClampDate(this.date, year, this.month);
        // TODO: should emit intermediate value ??
        // this.emit(this.date, true);
        this.currentView = 'month';
      }
    },

    // 清除
    changeToClear() {
      this.date = null;
      this.emit(this.date);
    },

    // 此刻
    changeToNow() {
      // NOTE: not a permanent solution
      //       consider disable "now" button in the future
      if ((!this.disabledDate || !this.disabledDate(new Date())) && this.checkDateWithinRange(new Date())) {
        this.date = new Date();
        this.emit(this.date);
      }
    },

    confirm() {
      if (this.selectionMode === 'dates') {
        this.emit(this.value);
      } else {
        // value were emitted in handle{Date,Time}Pick, nothing to update here
        // deal with the scenario where: user opens the picker, then confirm without doing anything
        const value = this.value
          ? this.value
          : modifyWithTimeString(this.getDefaultValue(), this.defaultTime);
        this.date = new Date(value); // refresh date
        this.emit(value);
      }
    },

    resetView() {
      if (this.selectionMode === 'month') {
        this.currentView = 'month';
      } else if (this.selectionMode === 'year') {
        this.currentView = 'year';
      } else {
        this.currentView = 'date';
      }
    },

    handleEnter() {
      document.body.addEventListener('keydown', this.handleKeydown);
    },

    handleLeave() {
      this.$emit('dodestroy');
      document.body.removeEventListener('keydown', this.handleKeydown);
    },

    handleKeydown(event) {
      const keyCode = event.keyCode;
      const list = [38, 40, 37, 39];
      if (this.visible && !this.timePickerVisible) {
        if (list.indexOf(keyCode) !== -1) {
          this.handleKeyControl(keyCode);
          event.stopPropagation();
          event.preventDefault();
        }
        if (keyCode === 13 && this.userInputDate === null && this.userInputTimeH === null && this.userInputTimeM === null && this.userInputTimeS === null) { // Enter
          this.emit(this.date, false);
        }
      }
    },

    handleKeyControl(keyCode) {
      const mapping = {
        'year': {
          38: -4, 40: 4, 37: -1, 39: 1, offset: (date, step) => date.setFullYear(date.getFullYear() + step)
        },
        'month': {
          38: -4, 40: 4, 37: -1, 39: 1, offset: (date, step) => date.setMonth(date.getMonth() + step)
        },
        'week': {
          38: -1, 40: 1, 37: -1, 39: 1, offset: (date, step) => date.setDate(date.getDate() + step * 7)
        },
        'day': {
          38: -7, 40: 7, 37: -1, 39: 1, offset: (date, step) => date.setDate(date.getDate() + step)
        }
      };
      const mode = this.selectionMode;
      const year = 3.1536e10;
      const now = this.date.getTime();
      const newDate = new Date(this.date.getTime());
      while (Math.abs(now - newDate.getTime()) <= year) {
        const map = mapping[mode];
        map.offset(newDate, map[keyCode]);
        if (typeof this.disabledDate === 'function' && this.disabledDate(newDate)) {
          continue;
        }
        this.date = newDate;
        this.$emit('pick', newDate, true);
        break;
      }
    },

    // 输入时间改变
    handleVisibleTimeChange(val, type) {
      // 输入不是两位数字
      if (!RegExp(/^\d{1,2}$/).test(val)) {
        val = '00';
      }
      let hour = '';
      let min = '';
      let sec = '';
      if (type === 'H') {
        hour = val || '00';
        min = this.visibleTimeM || '00';
        sec = this.visibleTimeS || '00';
      } else if (type === 'M') {
        hour = this.visibleTimeH || '00';
        min = val || '00';
        sec = this.visibleTimeS || '00';
      } else if (type === 'S') {
        hour = this.visibleTimeH || '00';
        min = this.visibleTimeM || '00';
        sec = val || '00';
      }
      // 不可编辑但设置了默认值
      if (type !== 'H' && !this.hasTimeH && this.defaultTime && this.defaultTime.substring(0, 2)) {
        hour = this.defaultTime.substring(0, 2);
      }
      if (type !== 'M' && !this.hasTimeM && this.defaultTime && this.defaultTime.substring(3, 5)) {
        min = this.defaultTime.substring(3, 5);
      }
      if (type !== 'S' && !this.hasTimeS && this.defaultTime && this.defaultTime.substring(6, 8)) {
        sec = this.defaultTime.substring(6, 8);
      }

      // const time = parseDate(hour + ':' + min + ':' + sec, this.timeFormat);
      const time = parseDate(this.pad(hour) + ':' + this.pad(min) + ':' + this.pad(sec), 'HH:mm:ss');

      if (time && this.checkDateWithinRange(time)) {
        this.date = modifyDate(time, this.year, this.month, this.monthDate);
        this.userInputTimeH = null;
        this.userInputTimeM = null;
        this.userInputTimeS = null;
        this.$refs.timepicker.value = this.date;
        this.timePickerVisible = false;
        this.emit(this.date, true);
      }
    },

    handleVisibleDateChange(value) {
      const date = parseDate(value, this.dateFormat);
      if (date) {
        if (typeof this.disabledDate === 'function' && this.disabledDate(date)) {
          return;
        }
        this.date = modifyTime(date, this.date.getHours(), this.date.getMinutes(), this.date.getSeconds());
        this.userInputDate = null;
        this.resetView();
        this.emit(this.date, true);
      }
    },

    isValidValue(value) {
      return value && !isNaN(value) && (
        typeof this.disabledDate === 'function'
          ? !this.disabledDate(value)
          : true
      ) && this.checkDateWithinRange(value);
    },

    getDefaultValue() {
      // if default-value is set, return it
      // otherwise, return now (the moment this method gets called)
      return this.defaultValue ? new Date(this.defaultValue) : new Date();
    },

    checkDateWithinRange(date) {
      return this.selectableRange.length > 0
        ? timeWithinRange(date, this.selectableRange, this.format || 'HH:mm:ss')
        : true;
    },

    // 格式化数字
    pad(val, len) {
      val = String(val);
      len = len || 2;
      while (val.length < len) {
        val = '0' + val;
      }
      return val;
    }
  },

  components: {
    TimePicker, YearTable, MonthTable, DateTable, ElInput, ElButton
  },

  data() {
    return {
      popperClass: '',
      date: new Date(),
      value: '',
      defaultValue: null, // use getDefaultValue() for time computation
      defaultTime: null,
      showTime: false,
      selectionMode: 'day',
      shortcuts: '',
      visible: false,
      currentView: 'date',
      disabledDate: '',
      cellClassName: '',
      selectableRange: [],
      firstDayOfWeek: 7,
      showWeekNumber: false,
      timePickerVisible: false,
      format: '',
      arrowControl: false,
      userInputDate: null,
      userInputTimeH: null,
      userInputTimeM: null,
      userInputTimeS: null
    };
  },

  computed: {
    year() {
      return this.date.getFullYear();
    },

    month() {
      return this.date.getMonth();
    },

    week() {
      return getWeekNumber(this.date);
    },

    monthDate() {
      return this.date.getDate();
    },

    footerVisible() {
      return this.showTime || this.selectionMode === 'dates';
    },

    // 弹窗中根据format的时分秒格式，显示为X:X:X形式
    visibleTimeH() {
      // return formatDate(this.value || this.defaultValue, this.timeFormat);
      if (this.userInputTimeH !== null) {
        return this.userInputTimeH;
      }
      let dateObj = toDate(this.value || this.defaultValue);
      if (!dateObj) return '';
      if (this.timeFormat.includes('HH')) { // 包含2位24进制时
        return this.pad(dateObj.getHours());
      }
      if (this.timeFormat.includes('hh')) { // 包含2位12进制时
        return this.pad(dateObj.getHours() % 12 || 12);
      }
      // 前后if的先后顺序不能调换
      if (this.timeFormat.includes('H')) { // 包含1位24进制时
        return dateObj.getHours();
      }
      if (this.timeFormat.includes('h')) { // 包含1位12进制时
        return dateObj.getHours() % 12 || 12;
      }
      // 不可编辑但设置了默认值
      if (!this.hasTimeH && this.defaultTime && this.defaultTime.substring(0, 2)) {
        return this.defaultTime.substring(0, 2);
      }
      return '';
    },

    visibleTimeM() {
      if (this.userInputTimeM !== null) {
        return this.userInputTimeM;
      }
      let dateObj = toDate(this.value || this.defaultValue);
      if (!dateObj) return '';
      if (this.timeFormat.includes('mm')) { // 包含2位分
        return this.pad(dateObj.getMinutes());
      }
      // 前后if的先后顺序不能调换
      if (this.timeFormat.includes('m')) { // 包含1位分
        return dateObj.getMinutes();
      }
      // 不可编辑但设置了默认值
      if (!this.hasTimeM && this.defaultTime && this.defaultTime.substring(3, 5)) {
        return this.defaultTime.substring(3, 5);
      }
      return '';
    },

    visibleTimeS() {
      if (this.userInputTimeS !== null) {
        return this.userInputTimeS;
      }
      let dateObj = toDate(this.value || this.defaultValue);
      if (!dateObj) return '';
      if (this.timeFormat.includes('SSS')) { // 包含3位秒
        return this.pad(dateObj.getMilliseconds(), 3);
      }
      if (this.timeFormat.includes('SS')) { // 包含2位秒
        return this.pad(Math.round(dateObj.getMilliseconds() / 10), 2);
      }
      if (this.timeFormat.includes('ss')) { // 包含2位秒
        return this.pad(dateObj.getSeconds());
      }
      // 前后if的先后顺序不能调换
      if (this.timeFormat.includes('s')) { // 包含1位秒
        return dateObj.getSeconds();
      }
      if (this.timeFormat.includes('S')) { // 包含1位秒
        return Math.round(dateObj.getMilliseconds() / 100);
      }
      // 不可编辑但设置了默认值
      if (!this.hasTimeS && this.defaultTime && this.defaultTime.substring(6, 8)) {
        return this.defaultTime.substring(6, 8);
      }
      return '';
    },

    visibleDate() {
      if (this.userInputDate !== null) {
        return this.userInputDate;
      } else {
        return formatDate(this.value || this.defaultValue, this.dateFormat);
      }
    },

    // 是否显示时分秒的输入框
    hasTimeH() {
      if (RegExp(/h+/i).test(this.timeFormat)) {
        return true;
      }
    },

    hasTimeM() {
      if (RegExp(/m+/).test(this.timeFormat)) {
        return true;
      }
    },

    hasTimeS() {
      if (RegExp(/s+/i).test(this.timeFormat)) {
        return true;
      }
    },

    yearLabel() {
      const yearTranslation = this.t('el.datepicker.year');
      if (this.currentView === 'year') {
        const startYear = Math.floor(this.year / 10) * 10;
        if (yearTranslation) {
          return startYear + ' ' + yearTranslation + ' - ' + (startYear + 9) + ' ' + yearTranslation;
        }
        return startYear + ' - ' + (startYear + 9);
      }
      return this.year + ' ' + yearTranslation;
    },

    timeFormat() {
      if (this.format) {
        return extractTimeFormat(this.format);
      } else {
        return 'HH:mm:ss';
      }
    },

    dateFormat() {
      if (this.format) {
        return extractDateFormat(this.format);
      } else {
        return 'yyyy-MM-dd';
      }
    }
  }
};
</script>

<style lang="scss" scoped>
.el-date-picker__time-wrap-cisdi {
  display: flex;
  margin: -10px 15px 10px 15px;
  justify-content: center;
  white-space: nowrap;

  &>span {
    font-size: 14px;
    margin: 0 5px 0 10px;

    &:first-of-type {
      margin-left: 5px;
    }

    &:last-of-type {
      margin-right: 0;
    }
  }

  &>div {
    width: 80px;
    flex: 1;

    /deep/ .el-input__inner {
      text-align: center;
    }
  }
}

.el-picker-panel__footer-cisdi {
  display: flex;
  justify-content: flex-end;
  padding-left: 30px;
  padding-right: 20px;
}

.el-picker-panel__link-btn {
  font-size: 14px!important;
}

.el-picker-panel__link-btn-clear {
  margin-right: 74px;
}

.el-picker-panel__link-btn-now {
  margin-right: 55px;
}

/deep/ .el-date-table td span {
  border-radius: 3px;
  border: 1px solid #fff;
}

</style>