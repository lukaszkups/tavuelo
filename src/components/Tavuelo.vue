<template>
  <div class='tavuelo__wrapper'>
    <div v-if='hasSearch' class='tavuelo__search'>
      <input
        v-model='searchQuery'
        placeholder='Search'
      />
    </div>
    <table
      :class='["tavuelo", {
        "tavuelo--flex-table": this.useFlex,
        "tavuelo--wrap-content": this.wrapContent,
      }]'
    >
      <thead>
        <tr>
          <th
            v-for='column in computedColumns'
            :key='column.tavuelo_id'
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
                :class='["tavuelo-sorting", {
                  "tavuelo-sorting__active": column.dataSource === currentSortDataName,
                  "sorting-asc": column.dataSource === currentSortDataName && currentSortDirection === "asc",
                  "sorting-desc": column.dataSource === currentSortDataName && currentSortDirection === "desc",
                }]'
                @click='toggleSorting(column)'
              ></div>
            </div>
          </th>
        </tr>
      </thead>
      <tbody>
        <tr
          v-for='row in computedData'
          :key='row.tavuelo_id'
          @click='rowClick && typeof rowClick === "function" ? rowClick(row) : false'
        >
          <td
            v-for='column in computedColumns'
            :key='`${row.tavuelo_id}-${column.tavuelo_id}`'
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
          </td>
        </tr>
      </tbody>
    </table>
    <ul
      v-if='hasPagination'
      class='tavuelo__pagination'
    >
      <li
        :class='["tavuelo__pagination--first-page", {
          "disabled": !activePage || activePage === 0
        }]'
        @click='!Number.isNaN(activePage) && activePage !== 0 ? changePage(0) : false'
      >&laquo;</li>
      <li
        :class='["tavuelo__pagination--previous-page", {
          "disabled": !activePage || activePage === 0
        }]'
        @click='!Number.isNaN(activePage) && activePage !== 0 ? changePage(activePage - 1) : false'
      >&lsaquo;</li>
      <li
        v-for='page in pageList'
        :key='page'
        :class='["tavuelo__pagination--page", {"active-page": page === activePage}]'
        @click='changePage(page)'
      >{{ page + 1 }}</li>
      <li
        :class='["tavuelo__pagination--next-page", {
          "disabled": isNaN(activePage) || activePage === lastPage
        }]'
        @click='!Number.isNaN(activePage) && activePage < lastPage
        ? changePage(activePage + 1)
        : false'
      >&rsaquo;</li>
      <li
        :class='["tavuelo__pagination--last-page", {
          "disabled": isNaN(activePage) || activePage === lastPage
        }]'
        @click='!Number.isNaN(activePage) && activePage !== lastPage
          ? changePage(lastPage)
          : false'
      >&raquo;</li>
    </ul>
  </div>
</template>
<script>
export default {
  name: 'Tavuelo',
  data() {
    return {
      activePage: 0,
      dataCopy: [],
      searchQuery: '',
      currentSortDataName: '',
      currentSortDirection: '',
    };
  },
  props: {
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
  },
  computed: {
    computedColumns() {
      if (this.columns && this.columns.length) {
        return this.columns.map((col, index) => ({ ...col, tavuelo_id: index }));
      }
      return [];
    },
    indexedData() {
      if (this.dataCopy && this.dataCopy.length) {
        return this.dataCopy.map((row, index) => ({ ...row, tavuelo_id: index }));
      }
      return [];
    },
    filteredData() {
      if (this.customFiltering && typeof this.customFiltering === 'function') {
        return this.customFiltering([...this.indexedData], this.searchQuery);
      }
      if (this.indexedData && this.indexedData.length) {
        let indexedDataCopy = [...this.indexedData];
        indexedDataCopy = indexedDataCopy.filter(entry => {
          if (this.searchCaseSensitive) {
            return this.searchColumns.find(searchColumn => String(entry[searchColumn]).includes(this.searchQuery));
          }
          const lowercaseQuery = this.searchQuery.toLowerCase();
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
    lastPage() {
      return this.hasPagination && this.pageList.length ? this.pageList.length - 1 : 0;
    },
  },
  methods: {
    changePage(page) {
      this.activePage = page;
    },
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
      let dataCopy = [...this.dataCopy];
      if (this.customSortRules && Object.prototype.hasOwnProperty.call(this.customSortRules, dataSourceName)) {
        dataCopy = this.customSortRules[dataSourceName](dataCopy, direction);
      } else {
        dataCopy.sort((a, b) => String(a[dataSourceName]).localeCompare(String(b[dataSourceName])));
        if (direction === 'desc') {
          dataCopy.reverse();
        }
      }
      this.dataCopy = dataCopy;
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
  },
  watch: {
    filteredData() {
      this.activePage = 0;
    },
    data(newVal) {
      this.dataCopy = [...newVal];
    },
  },
  created() {
    this.dataCopy = [...this.data];
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

      tr:not(:last-of-type)
        td
          border-bottom: none

  .tavuelo__pagination
    display: block
    list-style-type: none
    margin: 15px 0
    padding: 0

    li
      display: inline-block
      height: 20px
      line-height: 20px
      text-align: center
      width: 20px

      &:hover
        cursor: pointer
        font-weight: bold

      &.active-page
        font-weight: bold
        text-decoration: underline

      &.disabled
        color: silver
        &:hover
          cursor: not-allowed

  .tavuelo__search
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
