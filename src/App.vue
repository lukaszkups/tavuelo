<template>
  <div id="app">
    <tavuelo
      :columns='tableColumns'
      :data='tableData'
      :perPage='10'
      :hasSearch='true'
      :searchColumns='["first_name", "last_name", "location", "age", "fullName", "login"]'
      :searchCaseSensitive='true'
      :wrapContent='true'
      defaultSortDataName='first_name'
      :clickHeaderToSort='true'
      :rowClick='rowClickCallback'
      :downloadDataButton='true'
      downloadDataFileType='csv'
      :customSortRules='customSortRules'
      :useNoDataSlot='true'
    >
      <template slot='noDataSlot'>There is no data here!</template>
      <template slot='fullName' slot-scope='{entry}'>
        {{ entry.first_name }} {{ entry.last_name }}
      </template>
      <template slot='email' slot-scope='{entry}'>
        <a :href='`mailto:${entry.email}`'>{{ entry.email }}</a>
      </template>
      <template slot='login' slot-scope='{entry}'>
        {{entry.first_name}}{{entry.age}}{{entry.last_name}}
      </template>
    </tavuelo>
  </div>
</template>

<script>
import Tavuelo from './components/Tavuelo.vue';
import { exampleTableData } from './helpers/dataHelper';

export default {
  name: 'app',
  data() {
    return {
      tableColumns: [
        { title: 'First name', dataSource: 'first_name', minWidth: '250px' },
        { title: 'Last name', dataSource: 'last_name', width: '150px' },
        { title: 'Age', dataSource: 'age', width: '60px', onClick: this.cellClickCallback },
        { title: 'Location', dataSource: 'location' },
        { title: 'Active', dataSource: 'active', type: 'bool', tooltip: 'Information if user is active', width: '80px' },
        { title: 'Full name', dataSource: 'fullName', type: 'slot', slotName: 'fullName', computedValue: row => `${row.first_name} ${row.last_name}` },
        { title: 'E-mail', dataSource: 'email', type: 'slot', slotName: 'email' },
        { title: 'Login', dataSource: 'login', type: 'slot', slotName: 'login', computedValue: row => `${row.first_name}${row.age}${row.last_name}` },
      ],
      customSortRules: {
        fullName: (dataCopy, direction) => {
          dataCopy.sort((a, b) => String(`${a.first_name} ${a.last_name}`).localeCompare(String(`${b.first_name} ${b.last_name}`)));
          // .sort and then .reverse is currently most efficient way if sort direction is set to `desc`
          if (direction === 'desc') {
            dataCopy.reverse();
          }
          return dataCopy;
        },
      },
    };
  },
  computed: {
    tableData() {
      return exampleTableData();
    },
  },
  components: {
    Tavuelo,
  },
  methods: {
    rowClickCallback(row) {
      /* eslint-ignore-next-line no-console */
      console.log('ROW CLICK', row);
    },
    cellClickCallback(e, row, column) {
      /* eslint-ignore-next-line no-console */
      console.log('CELL CLICK', e, row, column);
    },
  },
};
</script>

<style lang="scss">
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
