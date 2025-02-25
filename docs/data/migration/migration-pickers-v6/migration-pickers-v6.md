---
productId: x-date-pickers
---

# Migration from v6 to v7

<!-- #default-branch-switch -->

<p class="description">This guide describes the changes needed to migrate the Date and Time Pickers from v6 to v7.</p>

## Introduction

TBD

## Start using the new release

In `package.json`, change the version of the date pickers package to `next`.

```diff
-"@mui/x-date-pickers": "6.x.x",
+"@mui/x-date-pickers": "next",
```

## Breaking changes

Since `v7` is a major release, it contains changes that affect the public API.
These changes were done for consistency, improved stability and to make room for new features.

### Rename `components` to `slots`

The `components` and `componentsProps` props are renamed to `slots` and `slotProps` props respectively.
This is a slow and ongoing effort between the different MUI libraries.
To smooth the transition, they were deprecated during the [v6](/x/migration/migration-pickers-v5/#rename-components-to-slots-optional).
And are removed from the v7.

If not already done, this modification can be handled by the codemod

```bash
npx @mui/x-codemod v7.0.0/pickers/ <path>
```

Take a look at [the RFC](https://github.com/mui/material-ui/issues/33416) for more information.

:::warning
If this codemod is applied on a component with both a `slots` and a `components` prop, the output will contain two `slots` props.
You are then responsible for merging those two props manually.

For example:

```tsx
// Before running the codemod
<DatePicker
  slots={{ textField: MyTextField }}
  components={{ toolbar: MyToolbar }}
/>

// After running the codemod
<DatePicker
  slots={{ textField: MyTextField }}
  slots={{ toolbar: MyToolbar }}
/>
```

The same applies to `slotProps` and `componentsProps`.
:::

### Change the imports of the `calendarHeader` slot

The imports related to the `calendarHeader` slot have been moved from `@mui/x-date-pickers/DateCalendar` to `@mui/x-date-pickers/PIckersCalendarHeader`:

```diff
  export {
    pickersCalendarHeaderClasses,
    PickersCalendarHeaderClassKey,
    PickersCalendarHeaderClasses,
    PickersCalendarHeader,
    PickersCalendarHeaderProps,
    PickersCalendarHeaderSlotsComponent,
    PickersCalendarHeaderSlotsComponentsProps,
    ExportedPickersCalendarHeaderProps,
- } from '@mui/x-date-pickers/DateCalendar';
+ } from '@mui/x-date-pickers/PickersCalendarHeader';
```

## Field components

### Replace the section `hasLeadingZeros` property

:::success
This only impacts you if you are using the `unstableFieldRef` prop to imperatively access the section object.
:::

The property `hasLeadingZeros` has been removed from the sections in favor of the more precise `hasLeadingZerosInFormat` and `hasLeadingZerosInInput` properties.
To keep the same behavior, you can replace it by `hasLeadingZerosInFormat`

```diff
 const fieldRef = React.useRef<FieldRef<FieldSection>>(null);

 React.useEffect(() => {
     const firstSection = fieldRef.current!.getSections()[0]
-    console.log(firstSection.hasLeadingZeros)
+    console.log(firstSection.hasLeadingZerosInFormat)
 }, [])

 return (
   <DateField unstableFieldRef={fieldRef} />
 );
```

## Removed formats

### Remove the `monthAndYear` format

The `monthAndYear` format has been removed.
It was used in the header of the calendar views, you can replace it with the new `format` prop of the `calendarHeader` slot:

```diff
  <LocalizationProvider
    adapter={AdapterDayJS}
-   formats={{ monthAndYear: 'MM/YYYY' }}
  />
    <DatePicker
+     slotProps={{ calendarHeader: { format: 'MM/YYYY' }}}
    />
     <DateRangePicker
+     slotProps={{ calendarHeader: { format: 'MM/YYYY' }}}
    />
  <LocalizationProvider />
```

## Use UTC with the Day.js adapter

The `dateLibInstance` prop of `LocalizationProvider` does not work with `AdapterDayjs` anymore.
This prop was used to set the pickers in UTC mode before the implementation of a proper timezone support in the components.
You can learn more about the new approach on the [dedicated doc page](https://mui.com/x/react-date-pickers/timezone/).

```diff
  // When a `value` or a `defaultValue` is provided
  <LocalizationProvider
    adapter={AdapterDayjs}
-   dateLibInstance={dayjs.utc}
  >
    <DatePicker value={dayjs.utc('2022-04-17')} />
  </LocalizationProvider>

  // When no `value` or `defaultValue` is provided
  <LocalizationProvider
    adapter={AdapterDayjs}
-   dateLibInstance={dayjs.utc}
  >
-   <DatePicker />
+   <DatePicker timezone="UTC" />
  </LocalizationProvider>
```

## Adapters

:::success
The following breaking changes only impact you if you are using the adapters outside the pickers like displayed in the following example:

```tsx
import { AdapterDayjs } from '@mui/x-date-pickers/AdapterDayjs';

const adapter = new AdapterDays();
adapter.isValid(dayjs('2022-04-17T15:30'));
```

If you are just passing an adapter to `LocalizationProvider`, then you can safely skip this section.
:::

### Remove the `getDiff` method

The `getDiff` method have been removed, you can directly use your date library:

```diff
  // For Day.js
- const diff = adapter.getDiff(value, comparing, unit);
+ const diff = value.diff(comparing, unit);

  // For Luxon
- const diff = adapter.getDiff(value, comparing, unit);
+ const getDiff = (value: DateTime, comparing: DateTime | string, unit?: AdapterUnits) => {
+   const parsedComparing = typeof comparing === 'string'
+     ? DateTime.fromJSDate(new Date(comparing))
+     : comparing;
+   if (unit) {
+     return Math.floor(value.diff(comparing).as(unit));
+   }
+   return value.diff(comparing).as('millisecond');
+ };
+
+ const diff = getDiff(value, comparing, unit);

  // For DateFns
- const diff = adapter.getDiff(value, comparing, unit);
+ const getDiff = (value: Date, comparing: Date | string, unit?: AdapterUnits) => {
+   const parsedComparing = typeof comparing === 'string' ? new Date(comparing) : comparing;
+   switch (unit) {
+     case 'years':
+       return dateFns.differenceInYears(value, parsedComparing);
+     case 'quarters':
+       return dateFns.differenceInQuarters(value, parsedComparing);
+     case 'months':
+       return dateFns.differenceInMonths(value, parsedComparing);
+     case 'weeks':
+       return dateFns.differenceInWeeks(value, parsedComparing);
+     case 'days':
+       return dateFns.differenceInDays(value, parsedComparing);
+     case 'hours':
+       return dateFns.differenceInHours(value, parsedComparing);
+     case 'minutes':
+       return dateFns.differenceInMinutes(value, parsedComparing);
+     case 'seconds':
+       return dateFns.differenceInSeconds(value, parsedComparing);
+     default: {
+       return dateFns.differenceInMilliseconds(value, parsedComparing);
+     }
+   }
+ };
+
+ const diff = getDiff(value, comparing, unit);

  // For Moment
- const diff = adapter.getDiff(value, comparing, unit);
+ const diff = value.diff(comparing, unit);
```

### Remove the `getFormatHelperText` method

The `parseISO` method have been removed, you can use the `expandFormat` instead:

```diff
- const expandedFormat = adapter.getFormatHelperText(format);
+ const expandedFormat = adapter.expandFormat(format);
```

And if you need the exact same output you can apply the following transformation:

```diff
  // For Day.js
- const expandedFormat = adapter.getFormatHelperText(format);
+ const expandedFormat = adapter.expandFormat(format).replace(/a/gi, '(a|p)m').toLocaleLowerCase();

  // For Luxon
- const expandedFormat = adapter.getFormatHelperText(format);
+ const expandedFormat = adapter.expandFormat(format).replace(/(a)/g, '(a|p)m').toLocaleLowerCase();

  // For DateFns
- const expandedFormat = adapter.getFormatHelperText(format);
+ const expandedFormat = adapter.expandFormat(format).replace(/(aaa|aa|a)/g, '(a|p)m').toLocaleLowerCase();

  // For Moment
- const expandedFormat = adapter.getFormatHelperText(format);
+ const expandedFormat = adapter.expandFormat(format).replace(/a/gi, '(a|p)m').toLocaleLowerCase();
```

### Remove the `getMeridiemText` method

The `getMeridiemText` method have been removed, you can use the `setHours`, `date` and `format` methods to recreate its behavior:

```diff
- const meridiem = adapter.getMeridiemText('am');
+ const getMeridiemText = (meridiem: 'am' | 'pm') => {
+   const date = adapter.setHours(adapter.date()!, meridiem === 'am' ? 2 : 14);
+   return utils.format(date, 'meridiem');
+ };
+
+ const meridiem = getMeridiemText('am');
```

### Remove the `getMonthArray` method

The `getMonthArray` method have been removed, you can use the `startOfYear` and `addMonths` methods to recreate its behavior:

```diff
- const monthArray = adapter.getMonthArray(value);
+ const getMonthArray = (year) => {
+   const firstMonth = utils.startOfYear(year);
+   const months = [firstMonth];
+
+   while (months.length < 12) {
+     const prevMonth = months[months.length - 1];
+     months.push(utils.addMonths(prevMonth, 1));
+   }
+
+   return months;
+ }
+
+ const monthArray = getMonthArray(value);
```

### Remove the `getNextMonth` method

The `getNextMonth` method have been removed, you can use the `addMonths` method instead:

```diff
- const nextMonth = adapter.getNextMonth(value);
+ const nextMonth = adapter.addMonths(value, 1);
```

### Remove the `getPreviousMonth` method

The `getPreviousMonth` method have been removed, you can use the `addMonths` method instead:

```diff
- const previousMonth = adapter.getPreviousMonth(value);
+ const previousMonth = adapter.addMonths(value, -1);
```

### Remove the `getWeekdays` method

The `getWeekdays` method have been removed, you can use the `startOfWeek` and `addDays` methods instead:

```diff
- const weekDays = adapter.getWeekdays(value);
+ const getWeekdays = (value) => {
+   const start = adapter.startOfWeek(value);
+   return [0, 1, 2, 3, 4, 5, 6].map((diff) => utils.addDays(start, diff));
+ };
+
+ const weekDays = getWeekdays(value);
```

### Remove the `isNull` method

The `parseISO` method have been removed, you can replace it with a very basic check:

```diff
- const isNull = adapter.isNull(value);
+ const isNull = value === null;
```

### Remove the `mergeDateAndTime` method

The `mergeDateAndTime` method have been removed, you can use the `setHours`, `setMinutes`, and `setSeconds` methods to recreate its behavior:

```diff
- const result = adapter.mergeDateAndTime(valueWithDate, valueWithTime);
+ const mergeDateAndTime = <TDate>(
+   dateParam,
+   timeParam,
+ ) => {
+   let mergedDate = dateParam;
+   mergedDate = utils.setHours(mergedDate, utils.getHours(timeParam));
+   mergedDate = utils.setMinutes(mergedDate, utils.getMinutes(timeParam));
+   mergedDate = utils.setSeconds(mergedDate, utils.getSeconds(timeParam));
+
+   return mergedDate;
+ };
+
+ const result = mergeDateAndTime(valueWithDate, valueWithTime);
```

### Remove the `parseISO` method

The `parseISO` method have been removed, you can directly use your date library:

```diff
  // For Day.js
- const value = adapter.parseISO(isoString);
+ const value = dayjs(isoString);

  // For Luxon
- const value = adapter.parseISO(isoString);
+ const value = DateTime.fromISO(isoString);

  // For DateFns
- const value = adapter.parseISO(isoString);
+ const value = dateFns.parseISO(isoString);

  // For Moment
- const value = adapter.parseISO(isoString);
+ const value = moment(isoString, true);
```

### Remove the `toISO` method

The `toISO` method have been removed, you can directly use your date library:

```diff
- const isoString = adapter.toISO(value);
+ const isoString = value.toISOString();
+ const isoString = value.toUTC().toISO({ format: 'extended' });
+ const isoString = dateFns.formatISO(value, { format: 'extended' });
+ const isoString = value.toISOString();
```

The `getYearRange` method used to accept two params and now accepts a tuple to be consistent with the `isWithinRange` method:

```diff
- adapter.getYearRange(start, end);
+ adapter.getYearRange([start, end])
```

### Restrict the input format of the `isEqual` method

The `isEqual` method used to accept any type of value for its two input and tried to parse them before checking if they were equal.
The method has been simplified and now only accepts an already-parsed date or `null` (ie: the same formats used by the `value` prop in the pickers)

```diff
 const adapterDayjs = new AdapterDayjs();
 const adapterLuxon = new AdapterLuxon();
 const adapterDateFns = new AdapterDateFns();
 const adapterMoment = new AdatperMoment();

 // Supported formats
 const isValid = adapterDayjs.isEqual(null, null); // Same for the other adapters
 const isValid = adapterLuxon.isEqual(DateTime.now(), DateTime.fromISO('2022-04-17'));
 const isValid = adapterMoment.isEqual(moment(), moment('2022-04-17'));
 const isValid = adapterDateFns.isEqual(new Date(), new Date('2022-04-17'));

 // Non-supported formats (JS Date)
- const isValid = adapterDayjs.isEqual(new Date(), new Date('2022-04-17'));
+ const isValid = adapterDayjs.isEqual(dayjs(), dayjs('2022-04-17'));

- const isValid = adapterLuxon.isEqual(new Date(), new Date('2022-04-17'));
+ const isValid = adapterLuxon.isEqual(DateTime.now(), DateTime.fromISO('2022-04-17'));

- const isValid = adapterMoment.isEqual(new Date(), new Date('2022-04-17'));
+ const isValid = adapterMoment.isEqual(moment(), moment('2022-04-17'));

 // Non-supported formats (string)
- const isValid = adapterDayjs.isEqual('2022-04-16', '2022-04-17');
+ const isValid = adapterDayjs.isEqual(dayjs('2022-04-17'), dayjs('2022-04-17'));

- const isValid = adapterLuxon.isEqual('2022-04-16', '2022-04-17');
+ const isValid = adapterLuxon.isEqual(DateTime.fromISO('2022-04-17'), DateTime.fromISO('2022-04-17'));

- const isValid = adapterMoment.isEqual('2022-04-16', '2022-04-17');
+ const isValid = adapterMoment.isEqual(moment('2022-04-17'), moment('2022-04-17'));

- const isValid = adapterDateFns.isEqual('2022-04-16', '2022-04-17');
+ const isValid = adapterDateFns.isEqual(new Date('2022-04-17'), new Date('2022-04-17'));
```

### Restrict the input format of the `isValid` method

The `isValid` method used to accept any type of value and tried to parse them before checking their validity.
The method has been simplified and now only accepts an already-parsed date or `null`.
Which is the same type as the one accepted by the components `value` prop.

```diff
 const adapterDayjs = new AdapterDayjs();
 const adapterLuxon = new AdapterLuxon();
 const adapterDateFns = new AdapterDateFns();
 const adapterMoment = new AdatperMoment();

 // Supported formats
 const isValid = adapterDayjs.isValid(null); // Same for the other adapters
 const isValid = adapterLuxon.isValid(DateTime.now());
 const isValid = adapterMoment.isValid(moment());
 const isValid = adapterDateFns.isValid(new Date());

 // Non-supported formats (JS Date)
- const isValid = adapterDayjs.isValid(new Date('2022-04-17'));
+ const isValid = adapterDayjs.isValid(dayjs('2022-04-17'));

- const isValid = adapterLuxon.isValid(new Date('2022-04-17'));
+ const isValid = adapterLuxon.isValid(DateTime.fromISO('2022-04-17'));

- const isValid = adapterMoment.isValid(new Date('2022-04-17'));
+ const isValid = adapterMoment.isValid(moment('2022-04-17'));

 // Non-supported formats (string)
- const isValid = adapterDayjs.isValid('2022-04-17');
+ const isValid = adapterDayjs.isValid(dayjs('2022-04-17'));

- const isValid = adapterLuxon.isValid('2022-04-17');
+ const isValid = adapterLuxon.isValid(DateTime.fromISO('2022-04-17'));

- const isValid = adapterMoment.isValid('2022-04-17');
+ const isValid = adapterMoment.isValid(moment('2022-04-17'));

- const isValid = adapterDateFns.isValid('2022-04-17');
+ const isValid = adapterDateFns.isValid(new Date('2022-04-17'));
```
