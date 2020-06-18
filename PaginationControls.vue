<template>
  <div class="pagination-controls h-12 flex justify-around items-center">
    <i
      data-cy="pagination-start"
      class="fa fa-angle-double-left w-4 font-extrabold fa-black text-2xl mr-2"
      @click="$emit('start')"
    ></i>
    <i
      data-cy="pagination-prev"
      class="fa fa-angle-left font-bold fa-black w-4 text-2xl mx-2"
      @click="$emit('prev')"
    ></i>
    <div class="pagination-nums flex justify-between w-full">
      <span
        data-cy="pagination-page-selector"
        :class="{invisible: !pages[0]}"
        @click="$emit('pageChangeRequest', 0)"
      >{{pages[0]}}</span>
      <span
        data-cy="pagination-page-selector"
        :class="{invisible: !pages[1]}"
        @click="$emit('pageChangeRequest', 1)"
      >{{pages[1]}}</span>
      <span data-cy="pagination-page-selector" class="bg-white currentPage">{{pages[2]}}</span>
      <span
        data-cy="pagination-page-selector"
        @click="$emit('pageChangeRequest', 3)"
        v-if="shouldShowPage(3)"
      >{{pages[3]}}</span>
      <!-- To prevent jarring UI effect when user reaches end of pagination -->
      <span v-else class="space-filler">&nbsp;</span>
      <span
        data-cy="pagination-page-selector"
        @click="$emit('pageChangeRequest', 4)"
        v-if="shouldShowPage(4)"
      >{{pages[4]}}</span>
      <span data-cy="pagination-page-selector" v-else class="space-filler">&nbsp;</span>
    </div>
    <i
      data-cy="pagination-next"
      class="fa fa-angle-right font-bold fa-black w-4 text-2xl mx-2"
      @click="$emit('next')"
    ></i>
    <i
      data-cy="pagination-end"
      class="fa fa-angle-double-right w-4 fa-black font-extrabold text-2xl ml-2"
      @click="$emit('end')"
    ></i>
  </div>
</template>
<script>
export default {
  name: "PaginationControls",

  props: {
    pages: {
      type: Array,
      required: true,
      default: function() {
        return ["", "", 1, 2, 3];
      }
    },

    paginateBy: {
      type: Number,
      required: true,
      default: 25
    },

    totalResults: {
      type: Number,
      required: true
    }
  },

  methods: {
    /**
     * Checking whether to show the later pages
     * based on total results passed in.
     * @param {Number} - i The index of the page number on the pages prop
     */
    shouldShowPage(i) {
      if (this.pages[i] * this.paginateBy <= this.totalResults) {
        return true;
      } else {
        /**
         * Case when paginateBy & current page product are greater than totalResults
         * but there are still items left to show
         */

        if (
          this.pages[i] * this.paginateBy - this.totalResults <
          this.paginateBy
        ) {
          return true;
        }
        return false;
      }
    }
  }
};
</script>
<style scoped>
.pagination-controls {
  min-width: 135px;
  max-width: 180px;
}

span {
  min-width: 16px;
  display: flex;
  justify-content: center;
  align-items: center;
  border: 1px solid transparent;
  cursor: pointer;
  padding: 1px;
}

.currentPage {
  border: 1px solid #979797;
  background-image: linear-gradient(
    rgb(255, 255, 255) 0%,
    rgb(220, 220, 220) 100%
  );
}

i {
  color: #979797;
}

.space-filler {
  cursor: default;
}
</style>
