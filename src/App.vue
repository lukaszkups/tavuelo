<template>
  <div id="app">
    <tavuelo
      :columns='tableColumns'
      :data='tableData'
      :per-page='20'
      :has-search='true'
      :searchColumns='["first_name", "last_name", "location", "age"]'
      :searchCaseSensitive='true'
      :useFlex='true'
      :wrapContent='true'
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
        { title: 'Age', dataSource: 'age', width: '60px' },
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
