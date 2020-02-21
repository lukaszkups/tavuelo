<img src="/public/tavuelo-logo.svg" alt="tavuelo logo" width="150px" />

# tavuelo
Easy to use but with advanced features a Vue.js table component

# Installation

First, install npm package:

```
npm i --save tavuelo
```

# Demo

Clone this repository to your local hard drive.

Install all dependencies:

```
npm i
```

Run the demo:

```
npm run serve
```

Enter `http://localhost:8080` in your browser and check the example of Tavuelo functionalities. Demo code can be found at `/src/App.vue` file.

# Usage

Global usage:

```
// main.js file
import tavuelo from 'tavuelo'

Vue.use(tavuelo)
```

Local usage:

```
// in your Vue.js component
import Tavuelo from 'tavuelo'

export default {
  name: 'YourComponent',
  components: {
    Tavuelo
  }
}
```

## Template usage example:

Minimal example:

```
<template>
  <tavuelo
    :columns='tableColumns'
    :data='tableData'
  ></tavuelo>
</template>
```

Extended example:

```
<template>
<tavuelo
  :columns='[
    { title: "First name", dataSource: "first_name", minWidth: "250px" },
    { title: "Last name", dataSource: "last_name", width: "150px" },
    { title: "Age", dataSource: "age", width: "60px" },
    { title: "Location", dataSource: "location" },
    { title: "Active", dataSource: "active", type: "bool", tooltip: "Information if user is active", width: "80px" },
    { title: 'Full name', dataSource: 'fullName', type: 'slot', slotName: 'fullName', computedValue: row => `${row.first_name} ${row.last_name}` },
    { title: 'Login', dataSource: 'login', type: 'slot', slotName: 'login', computedValue: row => `${row.first_name}${row.age}${row.last_name}` }
  ]"
  :data='[
    {"first_name":"Anthony","last_name":"Levine","age":52,"location":"Lamorteau","active":0,"joined":"1993-11-25T16:42:12-08:00","email":"Sed.nec@eleifendvitae.edu","phone":346593579},
    {"first_name":"Warren","last_name":"Mcintyre","age":61,"location":"Zwevegem","active":1,"joined":"2013-02-07T05:05:17-08:00","email":"ante.lectus.convallis@pharetrased.ca","phone":349017516}
  ]'
  :customSortRules='{
    fullName: (dataCopy, direction) => {
      dataCopy.sort((a, b) => String(`${a.first_name} ${a.last_name}`).localeCompare(String(`${b.first_name} ${b.last_name}`)));
      if (direction === 'desc') {
        dataCopy.reverse();
      }
      return dataCopy;
    },
  }'
  :perPage='20'
  :hasSearch='true'
  :searchColumns='["first_name", "last_name", "location", "age"]'
  :searchCaseSensitive='true'
  :useFlex='true'
  :wrapContent='true'
  defaultSortDataName='first_name'
  :clickHeaderToSort='true'
  :downloadDataButton='true'
  :useNoDataSlot='true'
>
  <template slot='noDataSlot'>There is no data here!</template>
  <template slot='fullName' slot-scope='{entry}'>
    {{ entry.first_name }} {{ entry.last_name }}
  </template>
  <template slot='login' slot-scope='{entry}'>
    {{entry.first_name}}{{entry.age}}{{entry.last_name}}
  </template>
</tavuelo>
</template>
```

# API

## Table component

