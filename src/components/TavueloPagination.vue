<template>
  <ul
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
    <template v-if='showAllPages'>
      <li
        v-for='page in pageList'
        :key='page'
        :class='["tavuelo__pagination--page", {"active-page": page === activePage}]'
        @click='changePage(page)'
      >{{ page + 1 }}</li>
    </template>
    <template v-else>
      <li class='tavuelo__pagination-text'>Page </li>
      <li class='tavuelo__pagination-text'>
        <input
          class='tavuelo__pagination-input'
          :value='activePage + 1'
          @blur='(e) => changePage(e.target.value, true)'
        />
      </li>
      <li class='tavuelo__pagination-text'> of {{ lastPage + 1 }}</li>
    </template>
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
</template>
<script>
export default {
  name: 'TavueloPagination',
  props: {
    activePage: {
      type: Number,
      required: true,
    },
    pageList: {
      type: Array,
      required: true,
    },
    showAllPages: {
      type: Boolean,
      required: true,
    },
  },
  computed: {
    lastPage() {
      return this.pageList.length ? this.pageList.length - 1 : 0;
    },
  },
  methods: {
    changePage(page, subtractOne) {
      this.showJumpToPage = false;
      if (Number.isNaN(page) || page <= 0 || page - 1 > this.lastPage) {
        this.$emit('update:activePage', 0);
      } else if (subtractOne) {
        this.$emit('update:activePage', Number(page - 1));
      } else {
        this.$emit('update:activePage', Number(page));
      }
    },
  },
};
</script>
<style lang="sass" scoped>
.tavuelo__wrapper
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

      &.tavuelo__pagination-text
        min-width: 45px

        &:hover
          font-weight: normal
          cursor: default

      &.active-page
        font-weight: bold
        text-decoration: underline

      &.disabled
        color: silver
        &:hover
          cursor: not-allowed

    &-input
      width: 30px
      border: 1px solid grey
</style>
