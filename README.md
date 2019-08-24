# tavuelo
A simple Vue.js table component

# Installation

First, install npm package:

```
npm i --save tavuelo
```

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

Extensed example:

```
<template>
<tavuelo
  :columns='[
    { title: "First name", dataSource: "first_name", minWidth: "250px" },
    { title: "Last name", dataSource: "last_name", width: "150px" },
    { title: "Age", dataSource: "age", width: "60px" },
    { title: "Location", dataSource: "location" },
    { title: "Active", dataSource: "active", type: "bool", tooltip: "Information if user is active", width: "80px" },
  ]"
  :data='[
    {"first_name":"Anthony","last_name":"Levine","age":52,"location":"Lamorteau","active":0,"joined":"1993-11-25T16:42:12-08:00","email":"Sed.nec@eleifendvitae.edu","phone":346593579},
    {"first_name":"Warren","last_name":"Mcintyre","age":61,"location":"Zwevegem","active":1,"joined":"2013-02-07T05:05:17-08:00","email":"ante.lectus.convallis@pharetrased.ca","phone":349017516}
  ]'
  :perPage='20'
  :hasSearch='true'
  :searchColumns='["first_name", "last_name", "location", "age"]'
  :searchCaseSensitive='true'
  :useFlex='true'
  :wrapContent='true'
  defaultSortDataName='first_name'
  :clickHeaderToSort='true'
></tavuelo>
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

# Roadmap

Roadmap to version `1.0.0.` is available at [issues tab](https://github.com/lukaszkups/tavuelo/issues).

Feel free to add own `enhacement` issues there.

# Troubleshoot & support

You can submit bugs similarly to feature request at github's [issues tab](https://github.com/lukaszkups/tavuelo/issues) with `bug` label.

For more alike real-time support, feel free to [tweet to me](https://twitter.com/lukaszkups).

# License

[MIT](https://github.com/lukaszkups/tavuelo/blob/master/LICENSE)

# Author

[@lukaszkups](https://lukaszkups.net)