|Component's prop|Default value|Value examples|Description|
| --- | --- | --- | --- |
|columns|`[]`|`[{title: 'First name', dataSource: 'first_name'}, {title: 'Last name', dataSource: 'last_name'}]`| An array of objects that represents table columns|
|data|`[]`|`[{id: 1, first_name: 'Lukasz', last_name: 'Kups'}, {id: 2, first_name: 'Joe', last_name: 'Doe'}]`| An array of objects that represents table contents data|
|perPage|`0`|`20`|A number that defines how many items should be displayed on each pagination when pagination is enabled (`0` disables pagination|
|hasSearch|`false`|`true`|A boolean value which sets if table should have search functionality (it will display a search input above the table|
|customFiltering|`n/a`|`(data, searchQuery) => {...}`|A function that is executed instead of default filtering while search. It takes two parameters: copy of table's data (array) and string that is the search input value|
|searchColumns|`[]`|`['first_name, 'last_name', 'location']|`|An array that contains string names of column's data sources|
|searchCaseSensitive|`false`|`true`|A boolean value to set if search string should be case sensitive or not|
|useFlex|`true`|`false`|A boolean value to set if table columns should behave like flex elements (e.g. columns won't resize on page change)|
|wrapContent|`false`|`true`|A boolean value that adds ellipsis effect to each table cell if content overflows max column width|
|defaultSortDataName|`''`|`'first_name'`|A string value that points on column's data source that should be default sorting based on|
|defaultSortDirection|`'asc'`|`'desc'`|A string `asc` or `desc` that set the default sort direction|
|customSortRules|`n/a`|`{first_name: (dataCopy, sortDirection) => {...}}`|An object that keys corresponds to column data sources and values are functions that handle custom sort logic while taking 2 parameters: `dataCopy` which is exact copy of table's data and `sortDirection` (which takes `asc` or `desc` as its value)|
|clickHeaderToSort|`false`|`true`|A boolean value that binds column sorting with whole table header cell (by default, tavuelo handle sorting by clicking sort arrows only)|
|rowClick|`null`|`rowclickCallback(row) { ... }`|A function that is executed when user clicks on any table body row. This function takes `row` parameter by default so you can learn on which row user clicked|
|downloadDataButton|`false`|`true`|A boolean value that toggle visibility of button that triggers data download action|
|downloadDataFileType|`json`|`csv`|A string value that determines if data is available to download as json or csv file|
|noDataLabel|`No data`|`There is no results`|A string message that shows when there is no results/data in the table|
|useNoDataSlot|`false`|`true`|A boolean value that toggle usage of slot block that will be visible when there won't be any results/data in the table. If it is set to `true` then tavuelo will use the template block with `noDataSlot` slot name instead of displaying text that has been set in `noDataLabel` property|
|showAllPages|`false`|`true`|A boolean value that toggle how pagination is displayed. By default it shows [First page][Prev page]Page [input which changes page that equals entered value on blur] of TOTAL_PAGES [Next page][Last page]. If set to `true`, it displays all clickable page numbers as separate buttons, e.g. [First page][Prev page][1][2][3][4][5][Next page][Last page]|
|selectableRows|`false`|`true`|A boolean value that enable or disable row selection feature. Once enabled, an extra column will be added at the beginning of each row with checkbox input|
|selectedRows.sync|`[]`|`[1, 2, 5]`|Array that contains IDs of entries which are selected. On change, tavuelo emits updated array so thanks to `.sync` modifier you can use this value in your application. Works only if `selectableRows` prop is `true`|
|selectAllRowsButton|`false`|`true`|A boolean value that toggle visibility of `Select all` button in the selection column header. Once clicked, it will select all available records (or if all are already selected, it will deselect it) and update `selectedRows` prop. Button label can be set via CSS :before `content` property.|
|selectRowsOnPageButton|`false`|`true`|A boolean value that toggle visibility of `Select all on page` button in the selection column header. Once clicked, it will select all records that are displayed on current page (or if all are already selected, it will deselect it) and update `selectedRows` prop. Button label can be set via CSS :before `content` property.|

## Table column definition

To create table properly you have to pass an array of columns definitions to `:columns` component property, e.g.:

```
[
  {title: 'First name', dataSource: 'first_name'},
  {title: 'Last name', dataSource: 'last_name'},
  {title: 'Active', dataSource: 'is_active', type: 'bool'}, // type 'text' is default
  { title: 'Full name', dataSource: 'fullName', type: 'slot', slotName: 'fullName', computedValue: row => `${row.first_name} ${row.last_name}` }
]
```

|Property name|Available values|Example value|Description|
| --- | --- | --- | --- |
|title|Any string|`First name`|Title of the column that will be displayed in table header|
|tooltip|Any string|`First name of the user`|If defined, when user hover on table header cell, it will display additional content as tooltip bubble. Handy when table gets really wide and want to show shorter table headings with extensive descriptions inside tooltips.|
|dataSource|Any string|`first_name`|Name of the property that should be displayed in the column.|
|type|`text`(default), `bool`, `slot`|`bool`|Defines type of the column contents. By default, `text` column data type is set. When `bool`, it displays tick (true) or cross(false) icon instead of plain value. When `slot`, the `slotName` prop is required that will tell tavuelo what slot block should be used to display the cell contents|
|onClick|Any function|`cellClickCallback(row, column, event) { ... }`|A function that is executed when user clicks on given table column cell. This function takes `row`, `column` and `event` parameters by default so you can use them in your callback.|
|slotName|Any string|`fullName`|Name of the slot block that will be used to display contents of the cell. Slot block also needs to have defined `slot-scope='{entry}'` property. The `entry` value contains object of current table row.|
|computedValue|Any function|`getFullName(row)`|A function that returns computed value of the cell - handy when using slot-based columns and wants to handle filtering/sorting on this column (as the result of this function will be used for filter/sorting).

# Roadmap

Roadmap to version `1.0.0.` is available at [issues tab](https://github.com/lukaszkups/tavuelo/issues).

Feel free to add own `feature request` issues there.

# Changelog

You can find the release notes/changelog at [lukaszkups website](https://lukaszkups.net/notes/?category=tavuelo)

# Troubleshoot & support

You can submit bugs similarly to feature request at github's [issues tab](https://github.com/lukaszkups/tavuelo/issues) with `bug` label.

For more alike real-time support, feel free to [tweet to me](https://twitter.com/lukaszkups).

# License

[MIT](https://github.com/lukaszkups/tavuelo/blob/master/LICENSE)

# Author

[@lukaszkups](https://lukaszkups.net)
