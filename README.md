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

# License

MIT.

# Author

![@lukaszkups](https://lukaszkups.net)
