<template>
  <div id="app">
    <tavuelo
      :columns='tableColumns'
      :data='tableData'
      :per-page='11'
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
        { title: 'First name', dataSource: 'first_name' },
        { title: 'Last name', dataSource: 'last_name' },
        { title: 'Age', dataSource: 'age' },
        { title: 'Location', dataSource: 'location' },
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
