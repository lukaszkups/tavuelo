<template>
  <div id="app">
    <tavuelo
      :columns='tableColumns'
      :data='tableData'
      :perPage='20'
      :hasSearch='true'
      :searchColumns='["first_name", "last_name", "location", "age"]'
      :searchCaseSensitive='true'
      :wrapContent='true'
      defaultSortDataName='first_name'
      :clickHeaderToSort='true'
      :rowClick='rowClickCallback'
    ></tavuelo>
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
      ],
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
      console.log('ROW CLICK', row);
    },
    cellClickCallback(e, row) {
      e.stopPropagation();
      console.log('CELL CLICK', row);
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
