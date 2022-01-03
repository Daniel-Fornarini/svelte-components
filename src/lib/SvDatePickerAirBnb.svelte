<script context="module" lang="ts">
  export interface Colors {
    selected: string;
    inRange: string;
    selectedText: string;
    text: string;
    inRangeBorder: string;
    disabled: string;
    hoveredInRange: string;
  }

  export interface Texts {
    apply: string;
    cancel: string;
    keyboardShortcuts: string;
  }

  export interface Keys {
    arrowDown: number;
    arrowUp: number;
    arrowRight: number;
    arrowLeft: number;
    enter: number;
    pgUp: number;
    pgDn: number;
    end: number;
    home: number;
    questionMark: number;
    esc: number;
  }

  export interface DatepickerOptions {
    ariaLabels?: any;
    keyboardShortcuts?: any[];
    dateLabelFormat?: string;
    sundayFirst?: boolean;
    colors?: Colors;
    monthNames?: string[];
    days?: string[];
    daysShort?: string[];
    texts?: Texts;
  }

  export const datepickerConfig: DatepickerOptions = {}
</script>

<script lang="ts">
  import format from 'date-fns/format';
  import subMonths from 'date-fns/subMonths';
  import addMonths from 'date-fns/addMonths';
  import getDaysInMonth from 'date-fns/getDaysInMonth';
  import lastDayOfMonth from 'date-fns/lastDayOfMonth';
  import getMonth from 'date-fns/getMonth';
  import setMonth from 'date-fns/setMonth';
  import getYear from 'date-fns/getYear';
  import setYear from 'date-fns/setYear';
  import isSameMonth from 'date-fns/isSameMonth';
  import isSameDay from 'date-fns/isSameDay';
  import addDays from 'date-fns/addDays';
  import subDays from 'date-fns/subDays';
  import addWeeks from 'date-fns/addWeeks';
  import subWeeks from 'date-fns/subWeeks';
  import startOfMonth from 'date-fns/startOfMonth';
  import startOfWeek from 'date-fns/startOfWeek';
  import endOfWeek from 'date-fns/endOfWeek';
  import isBefore from 'date-fns/isBefore';
  import isAfter from 'date-fns/isAfter';
  import isValid from 'date-fns/isValid';
  import { debounce, copyObject, findAncestor, randomString } from './../helpers';
  import vClickOutside from 'v-click-outside';
  import ResizeSelect from '../directives/ResizeSelect';
  import { createEventDispatcher, onDestroy, onMount, tick } from 'svelte';

  export let triggerElementId: string = '';
  export let dateOne: string | Date;
  export let dateTwo: string | Date;
  export let minDate: string | Date;
  export let endDate: string | Date;
  export let mode: string = 'range';
  export let offsetY: number = 0;
  export let offsetX: number = 0;
  export let monthsToShow: number = 2;
  export let startOpen: boolean = false;
  export let fullscreenMobile: boolean = false;
  export let inline: boolean = false;
  export let mobileHeader: boolean = false;
  export let disabledDates: Date[] = [];
  export let enabledDates: Date[] = [];
  export let customizedDates: any[] = [];
  export let showActionButtons: boolean = true;
  export let showShortcutsMenuTrigger: boolean = true;
  export let showMonthYearSelect: boolean = false;
  export let yearsForSelect: number = 10;
  export let trigger: boolean = false;
  export let closeAfterSelect: boolean = false;
  export let isTest: boolean = process.env.NODE_ENV === 'test';

  const dispatch = createEventDispatcher();

  let el: HTMLElement;
  let applyButton: HTMLButtonElement;
  let keyboardShortcusMenuClose: HTMLButtonElement;

  let wrapperId: string = 'airbnb-style-datepicker-wrapper-' + randomString(5);
  let dateFormat: string = 'YYYY-MM-DD';
  let dateLabelFormat: string = 'dddd, MMMM D, YYYY';
  let showDatepicker: boolean = false;
  let showKeyboardShortcutsMenu: boolean = false;
  let showMonths: number = 2;
  const colors: Colors = {
    selected: '#00a699',
    inRange: '#66e2da',
    selectedText: '#fff',
    text: '#565a5c',
    inRangeBorder: '#33dacd',
    disabled: '#fff',
    hoveredInRange: '#67f6ee'
  };
  let sundayFirst: boolean = false;
  let ariaLabels: any = {
    chooseDate: (date) => date,
    chooseStartDate: (date) => `Choose ${date} as your start date.`,
    chooseEndDate: (date) => `Choose ${date} as your end date.`,
    selectedDate: (date) => `Selected. ${date}`,
    unavailableDate: (date) => `Not available. ${date}`,
    previousMonth: 'Move backward to switch to the previous month.',
    nextMonth: 'Move forward to switch to the next month.',
    closeDatepicker: 'Close calendar',
    openKeyboardShortcutsMenu: 'Open keyboard shortcuts menu.',
    closeKeyboardShortcutsMenu: 'Close keyboard shortcuts menu'
  };
  let monthNames: string[] = [
    'January',
    'February',
    'March',
    'April',
    'May',
    'June',
    'July',
    'August',
    'September',
    'October',
    'November',
    'December'
  ];
  let days: string[] = [
    'Monday',
    'Tuesday',
    'Wednesday',
    'Thursday',
    'Friday',
    'Saturday',
    'Sunday'
  ];
  let daysShort: string[] = ['Mon', 'Tue', 'Wed', 'Thur', 'Fri', 'Sat', 'Sun'];
  const texts: Texts = {
    apply: 'Apply',
    cancel: 'Cancel',
    keyboardShortcuts: 'Keyboard Shortcuts'
  };
  let keyboardShortcuts: any[] = [
    { symbol: '↵', label: 'Select the date in focus', symbolDescription: 'Enter key' },
    {
      symbol: '←/→',
      label: 'Move backward (left) and forward (right) by one day.',
      symbolDescription: 'Left or right arrow keys'
    },
    {
      symbol: '↑/↓',
      label: 'Move backward (up) and forward (down) by one week.',
      symbolDescription: 'Up or down arrow keys'
    },
    {
      symbol: 'PgUp/PgDn',
      label: 'Switch months.',
      symbolDescription: 'PageUp and PageDown keys'
    },
    {
      symbol: 'Home/End',
      label: 'Go to the first or last day of a week.',
      symbolDescription: 'Home or End keys'
    },
    { symbol: 'Esc', label: 'Close this panel', symbolDescription: 'Escape key' },
    { symbol: '?', label: 'Open this panel', symbolDescription: 'Question mark' }
  ];
  const keys: Keys = {
    arrowDown: 40,
    arrowUp: 38,
    arrowRight: 39,
    arrowLeft: 37,
    enter: 13,
    pgUp: 33,
    pgDn: 34,
    end: 35,
    home: 36,
    questionMark: 191,
    esc: 27
  };
  let startingDate: Date | string = '';
  let months = [];
  let years = [];
  let width: number = 300;
  let selectedDate1: string | Date = '';
  let selectedDate2: string | Date = '';
  let isSelectingDate1: boolean = true;
  let hoverDate: string | Date = '';
  let focusedDate: Date | string;
  let alignRight: boolean = false;
  let triggerPosition: any = {};
  let triggerWrapperPosition: any = {};
  let viewportWidth: string | undefined = undefined;
  let isMobile: boolean | undefined = undefined;
  let isTablet: boolean | undefined = undefined;
  let triggerElement: HTMLElement | undefined = undefined;

  $: wrapperClasses = {
    'asd__wrapper--datepicker-open': showDatepicker,
    'asd__wrapper--full-screen': showFullscreen,
    'asd__wrapper--inline': inline
  };
  $: wrapperStyles = {
    position: inline ? 'static' : 'absolute',
    top: inline ? '0' : triggerPosition.height + offsetY + 'px',
    left: !alignRight ? triggerPosition.left - triggerWrapperPosition.left + offsetX + 'px' : '',
    right: alignRight ? triggerWrapperPosition.right - triggerPosition.right + offsetX + 'px' : '',
    width: width * showMonths + 'px',
    zIndex: inline ? '0' : '100'
  };
  $: innerStyles = {
    'margin-left': showFullscreen ? '-' + viewportWidth : `-${width}px`
  };
  $: keyboardShortcutsMenuStyles = {
    left: showFullscreen ? viewportWidth : `${width}px`
  };
  $: monthWidthStyles = {
    width: showFullscreen ? viewportWidth : width + 'px'
  };
  $: mobileHeaderFallback = mode === 'range' ? 'Select dates' : 'Select date';
  $: showFullscreen = isMobile && fullscreenMobile;
  $: datesSelected = !!(
    (selectedDate1 && selectedDate1 !== '') ||
    (selectedDate2 && selectedDate2 !== '')
  );
  $: allDatesSelected = !!(
    selectedDate1 &&
    selectedDate1 !== '' &&
    selectedDate2 &&
    selectedDate2 !== ''
  );
  $: hasMinDate = !!(minDate && minDate !== '');
  $: isRangeMode = mode === 'range';
  $: isSingleMode = mode === 'single';
  $: datepickerWidth = width * showMonths;
  // FIXME: work?
  $: datePropsCompound = (dateOne as string) + (dateTwo as string);
  $: isDateTwoBeforeDateOne = !dateTwo ? false : isBefore(new Date(dateTwo), new Date(dateOne));
  // FIXME: reactive?
  $: visibleMonths = () => {
    const firstMonthArray = months.filter((m, index) => index > 0);
    const numberOfMonthsArray = [];
    for (let i = 0; i < showMonths; i++) {
      numberOfMonthsArray.push(i);
    }
    return numberOfMonthsArray.map((_, index) => firstMonthArray[index].firstDateOfMonth);
  };

  $: onSelectedDate1Change(selectedDate1);
  function onSelectedDate1Change(newValue) {
    let newDate = !newValue || newValue === '' ? '' : format(newValue, dateFormat);
    dispatch('date-one-selected', newDate);
  }
  $: onSelectedDate2Change(selectedDate2);
  function onSelectedDate2Change(newValue) {
    let newDate = !newValue || newValue === '' ? '' : format(newValue, dateFormat);
    dispatch('date-two-selected', newDate);
  }
  $: onModeChange(mode);
  function onModeChange(newValue) {
    setStartDates();
  }
  $: onMinDateChange(minDate);
  function onMinDateChange(newValue) {
    setStartDates();
    generateMonths();
    generateYears();
  }
  $: onEndDateChange(endDate);
  function onEndDateChange(newValue) {
    generateYears();
  }
  $: onDatePropsCompoundChange(datePropsCompound);
  function onDatePropsCompoundChange(newValue) {
    if (dateOne !== selectedDate1) {
      startingDate = dateOne;
      setStartDates();
      generateMonths();
      generateYears();
    }
    if (isDateTwoBeforeDateOne) {
      selectedDate2 = '';
      dispatch('date-two-selected', '');
    }
  }
  $: onTriggerChange(trigger);
  function onTriggerChange(newValue) {
    if (newValue) {
      setTimeout(() => {
        openDatepicker();
      }, 0);
    }
  }

  onMount(() => {
    setupDatepicker();
    if (sundayFirst) {
      setSundayToFirstDayInWeek();
    }
    viewportWidth = window.innerWidth + 'px';
    isMobile = window.innerWidth < 768;
    isTablet = window.innerWidth >= 768 && window.innerWidth <= 1024;
    this._handleWindowResizeEvent = debounce(() => {
      positionDatepicker();
      setStartDates();
    }, 200);
    this._handleWindowClickEvent = (event) => {
      if (event.target.id === triggerElementId) {
        event.stopPropagation();
        event.preventDefault();
        toggleDatepicker();
      }
    };
    window.addEventListener('resize', this._handleWindowResizeEvent);
    triggerElement = isTest
      ? document.createElement('input')
      : document.getElementById(triggerElementId);
    setStartDates();
    generateMonths();
    generateYears();
    if (startOpen || inline) {
      openDatepicker();
    }
    el.addEventListener('keyup', handleKeyboardInput);
    el.addEventListener('keydown', trapKeyboardInput);
    triggerElement.addEventListener('keyup', handleTriggerInput);
    triggerElement.addEventListener('click', this._handleWindowClickEvent);
  });

  onDestroy(() => {
    window.removeEventListener('resize', this._handleWindowResizeEvent);
    window.removeEventListener('click', this._handleWindowClickEvent);
    el.removeEventListener('keyup', handleKeyboardInput);
    el.removeEventListener('keydown', trapKeyboardInput);
    triggerElement.removeEventListener('keyup', handleTriggerInput);
    triggerElement.removeEventListener('click', this._handleWindowClickEvent);
  });

  function getDayStyles(date: Date) {
    const selected = isSelected(date);
    const inRange = isInRange(date);
    const disabled = isDisabled(date);
    const hoveredInRange = isHoveredInRange(date);
    let styles = {
      width: (width - 30) / 7 + 'px',
      background: selected
        ? colors.selected
        : hoveredInRange
        ? colors.hoveredInRange
        : inRange
        ? colors.inRange
        : '',
      color: selected
        ? colors.selectedText
        : inRange || hoveredInRange
        ? colors.selectedText
        : colors.text,
      border: selected
        ? '1px double ' + colors.selected
        : (inRange && allDatesSelected) || hoveredInRange
        ? '1px double ' + colors.inRangeBorder
        : ''
    };
    if (disabled) {
      styles.background = colors.disabled;
    }
    return styles;
  }

  function getAriaLabelForDate(date: Date) {
    const dateLabel = format(date, dateLabelFormat);
    const disabled = isDisabled(date);
    if (disabled) {
      return ariaLabels.unavailableDate(dateLabel);
    }
    const selected = isSelected(date);
    if (selected) {
      return ariaLabels.selectedDate(dateLabel);
    }
    if (isRangeMode) {
      if (isSelectingDate1) {
        return ariaLabels.chooseStartDate(dateLabel);
      } else {
        return ariaLabels.chooseEndDate(dateLabel);
      }
    } else {
      return ariaLabels.chooseDate(dateLabel);
    }
  }

  function handleClickOutside(event) {
    if (event.target.id === triggerElementId || !showDatepicker || inline) {
      return;
    }
    closeDatepicker();
  }

  function shouldHandleInput(event, key) {
    return event.keyCode === key && (!event.shiftKey || event.keyCode === 191) && showDatepicker;
  }

  function handleTriggerInput(event) {
    if (mode === 'single') {
      setDateFromText(event.target.value);
    }
  }

  function trapKeyboardInput(event) {
    // prevent keys that are used as keyboard shortcuts from propagating out of this element
    // except for the enter key, which is needed to activate buttons
    const shortcutKeyCodes = Object.keys(keys).map((key) => keys[key]);
    shortcutKeyCodes.splice(shortcutKeyCodes.indexOf(13), 1);
    const shouldPreventDefault = shortcutKeyCodes.indexOf(event.keyCode) > -1;
    if (shouldPreventDefault) event.preventDefault();
  }

  function handleKeyboardInput(event) {
    if (shouldHandleInput(event, keys.esc)) {
      if (showKeyboardShortcutsMenu) {
        closeKeyboardShortcutsMenu();
      } else {
        closeDatepicker();
      }
    } else if (showKeyboardShortcutsMenu) {
      // if keyboard shortcutsMenu is open, then esc is the only key we want to have fire events
    } else if (shouldHandleInput(event, keys.arrowDown)) {
      const newDate = addWeeks(new Date(focusedDate), 1);
      const changeMonths = !isSameMonth(newDate, new Date(focusedDate));
      setFocusedDate(newDate);
      if (changeMonths) nextMonth();
    } else if (shouldHandleInput(event, keys.arrowUp)) {
      const newDate = subWeeks(new Date(focusedDate), 1);
      const changeMonths = !isSameMonth(newDate, new Date(focusedDate));
      setFocusedDate(newDate);
      if (changeMonths) previousMonth();
    } else if (shouldHandleInput(event, keys.arrowRight)) {
      const newDate = addDays(new Date(focusedDate), 1);
      const changeMonths = !isSameMonth(newDate, new Date(focusedDate));
      setFocusedDate(newDate);
      if (changeMonths) nextMonth();
    } else if (shouldHandleInput(event, keys.arrowLeft)) {
      const newDate = subDays(new Date(focusedDate), 1);
      const changeMonths = !isSameMonth(newDate, new Date(focusedDate));
      setFocusedDate(newDate);
      if (changeMonths) previousMonth();
    } else if (shouldHandleInput(event, keys.enter)) {
      // on enter key, only select the date if a date is currently in focus
      const target = event.target;
      if (!showKeyboardShortcutsMenu && target && target.tagName === 'TD') {
        selectDate(focusedDate);
      }
    } else if (shouldHandleInput(event, keys.pgUp)) {
      setFocusedDate(subMonths(new Date(focusedDate), 1));
      previousMonth();
    } else if (shouldHandleInput(event, keys.pgDn)) {
      setFocusedDate(addMonths(new Date(focusedDate), 1));
      nextMonth();
    } else if (shouldHandleInput(event, keys.home)) {
      const newDate = startOfWeek(new Date(focusedDate), {
        weekStartsOn: sundayFirst ? 0 : 1
      });
      const changeMonths = !isSameMonth(newDate, new Date(focusedDate));
      setFocusedDate(newDate);
      if (changeMonths) previousMonth();
    } else if (shouldHandleInput(event, keys.end)) {
      const newDate = endOfWeek(new Date(focusedDate), {
        weekStartsOn: sundayFirst ? 0 : 1
      });
      const changeMonths = !isSameMonth(newDate, new Date(focusedDate));
      setFocusedDate(newDate);
      if (changeMonths) nextMonth();
    } else if (shouldHandleInput(event, keys.questionMark)) {
      openKeyboardShortcutsMenu();
    }
  }

  function setDateFromText(value) {
    if (!value || value.length < 10) {
      return;
    }
    // make sure format is either 'YYYY-MM-DD' or 'DD.MM.YYYY'
    const isFormatYearFirst = value.match(
      /^(\d{4})-(0[1-9]|1[0-2])-(0[1-9]|1[0-9]|2[0-9]|3[0-1])$/
    );
    const isFormatDayFirst = value.match(
      /^(0[1-9]|1[0-9]|2[0-9]|3[0-1])[.](0[1-9]|1[0-2])[.](\d{4})$/
    );
    if (!isFormatYearFirst && !isFormatDayFirst) {
      return;
    }
    if (isFormatDayFirst) {
      //convert to YYYY-MM-DD
      value = `${value.substring(6, 10)}-${value.substring(3, 5)}-${value.substring(0, 2)}`;
    }
    const valueAsDateObject = new Date(value);
    if (!isValid(valueAsDateObject)) {
      return;
    }
    const formattedDate = format(valueAsDateObject, dateFormat);
    if (
      isDateDisabled(formattedDate) ||
      isBeforeMinDate(formattedDate) ||
      isAfterEndDate(formattedDate)
    ) {
      return;
    }
    startingDate = subMonths(valueAsDateObject, 1);
    generateMonths();
    generateYears();
    selectDate(formattedDate);
  }

  function isMonthDisabled(year, monthIndex) {
    const monthDate = new Date(year, monthIndex);
    if (hasMinDate && isBefore(monthDate, startOfMonth(new Date(minDate)))) {
      return true;
    }
    return isAfterEndDate(monthDate);
  }

  function generateMonths() {
    months = [];
    let currentMonth = startingDate;
    for (let i = 0; i < showMonths + 2; i++) {
      months.push(getMonthObject(new Date(currentMonth)));
      currentMonth = addMonthsFormatted(currentMonth);
    }
  }

  function generateYears() {
    if (!showMonthYearSelect) return;
    years = [];
    const currentYear = getYear(new Date(startingDate));
    const startYear = minDate ? getYear(new Date(minDate)) : currentYear - yearsForSelect;
    const endYear = endDate ? getYear(new Date(endDate)) : currentYear + yearsForSelect;
    for (var year = startYear; year <= endYear; year++) {
      years.push(year.toString());
    }
  }

  function setupDatepicker() {
    if (datepickerConfig.ariaLabels) {
      ariaLabels = copyObject(datepickerConfig.ariaLabels);
    }
    if (datepickerConfig.keyboardShortcuts) {
      keyboardShortcuts = copyObject(datepickerConfig.keyboardShortcuts);
    }
    if (datepickerConfig.dateLabelFormat) {
      dateLabelFormat = copyObject(datepickerConfig.dateLabelFormat);
    }
    if (datepickerConfig.sundayFirst) {
      sundayFirst = copyObject(datepickerConfig.sundayFirst);
    }
    if (datepickerConfig.colors) {
      const colorsOptions = copyObject(datepickerConfig.colors);
      colors.selected = colorsOptions.selected || colorsOptions.selected;
      colors.inRange = colorsOptions.inRange || colorsOptions.inRange;
      colors.hoveredInRange = colorsOptions.hoveredInRange || colorsOptions.hoveredInRange;
      colors.selectedText = colorsOptions.selectedText || colorsOptions.selectedText;
      colors.text = colorsOptions.text || colorsOptions.text;
      colors.inRangeBorder = colorsOptions.inRangeBorder || colorsOptions.inRangeBorder;
      colors.disabled = colorsOptions.disabled || colorsOptions.disabled;
    }
    if (datepickerConfig.monthNames && datepickerConfig.monthNames.length === 12) {
      monthNames = copyObject(datepickerConfig.monthNames);
    }
    if (datepickerConfig.days && datepickerConfig.days.length === 7) {
      days = copyObject(datepickerConfig.days);
    }
    if (datepickerConfig.daysShort && datepickerConfig.daysShort.length === 7) {
      daysShort = copyObject(datepickerConfig.daysShort);
    }
    if (datepickerConfig.texts) {
      const textsOptions = copyObject(datepickerConfig.texts);
      texts.apply = textsOptions.apply || textsOptions.apply;
      texts.cancel = textsOptions.cancel || textsOptions.cancel;
    }
  }

  function setStartDates() {
    let startDate = dateOne || new Date();
    if (hasMinDate && isBefore(new Date(startDate), new Date(minDate))) {
      startDate = minDate;
    }
    startingDate = subtractMonths(startDate);
    selectedDate1 = dateOne;
    selectedDate2 = dateTwo;
    focusedDate = new Date(startDate);
  }

  function setSundayToFirstDayInWeek() {
    const lastDay = days.pop();
    days.unshift(lastDay);
    const lastDayShort = daysShort.pop();
    daysShort.unshift(lastDayShort);
  }

  function getMonthObject(date: Date) {
    const firstDateOfMonth = format(date, 'YYYY-MM-01');
    const year = format(date, 'YYYY');
    const monthNumber = parseInt(format(date, 'M'));
    const monthName = monthNames[monthNumber - 1];
    return {
      year,
      firstDateOfMonth,
      monthName,
      monthNumber,
      weeks: getWeeks(new Date(firstDateOfMonth))
    };
  }

  function getWeeks(date: Date) {
    const weekDayNotInMonth = { dayNumber: 0 };
    const daysInMonth = getDaysInMonth(date);
    const year = format(date, 'YYYY');
    const month = format(date, 'MM');
    let firstDayInWeek = parseInt(format(date, sundayFirst ? 'd' : 'E'));
    if (sundayFirst) {
      firstDayInWeek++;
    }
    let weeks = [];
    let week = [];
    // add empty days to get first day in correct position
    for (let s = 1; s < firstDayInWeek; s++) {
      week.push(weekDayNotInMonth);
    }
    for (let d = 0; d < daysInMonth; d++) {
      let isLastDayInMonth = d >= daysInMonth - 1;
      let dayNumber = d + 1;
      let dayNumberFull = dayNumber < 10 ? '0' + dayNumber : dayNumber;
      week.push({
        dayNumber,
        dayNumberFull: dayNumberFull,
        fullDate: year + '-' + month + '-' + dayNumberFull
      });
      if (week.length === 7) {
        weeks.push(week);
        week = [];
      } else if (isLastDayInMonth) {
        for (let i = 0; i < 7 - week.length; i++) {
          week.push(weekDayNotInMonth);
        }
        weeks.push(week);
        week = [];
      }
    }
    return weeks;
  }

  function selectDate(date) {
    if (isBeforeMinDate(date) || isAfterEndDate(date) || isDateDisabled(date)) {
      return;
    }
    if (mode === 'single') {
      selectedDate1 = date;
      closeDatepicker();
      return;
    }
    if (isSelectingDate1 || isBefore(date, new Date(selectedDate1))) {
      selectedDate1 = date;
      isSelectingDate1 = false;
      if (isBefore(new Date(selectedDate2), date)) {
        selectedDate2 = '';
      }
    } else {
      selectedDate2 = date;
      isSelectingDate1 = true;
      if (isAfter(new Date(selectedDate1), date)) {
        selectedDate1 = '';
      } else if (showActionButtons) {
        // if user has selected both dates, focus the apply button for accessibility
        applyButton.focus();
      }
      if (allDatesSelected && closeAfterSelect) {
        closeDatepicker();
      }
    }
  }

  function setHoverDate(date: string) {
    hoverDate = date;
  }

  function setFocusedDate(date: Date) {
    const formattedDate = format(date, dateFormat);
    focusedDate = formattedDate;
    // TODO: ref
    const dateElement = $refs[`date-${formattedDate}`];
    // handle .focus() on ie11 by adding a short timeout
    if (dateElement && dateElement.length) {
      setTimeout(function () {
        dateElement[0].focus();
      }, 10);
    }
  }

  function resetFocusedDate(setToFirst: boolean) {
    if (focusedDate && !isDateVisible(focusedDate)) {
      const visibleMonthIdx = setToFirst ? 0 : visibleMonths.length - 1;
      const targetMonth = visibleMonths[visibleMonthIdx];
      const monthIdx = getMonth(targetMonth);
      const year = getYear(targetMonth);
      const newFocusedDate = setYear(setMonth(new Date(focusedDate), monthIdx), year);
      focusedDate = format(newFocusedDate, dateFormat);
    }
  }

  function isToday(date) {
    return format(new Date(), dateFormat) === date;
  }

  function isSameDate(date1, date2) {
    return isSameDay(date1, date2);
  }

  function isSelected(date) {
    if (!date) {
      return;
    }
    return selectedDate1 === date || selectedDate2 === date;
  }

  function isInRange(date) {
    if (!allDatesSelected || isSingleMode) {
      return false;
    }
    return (
      (isAfter(date, new Date(selectedDate1)) && isBefore(date, new Date(selectedDate2))) ||
      (isAfter(date, new Date(selectedDate1)) && isBefore(date, new Date(hoverDate)) && !allDatesSelected)
    );
  }

  function isHoveredInRange(date) {
    if (isSingleMode || allDatesSelected) {
      return false;
    }
    return (
      (isAfter(date, new Date(selectedDate1)) && isBefore(date, new Date(hoverDate))) ||
      (isAfter(date, new Date(hoverDate)) && isBefore(date, new Date(selectedDate1)))
    );
  }

  function isBeforeMinDate(date) {
    if (!minDate) {
      return false;
    }
    return isBefore(date, new Date(minDate));
  }

  function isAfterEndDate(date) {
    if (!endDate) {
      return false;
    }
    return isAfter(date, new Date(endDate));
  }

  function isDateVisible(date) {
    if (!date) {
      return false;
    }
    const start = subDays(visibleMonths[0], 1);
    const end = addDays(lastDayOfMonth(visibleMonths[monthsToShow - 1]), 1);
    return isAfter(date, start) && isBefore(date, end);
  }

  function isDateDisabled(date) {
    if (enabledDates.length > 0) {
      return enabledDates.indexOf(date) === -1;
    } else {
      return disabledDates.indexOf(date) > -1;
    }
  }

  function customizedDateClass(date) {
    let customizedClasses = '';
    if (customizedDates.length > 0) {
      for (var i = 0; i < customizedDates.length; i++) {
        if (customizedDates[i].dates.indexOf(date) > -1)
          customizedClasses += ` asd__day--${customizedDates[i].cssClass}`;
      }
    }
    return customizedClasses;
  }

  function isDisabled(date) {
    return isDateDisabled(date) || isBeforeMinDate(date) || isAfterEndDate(date);
  }

  function previousMonth() {
    startingDate = subtractMonths(months[0].firstDateOfMonth);
    months.unshift(getMonthObject(new Date(startingDate)));
    months.splice(months.length - 1, 1);
    dispatch('previous-month', visibleMonths);
    resetFocusedDate(false);
  }

  function nextMonth() {
    startingDate = addMonthsFormatted(months[months.length - 1].firstDateOfMonth);
    months.push(getMonthObject(new Date(startingDate)));
    months.splice(0, 1);
    dispatch('next-month', visibleMonths);
    resetFocusedDate(true);
  }

  function subtractMonths(date) {
    return format(subMonths(date, 1), dateFormat);
  }

  function addMonthsFormatted(date) {
    return format(addMonths(date, 1), dateFormat);
  }

  function toggleDatepicker() {
    if (showDatepicker) {
      closeDatepicker();
    } else {
      openDatepicker();
    }
  }

  function updateMonth(offset, year, event) {
    const newMonth = event.target.value;
    const monthIdx = monthNames.indexOf(newMonth);
    const newDate = setYear(setMonth(new Date(startingDate), monthIdx), year);
    startingDate = subMonths(newDate, offset);
    generateMonths();
  }

  function updateYear(offset, monthIdx: number, event) {
    const newYear = event.target.value;
    const newDate = setYear(setMonth(new Date(startingDate), monthIdx), newYear);
    startingDate = subMonths(newDate, offset);
    generateMonths();
  }

  async function openDatepicker() {
    positionDatepicker();
    setStartDates();
    triggerElement.classList.add('datepicker-open');
    showDatepicker = true;
    this._initialDate1 = dateOne;
    this._initialDate2 = dateTwo;
    dispatch('opened');
    await tick();
    if (!inline) {
      setFocusedDate(new Date(focusedDate));
    }
  }

  function closeDatepickerCancel() {
    if (showDatepicker) {
      selectedDate1 = this._initialDate1;
      selectedDate2 = this._initialDate2;
      dispatch('cancelled');
      closeDatepicker();
    }
  }

  function closeDatepicker() {
    if (inline) {
      return;
    }
    showDatepicker = false;
    showKeyboardShortcutsMenu = false;
    triggerElement.classList.remove('datepicker-open');
    dispatch('closed');
  }

  async function openKeyboardShortcutsMenu() {
    showKeyboardShortcutsMenu = true;
    const shortcutMenuCloseBtn = keyboardShortcusMenuClose;
    await tick();
    shortcutMenuCloseBtn.focus();
  }

  async function closeKeyboardShortcutsMenu() {
    showKeyboardShortcutsMenu = false;
    await tick();
    setFocusedDate(new Date(focusedDate));
  }

  function apply() {
    dispatch('apply');
    closeDatepicker();
  }

  async function positionDatepicker() {
    const triggerWrapperElement = findAncestor(triggerElement, '.datepicker-trigger');
    triggerPosition = triggerElement.getBoundingClientRect();
    if (triggerWrapperElement) {
      triggerWrapperPosition = triggerWrapperElement.getBoundingClientRect();
    } else {
      triggerWrapperPosition = { left: 0, right: 0 };
    }
    const viewportWidthNumber = document.documentElement.clientWidth || window.innerWidth;
    viewportWidth = viewportWidthNumber + 'px';
    isMobile = viewportWidthNumber < 768;
    isTablet = viewportWidthNumber >= 768 && viewportWidthNumber <= 1024;
    showMonths = isMobile ? 1 : isTablet && monthsToShow > 2 ? 2 : monthsToShow;
    await tick();
    const datepickerWrapper = document.getElementById(wrapperId);
    if (!triggerElement || !datepickerWrapper) {
      return;
    }
    const rightPosition = triggerElement.getBoundingClientRect().left + datepickerWrapper.getBoundingClientRect().width;
    alignRight = rightPosition > viewportWidthNumber;
  }
