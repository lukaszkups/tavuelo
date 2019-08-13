<template>
  <div class='tavuelo__wrapper'>
    <div v-if='hasSearch' class='tavuelo__search'>
      <input
        v-model='searchQuery'
        placeholder='Search'
      />
    </div>
    <table class='tavuelo'>
      <thead>
        <tr>
          <th
            v-for='column in computedColumns'
            :key='column.tavuelo_id'
          >
            <div v-if='column.tooltip'>
              <div class='tavuelo-tooltip tavuelo-tooltips'>
                {{ column.title }}
                <span>{{ column.tooltip }}</span>
              </div>
            </div>
            <div v-else>{{ column.title }}</div>
          </th>
        </tr>
      </thead>
      <tbody>
        <tr
          v-for='row in computedData'
          :key='row.tavuelo_id'
        >
          <td
            v-for='column in computedColumns'
            :key='`${row.tavuelo_id}-${column.tavuelo_id}`'
          >
            <div
              v-if='!column.type || column.type === "text"'
              class='cell'
            >{{ row[column.dataSource] }}</div>
            <div
              v-else-if='column.type === "bool"'
              class='cell'
            >
              <span v-if='row[column.dataSource]'>&check;</span>
              <span v-else>&cross;</span>
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
/* eslint-disable max-len, arrow-body-style, arrow-parens, array-callback-return */

export default {
  name: 'Tavuelo',
  data() {
    return {
      activePage: 0,
      searchQuery: '',
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
    searchCaseSensitive: {
      type: Boolean,
      default: false,
    },
    searchColumns: {
      type: Array,
      default: () => [],
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
      if (this.data && this.data.length) {
        return this.data.map((row, index) => ({ ...row, tavuelo_id: index }));
      }
      return [];
    },
    filteredData() {
      if (this.indexedData && this.indexedData.length) {
        let indexedDataCopy = [...this.indexedData];
        indexedDataCopy = indexedDataCopy.filter(entry => {
          if (this.searchCaseSensitive) {
            return this.searchColumns.find(searchColumn => entry[searchColumn].includes(this.searchQuery));
          }
          const lowercaseQuery = this.searchQuery.toLowerCase();
          return this.searchColumns.find(searchColumn => entry[searchColumn].toLowerCase().includes(lowercaseQuery));
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
        if (this.perPage >= indexedDataCopy.length) {
          return [0];
        }
        const amountOfPages = Math.ceil(indexedDataCopy.length / this.perPage);
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
  },
  watch: {
    filteredData() {
      this.activePage = 0;
    },
  },
};
</script>
<style lang='sass'>
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
    tbody
      tr
        td
          border: 1px solid grey
        &:hover
          td
            background: lavender

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
        // height: 30px
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
</style>