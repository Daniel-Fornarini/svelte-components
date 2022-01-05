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

  export interface Month {
    year: string;
    firstDateOfMonth: string;
    monthName: string;
    monthNumber: number;
    weeks: any[];
  }

  export const datepickerConfig: DatepickerOptions = {};
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
  import { clickOutside } from '../directives/clickOutside';
  //   import ResizeSelect from '../directives/ResizeSelect';
  import { createEventDispatcher, onDestroy, onMount, tick } from 'svelte';
  import { fade } from 'svelte/transition';
  import { parseISO, toDate } from 'date-fns';

  export let triggerElementId: string;
  export let dateOne: string | Date = '';
  export let dateTwo: string | Date = '';
  export let minDate: string | Date = '';
  export let endDate: string | Date = '';
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
  let dateFormat: string = 'yyyy-MM-dd';
  let dateLabelFormat: string = 'dddd, MMMM d, yyyy';
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
    chooseDate: (date: string) => date,
    chooseStartDate: (date: string) => `Choose ${date} as your start date.`,
    chooseEndDate: (date: string) => `Choose ${date} as your end date.`,
    selectedDate: (date: string) => `Selected. ${date}`,
    unavailableDate: (date: string) => `Not available. ${date}`,
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
  let months: Month[] = [];
  let years: string[] = [];
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
  let triggerElement!: HTMLElement;
  let fullDateRefs: Record<string, HTMLElement> = {};
  let _initialDate1: Date | string;
  let _initialDate2: Date | string;
  let windowInnerWidth: number;

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
    'z-index': inline ? '0' : '100'
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
  $: datePropsCompound = (dateOne as string) + (dateTwo as string);
  $: isDateTwoBeforeDateOne = !dateTwo
    ? false
    : isBefore(normalizeDate(dateTwo), normalizeDate(dateOne));
  $: visibleMonths = getVisibleMonths(months, showMonths);
  function getVisibleMonths(monthsAtr: any[], showMonthsAtr: number) {
    const firstMonthArray = monthsAtr.filter((m, index) => index > 0);
    const numberOfMonthsArray = [];
    for (let i = 0; i < showMonthsAtr; i++) {
      numberOfMonthsArray.push(i);
    }
    return numberOfMonthsArray.map((_, index) =>
      normalizeDate(firstMonthArray[index]?.firstDateOfMonth)
    );
  }

  $: onSelectedDate1Change(selectedDate1);
  function onSelectedDate1Change(newValue: Date | string) {
    let newDate = !newValue || newValue === '' ? '' : format(normalizeDate(newValue), dateFormat);
    dispatch('date-one-selected', newDate);
  }
  $: onSelectedDate2Change(selectedDate2);
  function onSelectedDate2Change(newValue: Date | string) {
    let newDate = !newValue || newValue === '' ? '' : format(normalizeDate(newValue), dateFormat);
    dispatch('date-two-selected', newDate);
  }
  $: onModeChange(mode);
  function onModeChange(newValue: string) {
    setStartDates();
  }
  $: onMinDateChange(minDate);
  function onMinDateChange(newValue: Date | string) {
    setStartDates();
    generateMonths();
    generateYears();
  }
  $: onEndDateChange(endDate);
  function onEndDateChange(newValue: Date | string) {
    generateYears();
  }
  $: onDatePropsCompoundChange(datePropsCompound);
  function onDatePropsCompoundChange(newValue: string) {
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
  function onTriggerChange(newValue: boolean) {
    if (newValue) {
      setTimeout(() => {
        openDatepicker();
      }, 0);
    }
  }

  function handleWindowResizeEvent(
    event: UIEvent & {
      currentTarget: EventTarget & Window;
    }
  ) {
    debounce(() => {
      positionDatepicker();
      setStartDates();
    }, 200);
  }

  function handleWindowClickEvent(event: MouseEvent) {
    if ((event.target as HTMLElement)?.id === triggerElementId) {
      event.stopPropagation();
      event.preventDefault();
      toggleDatepicker();
    }
  }

  onMount(() => {
    setupDatepicker();
    if (sundayFirst) {
      setSundayToFirstDayInWeek();
    }
    viewportWidth = windowInnerWidth + 'px';
    isMobile = windowInnerWidth < 768;
    isTablet = windowInnerWidth >= 768 && windowInnerWidth <= 1024;
    triggerElement = isTest
      ? document.createElement('input')
      : document.getElementById(triggerElementId)!;
    setStartDates();
    generateMonths();
    generateYears();
    if (startOpen || inline) {
      openDatepicker();
    }
    el.addEventListener('keyup', handleKeyboardInput);
    el.addEventListener('keydown', trapKeyboardInput);
    triggerElement.addEventListener('keyup', handleTriggerInput);
    triggerElement.addEventListener('click', handleWindowClickEvent);
  });

  onDestroy(() => {
    // window.removeEventListener('resize', this._handleWindowResizeEvent);
    // window.removeEventListener('click', this._handleWindowClickEvent);
    // el.removeEventListener('keyup', handleKeyboardInput);
    // el.removeEventListener('keydown', trapKeyboardInput);
    // triggerElement.removeEventListener('keyup', handleTriggerInput);
    // triggerElement.removeEventListener('click', this._handleWindowClickEvent);
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

  function getAriaLabelForDate(date: Date | string) {
    date = normalizeDate(date);
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

  function handleClickOutside(event: CustomEvent<any>) {
    if ((event.target as HTMLElement)?.id === triggerElementId || !showDatepicker || inline) {
      return;
    }
    closeDatepicker();
  }

  function shouldHandleInput(event: KeyboardEvent, key: number) {
    return event.keyCode === key && (!event.shiftKey || event.keyCode === 191) && showDatepicker;
  }

  function handleTriggerInput(event: KeyboardEvent) {
    if (mode === 'single') {
      setDateFromText((event.target as HTMLInputElement)?.value);
    }
  }

  function trapKeyboardInput(event: KeyboardEvent) {
    // prevent keys that are used as keyboard shortcuts from propagating out of this element
    // except for the enter key, which is needed to activate buttons
    const shortcutKeyCodes = Object.keys(keys).map((key) => (keys as any)[key]);
    shortcutKeyCodes.splice(shortcutKeyCodes.indexOf(13), 1);
    const shouldPreventDefault = shortcutKeyCodes.indexOf(event.keyCode) > -1;
    if (shouldPreventDefault) event.preventDefault();
  }

  function handleKeyboardInput(event: KeyboardEvent) {
    if (shouldHandleInput(event, keys.esc)) {
      if (showKeyboardShortcutsMenu) {
        closeKeyboardShortcutsMenu();
      } else {
        closeDatepicker();
      }
    } else if (showKeyboardShortcutsMenu) {
      // if keyboard shortcutsMenu is open, then esc is the only key we want to have fire events
    } else if (shouldHandleInput(event, keys.arrowDown)) {
      const newDate = addWeeks(normalizeDate(focusedDate), 1);
      const changeMonths = !isSameMonth(newDate, normalizeDate(focusedDate));
      setFocusedDate(newDate);
      if (changeMonths) nextMonth();
    } else if (shouldHandleInput(event, keys.arrowUp)) {
      const newDate = subWeeks(normalizeDate(focusedDate), 1);
      const changeMonths = !isSameMonth(newDate, normalizeDate(focusedDate));
      setFocusedDate(newDate);
      if (changeMonths) previousMonth();
    } else if (shouldHandleInput(event, keys.arrowRight)) {
      const newDate = addDays(normalizeDate(focusedDate), 1);
      const changeMonths = !isSameMonth(newDate, normalizeDate(focusedDate));
      setFocusedDate(newDate);
      if (changeMonths) nextMonth();
    } else if (shouldHandleInput(event, keys.arrowLeft)) {
      const newDate = subDays(normalizeDate(focusedDate), 1);
      const changeMonths = !isSameMonth(newDate, normalizeDate(focusedDate));
      setFocusedDate(newDate);
      if (changeMonths) previousMonth();
    } else if (shouldHandleInput(event, keys.enter)) {
      // on enter key, only select the date if a date is currently in focus
      const target: HTMLElement | null = event.target as HTMLElement;
      if (!showKeyboardShortcutsMenu && target && target.tagName === 'TD') {
        selectDate(focusedDate);
      }
    } else if (shouldHandleInput(event, keys.pgUp)) {
      setFocusedDate(subMonths(normalizeDate(focusedDate), 1));
      previousMonth();
    } else if (shouldHandleInput(event, keys.pgDn)) {
      setFocusedDate(addMonths(normalizeDate(focusedDate), 1));
      nextMonth();
    } else if (shouldHandleInput(event, keys.home)) {
      const newDate = startOfWeek(normalizeDate(focusedDate), {
        weekStartsOn: sundayFirst ? 0 : 1
      });
      const changeMonths = !isSameMonth(newDate, normalizeDate(focusedDate));
      setFocusedDate(newDate);
      if (changeMonths) previousMonth();
    } else if (shouldHandleInput(event, keys.end)) {
      const newDate = endOfWeek(normalizeDate(focusedDate), {
        weekStartsOn: sundayFirst ? 0 : 1
      });
      const changeMonths = !isSameMonth(newDate, normalizeDate(focusedDate));
      setFocusedDate(newDate);
      if (changeMonths) nextMonth();
    } else if (shouldHandleInput(event, keys.questionMark)) {
      openKeyboardShortcutsMenu();
    }
  }

  function setDateFromText(value: string) {
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
    // Date
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
    startingDate = subMonths(normalizeDate(formattedDate), 1);
    generateMonths();
    generateYears();
    selectDate(formattedDate);
  }

  function isMonthDisabled(year: number, monthIndex: number) {
    // Date
    const monthDate = new Date(year, monthIndex);
    if (hasMinDate && isBefore(monthDate, startOfMonth(normalizeDate(minDate)))) {
      return true;
    }
    return isAfterEndDate(monthDate);
  }

  function generateMonths() {
    months = [];
    let currentMonth = startingDate;
    for (let i = 0; i < showMonths + 2; i++) {
      months.push(getMonthObject(normalizeDate(currentMonth)));
      currentMonth = addMonthsFormatted(currentMonth);
    }
  }

  function generateYears() {
    if (!showMonthYearSelect) return;
    years = [];
    const currentYear = getYear(normalizeDate(startingDate));
    const startYear = minDate ? getYear(normalizeDate(minDate)) : currentYear - yearsForSelect;
    const endYear = endDate ? getYear(normalizeDate(endDate)) : currentYear + yearsForSelect;
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
    // Date
    let startDate = dateOne || new Date();
    if (hasMinDate && isBefore(normalizeDate(startDate), normalizeDate(minDate))) {
      startDate = minDate;
    }
    startingDate = subtractMonths(startDate);
    selectedDate1 = dateOne;
    selectedDate2 = dateTwo;
    focusedDate = normalizeDate(startDate);
  }

  function setSundayToFirstDayInWeek() {
    const lastDay = days.pop()!;
    days.unshift(lastDay);
    const lastDayShort = daysShort.pop()!;
    daysShort.unshift(lastDayShort);
  }

  function getMonthObject(date: Date) {
    const firstDateOfMonth = format(date, 'yyyy-MM-01');
    const year = format(date, 'yyyy');
    const monthNumber = parseInt(format(date, 'M'));
    const monthName = monthNames[monthNumber - 1];
    return {
      year,
      firstDateOfMonth,
      monthName,
      monthNumber,
      weeks: getWeeks(normalizeDate(firstDateOfMonth))
    };
  }

  function getWeeks(date: Date) {
    console.log(date);
    const weekDayNotInMonth = { dayNumber: 0 };
    const daysInMonth = getDaysInMonth(date);
    const year = format(date, 'yyyy');
    const month = format(date, 'MM');
    let firstDayInWeek = parseInt(format(date, 'i'));
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

  function selectDate(date: Date | string) {
    date = normalizeDate(date);
    if (isBeforeMinDate(date) || isAfterEndDate(date) || isDateDisabled(date)) {
      return;
    }
    if (mode === 'single') {
      selectedDate1 = date;
      closeDatepicker();
      return;
    }
    if (isSelectingDate1 || isBefore(date, normalizeDate(selectedDate1))) {
      selectedDate1 = date;
      isSelectingDate1 = false;
      if (isBefore(normalizeDate(selectedDate2), date)) {
        selectedDate2 = '';
      }
    } else {
      selectedDate2 = date;
      isSelectingDate1 = true;
      if (isAfter(normalizeDate(selectedDate1), date)) {
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
    const dateElement = fullDateRefs[`date-${formattedDate}`];
    // handle .focus() on ie11 by adding a short timeout
    if (dateElement) {
      setTimeout(function () {
        dateElement.focus();
      }, 10);
    }
  }

  function resetFocusedDate(setToFirst: boolean) {
    if (focusedDate && !isDateVisible(focusedDate)) {
      const visibleMonthIdx = setToFirst ? 0 : visibleMonths.length - 1;
      const targetMonth = visibleMonths[visibleMonthIdx];
      const monthIdx = getMonth(targetMonth);
      const year = getYear(targetMonth);
      const newFocusedDate = setYear(setMonth(normalizeDate(focusedDate), monthIdx), year);
      focusedDate = format(newFocusedDate, dateFormat);
    }
  }

  function isToday(date: string) {
    // Date
    return format(new Date(), dateFormat) === date;
  }

  function isSameDate(date1: Date | string, date2: Date | string) {
    date1 = normalizeDate(date1);
    date2 = normalizeDate(date2);
    return isSameDay(date1, date2);
  }

  function isSelected(date: Date | string) {
    date = normalizeDate(date);
    if (!date) {
      return;
    }
    const s1 = normalizeDate(selectedDate1);
    const s2 = normalizeDate(selectedDate2);
    return isSameDate(s1, date) || isSameDate(s2, date);
  }

  function isInRange(date: Date | string) {
    date = normalizeDate(date);
    if (!allDatesSelected || isSingleMode) {
      return false;
    }
    return (
      (isAfter(date, normalizeDate(selectedDate1)) &&
        isBefore(date, normalizeDate(selectedDate2))) ||
      (isAfter(date, normalizeDate(selectedDate1)) &&
        isBefore(date, normalizeDate(hoverDate)) &&
        !allDatesSelected)
    );
  }

  function isHoveredInRange(date: Date | string) {
    date = normalizeDate(date);
    if (isSingleMode || allDatesSelected) {
      return false;
    }
    return (
      (isAfter(date, normalizeDate(selectedDate1)) && isBefore(date, normalizeDate(hoverDate))) ||
      (isAfter(date, normalizeDate(hoverDate)) && isBefore(date, normalizeDate(selectedDate1)))
    );
  }

  function isBeforeMinDate(date: Date | string) {
    date = normalizeDate(date);
    if (!minDate) {
      return false;
    }
    return isBefore(date, normalizeDate(minDate));
  }

  function isAfterEndDate(date: Date | string) {
    date = normalizeDate(date);
    if (!endDate) {
      return false;
    }
    return isAfter(date, normalizeDate(endDate));
  }

  function isDateVisible(date: Date | string) {
    date = normalizeDate(date);
    if (!date) {
      return false;
    }
    const start = subDays(visibleMonths[0], 1);
    const end = addDays(lastDayOfMonth(visibleMonths[monthsToShow - 1]), 1);
    // console.group('date')
    // console.log(date);
    // console.log(start);
    // console.log(end);
    // console.groupEnd()
    return isAfter(date, start) && isBefore(date, end);
  }

  function isDateDisabled(date: Date | string) {
    date = normalizeDate(date);
    if (enabledDates.length > 0) {
      return enabledDates.indexOf(date) === -1;
    } else {
      return disabledDates.indexOf(date) > -1;
    }
  }

  function customizedDateClass(date: Date | string) {
    let customizedClasses = '';
    if (customizedDates.length > 0) {
      for (var i = 0; i < customizedDates.length; i++) {
        if (customizedDates[i].dates.indexOf(date) > -1)
          customizedClasses += ` asd__day--${customizedDates[i].cssClass}`;
      }
    }
    return customizedClasses;
  }

  function isDisabled(date: Date | string) {
    date = normalizeDate(date);
    return isDateDisabled(date) || isBeforeMinDate(date) || isAfterEndDate(date);
  }

  function previousMonth() {
    startingDate = subtractMonths(months[0].firstDateOfMonth);
    months.unshift(getMonthObject(normalizeDate(startingDate)));
    months.splice(months.length - 1, 1);
    dispatch('previous-month', visibleMonths);
    resetFocusedDate(false);
  }

  function nextMonth() {
    console.log(JSON.stringify(visibleMonths));
    console.log(months);
    startingDate = addMonthsFormatted(months[months.length - 1].firstDateOfMonth);
    months.push(getMonthObject(normalizeDate(startingDate)));
    months.splice(0, 1);
    dispatch('next-month', visibleMonths);
    resetFocusedDate(true);
    console.log(JSON.stringify(visibleMonths));
    console.log(months);
  }

  function subtractMonths(date: Date | string) {
    date = normalizeDate(date);
    return format(subMonths(date, 1), dateFormat);
  }

  function addMonthsFormatted(date: Date | string) {
    return format(addMonths(normalizeDate(date), 1), dateFormat);
  }

  function toggleDatepicker() {
    if (showDatepicker) {
      closeDatepicker();
    } else {
      openDatepicker();
    }
  }

  function updateMonth(offset: number, year: string, event: Event) {
    const newMonth = (event.target as HTMLSelectElement)?.value;
    const monthIdx = monthNames.indexOf(newMonth);
    const newDate = setYear(setMonth(normalizeDate(startingDate), monthIdx), +year);
    startingDate = subMonths(newDate, offset);
    generateMonths();
  }

  function updateYear(offset: number, monthIdx: number, event: Event) {
    const newYear = (event.target as HTMLSelectElement)?.value;
    const newDate = setYear(setMonth(normalizeDate(startingDate), monthIdx), +newYear);
    startingDate = subMonths(newDate, offset);
    generateMonths();
  }

  async function openDatepicker() {
    positionDatepicker();
    setStartDates();
    triggerElement!.classList.add('datepicker-open');
    showDatepicker = true;
    _initialDate1 = dateOne;
    _initialDate2 = dateTwo;
    dispatch('opened');
    await tick();
    if (!inline) {
      setFocusedDate(normalizeDate(focusedDate));
    }
  }

  function closeDatepickerCancel() {
    if (showDatepicker) {
      selectedDate1 = _initialDate1;
      selectedDate2 = _initialDate2;
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
    triggerElement!.classList.remove('datepicker-open');
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
    setFocusedDate(normalizeDate(focusedDate));
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
    const viewportWidthNumber = document.documentElement.clientWidth || windowInnerWidth;
    viewportWidth = viewportWidthNumber + 'px';
    isMobile = viewportWidthNumber < 768;
    isTablet = viewportWidthNumber >= 768 && viewportWidthNumber <= 1024;
    showMonths = isMobile ? 1 : isTablet && monthsToShow > 2 ? 2 : monthsToShow;
    await tick();
    const datepickerWrapper = document.getElementById(wrapperId);
    if (!triggerElement || !datepickerWrapper) {
      return;
    }
    const rightPosition =
      triggerElement.getBoundingClientRect().left + datepickerWrapper.getBoundingClientRect().width;
    alignRight = rightPosition > viewportWidthNumber;
  }

  function asdListComplete(node: HTMLElement, params: any) {
    return {
      delay: params.delay || 0,
      duration: params.duration || 400,
      css: (t: number, u: number) => `transform: translateY(${t * 30}px)`
    };
  }

  function stylesToString(styles: any) {
    const arrayStyles = Object.entries(styles);
    const stringStyles = [];
    for (const style of arrayStyles) {
      stringStyles.push(`${style[0]}: ${style[1]};`);
    }
    return stringStyles.join(' ');
  }

  function classesToString(classes: any) {
    const arrayClasses = Object.entries(classes);
    const stringClasses = [];
    for (const c of arrayClasses) {
      if (!!c[1] === true) {
        stringClasses.push(c[0]);
      }
    }
    return stringClasses.join(' ');
  }

  function numberToArray(number: number) {
    const numbers = [];
    for (let i = 0; i < number; i++) {
      numbers.push(i + 1);
    }
    return numbers;
  }

  function normalizeDate(date: string | Date): Date {
    if (typeof date === 'string') {
      return parseISO(date);
    }
    return date;
  }
</script>

<!-- TODO: v-show -->

<svelte:window
  bind:innerWidth={windowInnerWidth}
  on:resize={(event) => handleWindowResizeEvent(event)}
/>

<div
  id={wrapperId}
  class="asd__wrapper {classesToString(wrapperClasses)}"
  class:hidden={!showDatepicker}
  style={showFullscreen ? null : stylesToString(wrapperStyles)}
  bind:this={el}
  on:keyup={handleKeyboardInput}
  on:keydown={trapKeyboardInput}
  transition:fade
  use:clickOutside
  on:click_outside={handleClickOutside}
>
  {#if showFullscreen}
    <div class="asd__mobile-header asd__mobile-only">
      <button
        type="button"
        class="asd__mobile-close"
        on:click={closeDatepicker}
        aria-label={ariaLabels.closeDatepicker}
      >
        {#if $$slots['close-icon']}
          <slot name="close-icon" />
        {:else}
          <div class="asd__mobile-close-icon" aria-hidden="true">X</div>
        {/if}
      </button>
      <h3>{mobileHeader || mobileHeaderFallback}</h3>
    </div>
  {/if}
  <div class="asd__datepicker-header">
    <div class="asd__change-month-button asd__change-month-button--previous">
      <button on:click={previousMonth} type="button" aria-label={ariaLabels.previousMonth}>
        {#if $$slots['previous-month-icon']}
          <slot name="previous-month-icon" />
        {:else}
          <svg viewBox="0 0 1000 1000">
            <path
              d="M336.2 274.5l-210.1 210h805.4c13 0 23 10 23 23s-10 23-23 23H126.1l210.1 210.1c11 11 11 21 0 32-5 5-10 7-16 7s-11-2-16-7l-249.1-249c-11-11-11-21 0-32l249.1-249.1c21-21.1 53 10.9 32 32z"
            />
          </svg>
        {/if}
      </button>
    </div>
    <div class="asd__change-month-button asd__change-month-button--next">
      <button on:click={nextMonth} type="button" aria-label={ariaLabels.nextMonth}>
        {#if $$slots['next-month-icon']}
          <slot name="next-month-icon" />
        {:else}
          <svg viewBox="0 0 1000 1000">
            <path
              d="M694.4 242.4l249.1 249.1c11 11 11 21 0 32L694.4 772.7c-5 5-10 7-16 7s-11-2-16-7c-11-11-11-21 0-32l210.1-210.1H67.1c-13 0-23-10-23-23s10-23 23-23h805.4L662.4 274.5c-21-21.1 11-53.1 32-32.1z"
            />
          </svg>
        {/if}
      </button>
    </div>
    {#each numberToArray(showMonths) as month, i (month)}
      <div
        class="asd__days-legend"
        style="{stylesToString(monthWidthStyles)} {`left: ${width * i}px`}"
      >
        {#each daysShort as day, i (i)}
          <div class="asd__day-title">{day}</div>
        {/each}
      </div>
    {/each}
  </div>
  <div class="asd__inner-wrapper" style={stylesToString(innerStyles)}>
    <div transition:asdListComplete>
      {#each months as month, monthIndex (month.firstDateOfMonth)}
        <!-- content here -->
        <div
          class="asd__month {monthIndex === 0 || monthIndex > showMonths
            ? 'asd__month--hidden'
            : ''}"
          style={stylesToString(monthWidthStyles)}
        >
          <div class="asd__month-name">
            {#if showMonthYearSelect}
              <!-- TODO: v-resize-select -->
              <select
                bind:value={month.monthName}
                class="asd__month-year-select"
                tabindex={monthIndex === 0 || monthIndex > showMonths ? -1 : 0}
                on:change={(event) => updateMonth(monthIndex, month.year, event)}
              >
                {#each monthNames as monthName, idx (`month-${monthIndex}-${monthName}`)}
                  <!-- content here -->
                  <option value={monthName} disabled={isMonthDisabled(+month.year, idx)}
                    >{monthName}</option
                  >
                {/each}
              </select>
            {:else}
              <span>{month.monthName}</span>
            {/if}
            {#if showMonthYearSelect}
              <select
                class="asd__month-year-select"
                tabindex={monthIndex === 0 || monthIndex > showMonths ? -1 : 0}
                bind:value={month.year}
                on:change={(event) => updateYear(monthIndex, month.monthNumber - 1, event)}
              >
                {#if years.indexOf(month.year) === -1}
                  <!-- TODO: :key="`month-${monthIndex}-${year}`" -->
                  <option value={month.year} disabled={true}>{month.year}</option>
                {/if}
                {#each years as year (`month-${monthIndex}-${year}`)}
                  <option value={year}>{year}</option>
                {/each}
              </select>
            {:else}
              <span>{month.year}</span>
            {/if}
          </div>
          <table class="asd__month-table" role="presentation">
            <tbody>
              {#each month.weeks as week, index (index)}
                <tr class="asd__week">
                  {#each week as { fullDate, dayNumber }, index (`${index}_${dayNumber}`)}
                    <td
                      class="asd__day {classesToString(customizedDateClass(fullDate))}"
                      data-date={fullDate}
                      bind:this={fullDateRefs[`date-${fullDate}`]}
                      tabindex={isDateVisible(fullDate) && isSameDate(focusedDate, fullDate)
                        ? 0
                        : -1}
                      aria-label={isDateVisible(fullDate) ? getAriaLabelForDate(fullDate) : false}
                      class:asd__day--enabled={dayNumber !== 0}
                      class:asd__day--empty={dayNumber === 0}
                      class:asd__day--disabled={isDisabled(fullDate)}
                      class:asd__day--selected={fullDate &&
                        (selectedDate1 === fullDate || selectedDate2 === fullDate)}
                      class:asd__day--in-range={isInRange(fullDate)}
                      class:asd__day--today={fullDate && isToday(fullDate)}
                      class:asd__day--hovered={isHoveredInRange(fullDate)}
                      class:asd__selected-date-one={fullDate && fullDate === selectedDate1}
                      class:asd__selected-date-two={fullDate && fullDate === selectedDate2}
                      style={stylesToString(getDayStyles(fullDate))}
                      on:mouseover={() => setHoverDate(fullDate)}
                      on:focus={() => {}}
                    >
                      {#if dayNumber}
                        <!-- FIXME: date={fullDate} -->
                        <button
                          class="asd__day-button"
                          type="button"
                          tabindex="-1"
                          disabled={isDisabled(fullDate)}
                          on:click={() => selectDate(fullDate)}>{dayNumber}</button
                        >
                      {/if}
                    </td>
                  {/each}
                </tr>
              {/each}
            </tbody>
          </table>
        </div>
      {/each}
    </div>
    {#if showShortcutsMenuTrigger}
      <div
        class="asd__keyboard-shortcuts-menu {showKeyboardShortcutsMenu
          ? 'asd__keyboard-shortcuts-show'
          : ''}"
        style={stylesToString(keyboardShortcutsMenuStyles)}
      >
        <div class="asd__keyboard-shortcuts-title">{texts.keyboardShortcuts}</div>
        <button
          class="asd__keyboard-shortcuts-close"
          bind:this={keyboardShortcusMenuClose}
          tabindex="0"
          on:click={closeKeyboardShortcutsMenu}
          aria-label={ariaLabels.closeKeyboardShortcutsMenu}
        >
          {#if $$slots['close-shortcuts-icon']}
            <slot name="close-shortcuts-icon" />
          {:else}
            <div class="asd__mobile-close-icon" aria-hidden="true">X</div>
          {/if}
        </button>
        <ul class="asd__keyboard-shortcuts-list">
          {#each keyboardShortcuts as shortcut, i (i)}
            <li>
              <span class="asd__keyboard-shortcuts-symbol" aria-label={shortcut.symbolDescription}
                >{shortcut.symbol}</span
              >
              {shortcut.label}
            </li>
          {/each}
        </ul>
      </div>
    {/if}
  </div>
  {#if mode !== 'single' && showActionButtons}
    <div class="asd__action-buttons">
      <button on:click={closeDatepickerCancel} type="button">{texts.cancel}</button>
      <button
        bind:this={applyButton}
        on:click={apply}
        style={`color: ${colors.selected}`}
        type="button">{texts.apply}</button
      >
    </div>
  {/if}
  {#if showShortcutsMenuTrigger}
    <div class="asd__keyboard-shortcuts-trigger-wrapper">
      <button
        class="asd__keyboard-shortcuts-trigger"
        aria-label={ariaLabels.openKeyboardShortcutsMenu}
        tabindex="0"
        on:click={openKeyboardShortcutsMenu}
      >
        <span>?</span>
      </button>
    </div>
  {/if}
</div>

<style lang="scss" global>
  $tablet: 768px;
  $color-gray: rgba(0, 0, 0, 0.2);
  $border-normal: 1px solid $color-gray;
  $border: 1px solid #e4e7e7;
  $transition-time: 0.3s;
  .hidden {
    visibility: hidden;
  }
  .datepicker-trigger {
    position: relative;
    overflow: visible;
  }
  .asd {
    &__wrapper {
      border: $border-normal;
      white-space: nowrap;
      text-align: center;
      overflow: hidden;
      background-color: white;
      *,
      *:after,
      *:before {
        box-sizing: border-box;
      }
      &--full-screen {
        position: fixed;
        top: 0;
        right: 0;
        bottom: 0;
        left: 0;
        border: none;
        z-index: 100;
      }
    }
    &__inner-wrapper {
      transition: all $transition-time ease;
      position: relative;
    }
    &__datepicker-header {
      position: relative;
    }
    &__keyboard-shortcuts-trigger-wrapper {
      position: relative;
    }
    &__keyboard-shortcuts-trigger {
      background-color: transparent;
      cursor: pointer;
      position: absolute;
      bottom: 0px;
      right: 0px;
      font: inherit;
      border-width: 26px 33px 0px 0px;
      border-top: 26px solid transparent;
      border-right: 33px solid rgb(0, 166, 153);
      span {
        color: rgb(255, 255, 255);
        position: absolute;
        bottom: 0px;
        right: -28px;
      }
    }
    &__keyboard-shortcuts-show {
      display: block !important;
    }
    &__keyboard-shortcuts-close {
      background-color: transparent;
      border: none;
      position: absolute;
      top: 7px;
      right: 5px;
      padding: 5px;
      z-index: 100;
      cursor: pointer;
    }
    &__keyboard-shortcuts-menu {
      display: none;
      position: absolute;
      top: 0px;
      bottom: 0px;
      right: 0px;
      z-index: 10;
      overflow: auto;
      background: rgb(255, 255, 255);
      border-width: 1px;
      border-style: solid;
      border-color: rgb(219, 219, 219);
      border-image: initial;
      border-radius: 2px;
      padding: 22px;
      margin: 33px;
      text-align: left;
    }
    &__keyboard-shortcuts-title {
      font-size: 16px;
      font-weight: bold;
      margin: 0px;
    }
    &__keyboard-shortcuts-list {
      list-style: none;
      margin: 6px 0px;
      padding: 0px;
      white-space: initial;
    }
    &__keyboard-shortcuts-symbol {
      font-family: monospace;
      font-size: 12px;
      text-transform: uppercase;
      background: rgb(242, 242, 242);
      padding: 2px 6px;
      margin-right: 4px;
    }
    &__change-month-button {
      position: absolute;
      top: 12px;
      z-index: 10;
      background: white;
      &--previous {
        left: 0;
        padding-left: 15px;
      }
      &--next {
        right: 0;
        padding-right: 15px;
      }
      > button {
        background-color: white;
        border: $border;
        border-radius: 3px;
        padding: 4px 8px;
        cursor: pointer;
        &:hover {
          border: 1px solid #c4c4c4;
        }
        > svg {
          height: 19px;
          width: 19px;
          fill: #82888a;
        }
      }
    }
    &__days-legend {
      position: absolute;
      top: 50px;
      left: 10px;
      padding: 0 10px;
    }
    &__day-title {
      display: inline-block;
      width: percentage(1/7);
      text-align: center;
      margin-bottom: 4px;
      color: rgba(0, 0, 0, 0.7);
      font-size: 0.8em;
      margin-left: -1px;
    }
    &__month-table {
      border-collapse: collapse;
      border-spacing: 0;
      background: white;
      width: 100%;
      max-width: 100%;
    }
    &__month {
      transition: all $transition-time ease;
      display: inline-block;
      padding: 15px;
      &--hidden {
        height: 275px;
        visibility: hidden;
      }
    }
    &__month-name {
      font-size: 1.3em;
      text-align: center;
      margin: 0 0 30px;
      line-height: 1.4em;
      font-weight: bold;
    }
    &__month-year-select {
      &::-ms-expand {
        display: none;
      }
      -webkit-appearance: none;
      border: none;
      background-color: inherit;
      cursor: pointer;
      color: blue;
      font-size: inherit;
      font-weight: inherit;
      padding: 0;
    }
    &__day {
      $size: 38px;
      line-height: $size;
      height: $size;
      padding: 0;
      overflow: hidden;
      &--enabled {
        border: $border;
        &:hover {
          background-color: #e4e7e7;
        }
        &:focus {
          outline: auto 5px Highlight;
          outline: auto 5px -webkit-focus-ring-color;
        }
      }
      &--disabled,
      &--empty {
        opacity: 0.5;
        button {
          cursor: default;
        }
      }
      &--empty {
        border: none;
      }
      &--disabled {
        &:hover {
          background-color: transparent;
        }
      }
    }
    &__day-button {
      background: transparent;
      width: 100%;
      height: 100%;
      border: none;
      cursor: pointer;
      color: inherit;
      text-align: center;
      user-select: none;
      font-size: 15px;
      font-weight: inherit;
      padding: 0;
    }
    &__action-buttons {
      min-height: 50px;
      padding-top: 10px;
      margin-bottom: 12px;
      button {
        display: block;
        position: relative;
        background: transparent;
        border: none;
        font-weight: bold;
        font-size: 15px;
        cursor: pointer;
        &:hover {
          text-decoration: underline;
        }
        &:nth-child(1) {
          float: left;
          left: 15px;
        }
        &:nth-child(2) {
          float: right;
          right: 15px;
        }
      }
    }
    &__mobile-header {
      border-bottom: $border-normal;
      position: relative;
      padding: 15px 15px 15px 15px !important;
      text-align: center;
      height: 50px;
      h3 {
        font-size: 20px;
        margin: 0;
      }
    }
    &__mobile-only {
      display: none;
      @media (max-width: 600px) {
        display: block;
      }
    }
    &__mobile-close {
      border: none;
      position: absolute;
      top: 7px;
      right: 5px;
      padding: 5px;
      z-index: 100;
      cursor: pointer;
      &__icon {
        position: relative;
        font-size: 1.6em;
        font-weight: bold;
        padding: 0;
      }
    }
  }
</style>
