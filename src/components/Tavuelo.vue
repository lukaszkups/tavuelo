<template>
  <div class='tavuelo__wrapper'>
    <table class='tavuelo'>
      <thead>
        <tr>
          <th
            v-for='column in computedColumns'
            :key='column.id'
          >
            {{ column.title }}
          </th>
        </tr>
      </thead>
      <tbody>
        <tr
          v-for='row in computedData'
          :key='row.id'
        >
          <td
            v-for='column in computedColumns'
            :key='`${row.id}-${column.id}`'
          >
            <div class='cell'>{{ row[column.dataSource] }}</div>
          </td>
        </tr>
      </tbody>
    </table>
    <ul
      v-if='hasPagination'
      class='tavuelo__pagination'
    >
      <li
        class='tavuelo__pagination--first-page'
        @click='!Number.isNaN(activePage) && activePage !== 0 ? changePage(0) : false'
      >&laquo;</li>
      <li
        class='tavuelo__pagination--previous-page'
        @click='!Number.isNaN(activePage) && activePage !== 0 ? changePage(activePage - 1) : false'
      >&lsaquo;</li>
      <li
        v-for='page in pageList'
        :key='page'
        :class='["tavuelo__pagination--page", {"active-page": page === activePage}]'
        @click='changePage(page)'
      >{{ page + 1 }}</li>
      <li
        class='tavuelo__pagination--next-page'
        @click='!Number.isNaN(activePage) && activePage < lastPage
        ? changePage(activePage + 1)
        : false'
      >&rsaquo;</li>
      <li
        class='tavuelo__pagination--last-page'
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
    };
  },
  props: {
    data: {
      type: [Array, Function],
      default: () => [],
    },
    columns: {
      type: [Array, Function],
      default: () => [],
    },
    perPage: {
      type: Number,
      default: 0,
    },
  },
  computed: {
    computedColumns() {
      if (this.columns && this.columns.length) {
        return this.columns.map((col, index) => ({ ...col, id: index }));
      }
      return [];
    },
    indexedData() {
      if (this.data && this.data.length) {
        return this.data.map((row, index) => ({ ...row, id: index }));
      }
      return [];
    },
    computedData() {
      if (this.indexedData && this.indexedData.length) {
        if (this.hasPagination) {
          const startIndex = ((this.activePage + 1) * this.perPage) - 1;
          return this.indexedData.slice(startIndex, startIndex + this.perPage);
        }
        return this.indexedData;
      }
      return [];
    },
    hasPagination() {
      return !Number.isNaN(this.perPage) && this.perPage > 0;
    },
    pageList() {
      if (this.hasPagination && this.indexedData && this.indexedData.length) {
        if (this.perPage >= this.indexedData.length) {
          return [0];
        }
        const amountOfPages = Math.floor(this.indexedData.length / this.perPage);
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
</style>