</script>

<transition name="asd__fade">
    <div
      id="{wrapperId}"
      class="asd__wrapper"
      v-show="showDatepicker"
      class="{wrapperClasses}"
      style="{showFullscreen ? null : wrapperStyles}"
      v-click-outside="handleClickOutside"
    >
    {#if showFullscreen}
        <div class="asd__mobile-header asd__mobile-only">
           <button
             type="button"
             class="asd__mobile-close"
             on:click="{closeDatepicker}"
             aria-label="{ariaLabels.closeDatepicker}"
           >
                {#if $$slots['close-icon']}
                    <slot name="close-icon"></slot>
                {:else}
                    <div class="asd__mobile-close-icon" aria-hidden="true">X</div>
                {/if}
           </button>
           <h3>{ mobileHeader || mobileHeaderFallback }</h3>
         </div>
         <div class="asd__datepicker-header">
           <div class="asd__change-month-button asd__change-month-button--previous">
                <button on:click="{previousMonth}" type="button" aria-label="{ariaLabels.previousMonth}">
                    {#if $$slots['previous-month-icon']}
                        <slot name="previous-month-icon"></slot>
                    {:else}
                        <svg viewBox="0 0 1000 1000">
                        <path
                            d="M336.2 274.5l-210.1 210h805.4c13 0 23 10 23 23s-10 23-23 23H126.1l210.1 210.1c11 11 11 21 0 32-5 5-10 7-16 7s-11-2-16-7l-249.1-249c-11-11-11-21 0-32l249.1-249.1c21-21.1 53 10.9 32 32z"
                        ></path>
                        </svg>
                    {/if}
                </button>
           </div>
           <div class="asd__change-month-button asd__change-month-button--next">
             <button on:click="{nextMonth}" type="button" aria-label="{ariaLabels.nextMonth}">
                {#if $$slots['next-month-icon']}
                    <slot  name="next-month-icon"></slot>
                {:else}
                     <svg viewBox="0 0 1000 1000">
                       <path
                         d="M694.4 242.4l249.1 249.1c11 11 11 21 0 32L694.4 772.7c-5 5-10 7-16 7s-11-2-16-7c-11-11-11-21 0-32l210.1-210.1H67.1c-13 0-23-10-23-23s10-23 23-23h805.4L662.4 274.5c-21-21.1 11-53.1 32-32.1z"
                       ></path>
                     </svg>
                {/if}
             </button>
           </div>
           <!-- TODO: do this -->
           <div
             class="asd__days-legend"
             v-for="(month, index) in showMonths"
             :key="month"
             :style="[monthWidthStyles, {left: (width * index) + 'px'}]"
           >
             <div class="asd__day-title" v-for="(day, index) in daysShort" :key="index">{{ day }}</div>
           </div>
         </div>
   
         <div class="asd__inner-wrapper" style="{innerStyles}">
           <transition-group name="asd__list-complete" tag="div">
             <div
               v-for="(month, monthIndex) in months"
               :key="month.firstDateOfMonth"
               class="asd__month"
               :class="{'asd__month--hidden': monthIndex === 0 || monthIndex > showMonths}"
               :style="monthWidthStyles"
             >
               <div class="asd__month-name">
                 <select
                   v-if="showMonthYearSelect"
                   v-model="month.monthName"
                   class="asd__month-year-select"
                   :tabindex="monthIndex === 0 || monthIndex > showMonths ? -1 : 0"
                   @change="updateMonth(monthIndex, month.year, $event)"
                   v-resize-select
                 >
                   <option
                     v-for="(monthName, idx) in monthNames"
                     :value="monthName"
                     :disabled="isMonthDisabled(month.year, idx)"
                     :key="`month-${monthIndex}-${monthName}`"
                   >{{ monthName }}</option>
                 </select>
                 <span v-else>{{ month.monthName }}</span>
   
                 <select
                   v-if="showMonthYearSelect"
                   class="asd__month-year-select"
                   :tabindex="monthIndex === 0 || monthIndex > showMonths ? -1 : 0"
                   v-model="month.year"
                   @change="updateYear(monthIndex, month.monthNumber - 1, $event)"
                 >
                   <option
                     v-if="years.indexOf(month.year) === -1"
                     :value="month.year"
                     :key="`month-${monthIndex}-${year}`"
                     :disabled="true"
                   >{{ month.year }}</option>
                   <option
                     v-for="year in years"
                     :value="year"
                     :key="`month-${monthIndex}-${year}`"
                   >{{ year }}</option>
                 </select>
                 <span v-else>{{ month.year }}</span>
               </div>
   
               <table class="asd__month-table" role="presentation">
                 <tbody>
                   <tr class="asd__week" v-for="(week, index) in month.weeks" :key="index">
                     <td
                       class="asd__day"
                       v-for="({fullDate, dayNumber}, index) in week"
                       :key="index + '_' + dayNumber"
                       :data-date="fullDate"
                       :ref="`date-${fullDate}`"
                       :tabindex="isDateVisible(fullDate) && isSameDate(focusedDate, fullDate) ? 0 : -1"
                       :aria-label="isDateVisible(fullDate) ? getAriaLabelForDate(fullDate) : false"
                       :class="[{
                         'asd__day--enabled': dayNumber !== 0,
                         'asd__day--empty': dayNumber === 0,
                         'asd__day--disabled': isDisabled(fullDate),
                         'asd__day--selected': fullDate && (selectedDate1 === fullDate || selectedDate2 === fullDate),
                         'asd__day--in-range': isInRange(fullDate),
                         'asd__day--today': fullDate && isToday(fullDate),
                         'asd__day--hovered': isHoveredInRange(fullDate),
                         'asd__selected-date-one': fullDate && fullDate === selectedDate1,
                         'asd__selected-date-two': fullDate && fullDate === selectedDate2,
                       }, customizedDateClass(fullDate)]"
                       :style="getDayStyles(fullDate)"
                       @mouseover="() => { setHoverDate(fullDate) }"
                     >
                       <button
                         class="asd__day-button"
                         type="button"
                         v-if="dayNumber"
                         tabindex="-1"
                         :date="fullDate"
                         :disabled="isDisabled(fullDate)"
                         @click="() => { selectDate(fullDate) }"
                       >{{ dayNumber }}</button>
                     </td>
                   </tr>
                 </tbody>
               </table>
             </div>
           </transition-group>
           <div
             v-if="showShortcutsMenuTrigger"
             :class="{ 'asd__keyboard-shortcuts-menu': true, 'asd__keyboard-shortcuts-show': showKeyboardShortcutsMenu}"
             :style="keyboardShortcutsMenuStyles"
           >
             <div class="asd__keyboard-shortcuts-title">{{ texts.keyboardShortcuts }}</div>
             <button
               class="asd__keyboard-shortcuts-close"
               ref="keyboard-shortcus-menu-close"
               tabindex="0"
               @click="closeKeyboardShortcutsMenu"
               :aria-label="ariaLabels.closeKeyboardShortcutsMenu"
             >
               <slot v-if="$slots['close-shortcuts-icon']" name="close-shortcuts-icon"></slot>
               <div v-else class="asd__mobile-close-icon" aria-hidden="true">X</div>
             </button>
             <ul class="asd__keyboard-shortcuts-list">
               <li v-for="(shortcut, i) in keyboardShortcuts" :key="i">
                 <span
                   class="asd__keyboard-shortcuts-symbol"
                   :aria-label="shortcut.symbolDescription"
                 >{{ shortcut.symbol }}</span>
                 {{ shortcut.label }}
               </li>
             </ul>
           </div>
         </div>
         <div class="asd__action-buttons" v-if="mode !== 'single' && showActionButtons">
           <button @click="closeDatepickerCancel" type="button">{{ texts.cancel }}</button>
           <button
             ref="apply-button"
             @click="apply"
             :style="{color: colors.selected}"
             type="button"
           >{{ texts.apply }}</button>
         </div>
         <div v-if="showShortcutsMenuTrigger" class="asd__keyboard-shortcuts-trigger-wrapper">
           <button
             class="asd__keyboard-shortcuts-trigger"
             :aria-label="ariaLabels.openKeyboardShortcutsMenu"
             tabindex="0"
             @click="openKeyboardShortcutsMenu"
           >
             <span>?</span>
           </button>
         </div>
       </div>
    {/if}
</transition>