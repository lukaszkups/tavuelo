<template>
  <div class='tavuelo__wrapper'>
    <div class='tavuelo__wrapper--top'>
      <div v-if='downloadDataButton' class='tavuelo__download-data-wrapper'>
        <button
          class='tavuelo__button--download-data'
          @click='downloadDataButtonClick'
        >Download</button>
      </div>
      <div v-if='hasSearch' class='tavuelo__search'>
        <input
          v-model='searchQuery'
          placeholder='Search'
        />
      </div>
    </div>
    <table
      :class='["tavuelo", {
        "tavuelo--flex-table": this.useFlex,
        "tavuelo--wrap-content": this.wrapContent,
      }]'
    >
      <thead>
        <tr>
          <th v-if='selectableRows === true'>
            <span
              v-if='selectAllRowsButton === true'
              class='tavuelo--select-all'
              @click='toggleSelectAll'
            ></span>
            <span
              v-if='selectRowsOnPageButton === true'
              class='tavuelo--select-page'
              @click='toggleSelectPage'
            ></span>
          </th>
          <th
            v-for='column in computedColumns'
            :key='column[entryIdentifier]'
            :style='getComputedColumnStyle(column)'
            :class='{"column--has-sorting": clickHeaderToSort === true && searchColumns.includes(column.dataSource)}'
            @click='clickHeaderToSort === true && searchColumns.includes(column.dataSource) ? toggleSorting(column) : false'
          >
            <div class='cell'>
              <div v-if='column.tooltip'>
                <div class='tavuelo-tooltip tavuelo-tooltips'>
                  {{ column.title }}
                  <span>{{ column.tooltip }}</span>
                </div>
              </div>
              <div v-else>{{ column.title }}</div>
              <div
                v-if='searchColumns.includes(column.dataSource)'
                :class='["tavuelo-sorting", {
                  "tavuelo-sorting__active": column.dataSource === currentSortDataName,
                  "sorting-asc": column.dataSource === currentSortDataName && currentSortDirection === "asc",
                  "sorting-desc": column.dataSource === currentSortDataName && currentSortDirection === "desc",
                }]'
                @click='clickHeaderToSort === false && searchColumns.includes(column.dataSource) ? toggleSorting(column) : false'
              ></div>
            </div>
          </th>
        </tr>
      </thead>
      <tbody>
        <tr
          v-for='row in computedData'
          :key='row[entryIdentifier]'
          @click='rowClick && typeof rowClick === "function" ? rowClick(row) : false'
        >
          <td v-if='selectableRows === true'>
            <input
              @click.stop='() => updateRowSelection(row[entryIdentifier])'
              type='checkbox'
              :value='row[entryIdentifier]'
              v-model='selectedRows'
            />
          </td>
          <td
            v-for='column in computedColumns'
            :key='`${row[entryIdentifier]}-${column.tavuelo_id}`'
            :style='getComputedColumnStyle(column)'
            @click='onTableCellClick($event, column, row)'
          >
            <div
              v-if='!column.type || column.type === "text"'
              class='cell'
            >{{ row[column.dataSource] }}</div>
            <div
              v-else-if='column.type === "bool"'
              class='cell'
            >
              <span v-if='row[column.dataSource]' class='tavuelo-check-icon'>&check;</span>
              <span v-else class='tavuelo-cross-icon'>&cross;</span>
            </div>
            <div
              v-else-if='column.type === "slot"'
              class='cell'
            >
              <slot
                :name='column.slotName'
                :entry='row'
              ></slot>
            </div>
          </td>
        </tr>
        <tr
          v-if="!computedData || !computedData.length"
          class='tavuelo-no-data-row'
        >
          <td v-if='useNoDataSlot'>
            <slot
              name='noDataSlot'
            ></slot>
          </td>
          <td v-else>{{ noDataLabel }}</td>
        </tr>
      </tbody>
    </table>
    <tavuelo-pagination
      v-if='hasPagination'
      :activePage.sync='activePage'
      :pageList='pageList'
      :showAllPages='showAllPages'
    ></tavuelo-pagination>
  </div>
</template>
<script>
import TavueloPagination from '@/components/TavueloPagination.vue';

export default {
  name: 'Tavuelo',
  data() {
    return {
      activePage: 0,
      indexedData: [],
      searchQuery: '',
      currentSortDataName: '',
      currentSortDirection: '',
    };
  },
  components: {
    TavueloPagination,
  },
  props: {
    entryIdentifier: {
      type: String,
      default: 'id',
    },
    data: {
      type: Array,
      default: () => [],
    },
    columns: {
      type: Array,
      default: () => [],
    },
    perPage: {
      type: Number,
      default: 0,
    },
    hasSearch: {
      type: Boolean,
      default: false,
    },
    customFiltering: {
      type: Function,
    },
    searchCaseSensitive: {
      type: Boolean,
      default: false,
    },
    searchColumns: {
      type: Array,
      default: () => [],
    },
    useFlex: {
      type: Boolean,
      default: true,
    },
    wrapContent: {
      type: Boolean,
      default: false,
    },
    defaultSortDataName: {
      type: String,
    },
    defaultSortDirection: {
      type: String,
      default: 'asc',
    },
    customSortRules: {
      type: Object,
    },
    clickHeaderToSort: {
      type: Boolean,
      default: false,
    },
    rowClick: {
      type: Function,
    },
    downloadDataButton: {
      type: Boolean,
      default: false,
    },
    downloadDataFileType: {
      type: String,
      default: 'json', // available values: 'json', 'csv'
    },
    noDataLabel: {
      type: String,
      default: 'No data',
    },
    useNoDataSlot: {
      type: Boolean,
      default: false,
    },
    showAllPages: {
      type: Boolean,
      default: false,
    },
    selectableRows: {
      type: Boolean,
      default: false,
    },
    selectAllRowsButton: {
      type: Boolean,
      default: false,
    },
    selectRowsOnPageButton: {
      type: Boolean,
      default: false,
    },
    selectedRows: {
      type: Array,
      default: () => [],
    },
  },
  computed: {
    computedColumns() {
      if (this.columns && this.columns.length) {
        return this.columns.map((col, index) => {
          return { ...col, tavuelo_id: index };
        });
      }
      return [];
    },
    computedDataColumns() {
      const columns = [];
      this.computedColumns.map(col => {
        // check if there are any columns that contains computed value prop
        if (Object.prototype.hasOwnProperty.call(col, 'computedValue')) {
          columns.push({
            name: col.dataSource,
            computedValue: col.computedValue,
          });
        }
      });
      return columns;
    },
    // helper Boolean value
    hasComputedDataColumns() {
      return this.computedDataColumns && this.computedDataColumns.length;
    },
    filteredData() {
      if (this.customFiltering && typeof this.customFiltering === 'function') {
        return this.customFiltering([...this.indexedData], this.searchQuery);
      }
      if (this.indexedData && this.indexedData.length) {
        let indexedDataCopy = [...this.indexedData];
        // if case sensitive, then search through data as it is
        if (this.searchCaseSensitive) {
          indexedDataCopy = indexedDataCopy.filter(entry => {
            return this.searchColumns.find(searchColumn => String(entry[searchColumn]).includes(this.searchQuery));
          });
          return indexedDataCopy;
        }
        // if case insensitive, then lowercase everything
        const lowercaseQuery = this.searchQuery.toLowerCase();
        indexedDataCopy = indexedDataCopy.filter(entry => {
          return this.searchColumns.find(searchColumn => String(entry[searchColumn]).toLowerCase().includes(lowercaseQuery));
        });
        return indexedDataCopy;
      }
      return [];
    },
    computedData() {
      if (this.indexedData && this.indexedData.length) {
        let indexedDataCopy = [];
        if (this.hasSearch && this.searchColumns && this.searchColumns.length && this.searchQuery && this.searchQuery.length) {
          indexedDataCopy = [...this.filteredData];
        } else {
          indexedDataCopy = [...this.indexedData];
        }
        if (this.hasPagination) {
          const startIndex = this.activePage * this.perPage;
          return indexedDataCopy.slice(startIndex, startIndex + this.perPage);
        }
        return indexedDataCopy;
      }
      return [];
    },
    hasPagination() {
      return !Number.isNaN(this.perPage) && this.perPage > 0;
    },
    pageList() {
      if (this.hasPagination) {
        let indexedDataCopy = [];
        if (this.hasSearch && this.searchColumns && this.searchColumns.length && this.searchQuery && this.searchQuery.length) {
          indexedDataCopy = [...this.filteredData];
        } else {
          indexedDataCopy = [...this.indexedData];
        }
        const indexedDataCopyLength = indexedDataCopy.length;
        if (this.perPage >= indexedDataCopyLength && indexedDataCopyLength > 0) {
          return [0];
        }
        if (indexedDataCopyLength === 0) {
          return [];
        }
        const amountOfPages = Math.ceil(indexedDataCopyLength / this.perPage);
        return [...Array(amountOfPages).keys()];
      }
      return [];
    },
    localSelectedRows: {
      get() {
        return this.selectedRows;
      },
      set(newVal) {
        this.$emit('update:selectedRows', newVal);
      },
    },
  },
  methods: {
    getComputedColumnStyle(column) {
      const styles = {};
      if (column.width) {
        styles.width = column.width;
      }
      if (column.minWidth) {
        styles['min-width'] = column.minWidth;
      }
      return styles;
    },
    sortData(dataSourceName = this.currentSortDataName, direction = this.currentSortDirection) {
      let dataCopy = [...this.indexedData];
      if (this.customSortRules && Object.prototype.hasOwnProperty.call(this.customSortRules, dataSourceName)) {
        dataCopy = this.customSortRules[dataSourceName](dataCopy, direction);
      } else {
        dataCopy.sort((a, b) => String(a[dataSourceName]).localeCompare(String(b[dataSourceName])));
        // using .sort and then .reverse is currently most efficient way if sort direction is set to `desc`
        if (direction === 'desc') {
          dataCopy.reverse();
        }
      }
      this.indexedData = dataCopy;
    },
    toggleSorting(column) {
      if (column.dataSource === this.currentSortDataName) {
        this.currentSortDirection = this.currentSortDirection === 'asc' ? 'desc' : 'asc';
        this.sortData(this.currentSortDataName, this.currentSortDirection);
      } else {
        this.currentSortDataName = column.dataSource;
        this.currentSortDirection = 'asc';
        this.sortData();
      }
    },
    onTableCellClick(event, column, row) {
      if (column && column.onClick && typeof column.onClick === 'function') {
        event.stopPropagation();
        column.onClick(row, column, event);
      }
      return false;
    },
    convertDataToCsv() {
      // first, add column names
      let txt = '';
      const colAmount = this.columns.length;
      this.columns.map((column, index) => {
        txt += `${column.dataSource}${index < colAmount ? ',' : '\r\n'}`;
      });
      // then, add data
      this.computedData.map(row => {
        this.columns.map((column, index) => {
          txt += `${row[column.dataSource]}${index < colAmount ? ',' : '\r\n'}`;
        });
      });
      return txt;
    },
    // source: https://stackoverflow.com/a/33542499/1004946
    downloadDataButtonClick() {
      const data = this.downloadDataFileType === 'json' ? JSON.stringify(this.computedData) : this.convertDataToCsv();
      const blob = new Blob([data], { type: this.downloadDataFileType === 'json' ? 'application/json' : 'text/csv' });
      if (window.navigator.msSaveOrOpenBlob) {
        window.navigator.msSaveBlob(blob, `data.${this.downloadDataFileType}`);
      } else {
        const elem = window.document.createElement('a');
        elem.href = window.URL.createObjectURL(blob);
        elem.download = `data.${this.downloadDataFileType}`;
        document.body.appendChild(elem);
        elem.click();
        document.body.removeChild(elem);
      }
    },
    updateRowSelection(rowId) {
      const selectedRowsCopy = [...this.localSelectedRows];
      const activePageCopy = Number(this.activePage);
      const index = selectedRowsCopy.indexOf(rowId);
      if (index > -1) {
        selectedRowsCopy.splice(index, 1);
      } else {
        selectedRowsCopy.push(rowId);
      }
      this.localSelectedRows = selectedRowsCopy;
      this.$nextTick(() => {
        this.activePage = activePageCopy;
      });
    },
    toggleSelectAll() {
      const activePageCopy = Number(this.activePage);
      if (this.localSelectedRows.length === this.indexedData.length) {
        this.localSelectedRows = [];
      } else {
        const data = [];
        this.indexedData.map(entry => {
          data.push(entry[this.entryIdentifier]);
        });
        this.localSelectedRows = data;
      }
      this.$nextTick(() => {
        this.activePage = activePageCopy;
      });
    },
    toggleSelectPage() {
      // check if all entries of current page are selected
      let pageSelected = true;
      const missingEntries = [];
      const selectedCopy = [...this.localSelectedRows];
      const activePageCopy = Number(this.activePage);
      this.computedData.map(row => {
        if (!this.localSelectedRows.includes(row[this.entryIdentifier])) {
          pageSelected = false;
          // save IDs of entries that are not yet selected on current page
          missingEntries.push(row[this.entryIdentifier]);
        }
      });
      // if all entries from page are selected, then deselect them
      if (pageSelected) {
        this.computedData.map(row => {
          const index = selectedCopy.findIndex(id => id === row[this.entryIdentifier]);
          selectedCopy.splice(index, 1);
        });
        this.localSelectedRows = selectedCopy;
      } else {
        this.localSelectedRows = selectedCopy.concat(missingEntries);
      }
      this.$nextTick(() => {
        this.activePage = activePageCopy;
      });
    },
  },
  watch: {
    filteredData() {
      this.activePage = 0;
    },
    data(newVal) {
      this.indexedData = [...newVal];
    },
  },
  created() {
    this.indexedData = [...this.data];
    this.currentSortDataName = this.defaultSortDataName;
    this.currentSortDirection = this.defaultSortDirection;
    this.sortData();
  },
};
</script>
<style lang='sass' scoped>
.tavuelo__wrapper
  display: block

  .tavuelo
    border-collapse: collapse
    width: 100%

    thead
      tr
        th
          background: silver
          border: 1px solid grey

          &.column--has-sorting
            &:hover
              cursor: pointer

          .cell
            position: relative
            width: 100%
    tbody
      tr
        td
          border: 1px solid grey
        &:hover
          td
            background: lavender

    &.tavuelo--wrap-content
      tr
        height: 1.5em

        td, th
          overflow-x: hidden
          position: relative

          .cell
            position: absolute
            white-space: nowrap
            text-overflow: ellipsis
            height: 1.5em
            right: 0
            left: 0
            overflow-x: hidden

    &.tavuelo--flex-table
      tr
        display: flex

        th, td
          flex: 1

          &:not(:first-of-type)
            border-left: none

        th
          border-bottom: none

          .tavuelo--select-all
            &:before
              content: 'All'
          .tavuelo--select-page
            margin-left: 5px
            &:before
              content: 'Page'
          .tavuelo--select-all,
          .tavuelo--select-page
            &:hover
              cursor: pointer
              text-decoration: underline
              color: crimson


      tr:not(:last-of-type)
        td
          border-bottom: none
      tr.tavuelo-no-data-row
        td
          text-align: center
        &:hover
          td
            background-color: inherit

    &__download-data-wrapper
      display: inline-block
      float: left

    &__button
      &--download-data
        padding: 5px 10px
        border-radius: 5px
        text-align: center
        color: #FFFFFF
        background: #4ab2d6
        border: 1px solid #4ab2d6

        &:hover
          cursor: pointer
          background: darken(#4ab2d6, 10)

  .tavuelo__search
    display: inline-block
    float: right
    text-align: right

    input
      margin: 0 0 15px 0
      padding: 5px

  // tooltip styles
  .tavuelo-tooltip
    cursor: help
    &.tavuelo-tooltips
      position: relative
      display: inline
      span
        position: absolute
        width: 140px
        color: #FFFFFF
        background: #000000
        line-height: 11px
        font-size: 10px
        text-align: center
        visibility: hidden
        border-radius: 6px
        font-weight: 400
        padding: 5px
        &:after
          content: ''
          position: absolute
          top: 100%
          left: 50%
          margin-left: -8px
          width: 0
          height: 0
          border-top: 8px solid #000000
          border-right: 8px solid transparent
          border-left: 8px solid transparent
    &:hover.tavuelo-tooltips span
      visibility: visible
      bottom: 30px
      left: 50%
      margin-left: -76px
      z-index: 999

  // sorting styles
  .tavuelo-sorting
    cursor: pointer
    position: absolute
    right: 2px
    top: 3px
    width: 10px
    height: 100%

    &:before
      content: ' '
      line-height: 0
      border-left: 5px solid transparent
      border-right: 5px solid transparent
      border-bottom: 5px solid #000
      width: 0
      height: 0
      position: absolute
      top: 0
      left: 0
    &:after
      content: ' '
      line-height: 0
      border-left: 5px solid transparent
      border-right: 5px solid transparent
      border-top: 5px solid #000
      width: 0
      height: 0
      position: absolute
      top: 8px
      left: 0

    &__active
      &.sorting-asc
        &:before
          border-bottom-color: crimson
      &.sorting-desc
        &:after
          border-top-color: crimson
</style>
