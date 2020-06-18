<template>
  <div class="max-width-card mx-auto bg-white w-full p-6 shadow mb-12">
    <header
      class="flex justify-between w-full h-12 border-gray-200 bg-gray-100 items-center mb-2 border relative pl-4"
    >
      <input
        type="text"
        placeholder="Search..."
        class="w-1/5 h-8 border border-gray-400 ml-2 pl-2 mr-4"
        v-model="searchText"
      />
      <div v-show="showPaginationControls" class="pagination-amt-selection w-36 absolute">
        <span>Show</span>
        <select class="border bg-white" v-model.number="amountToPaginateBy" @change="moveToStart">
          <option v-if="isPaginationAmountValid(25)">25</option>
          <option v-if="isPaginationAmountValid(50)">50</option>
          <option v-if="isPaginationAmountValid(100)">100</option>
        </select>
        <span>rows</span>
      </div>
      <PaginationControls
        class="top-pagination-controls"
        v-show="showPaginationControls"
        @start="moveToStart"
        @end="moveToEnd"
        @next="moveNext"
        @prev="movePrev"
        @pageChangeRequest="goToPage"
        :pages="paginationPages"
        :paginateBy="amountToPaginateBy"
        :totalResults="numRows"
      />
      <p
        v-if="showPaginationControls"
        class="results-count mr-4"
      >Showing {{displayStartPoint}} to {{displayEndPoint}} of {{numRows}} results</p>
      <p v-else class="mr-6">{{numRows}} results</p>
    </header>
    <table class="w-full border-collapse" data-cy="data-table">
      <thead>
        <tr
          class="cust-table-row w-full border border-gray-200 bg-gray-100"
          :style="tableSetup.tableCSS"
          v-if="currentPageResults.length > 0"
        >
          <span
            class="w-full h-full flex justify-center items-center text-center"
            v-for="(obj, i) in tableSetup.headers"
            :key="i"
          >
            <TableHeaderSort
              v-if="obj.type"
              :title="obj.title"
              :isAscending="ascending"
              :isActive="activeHeader"
              @click.native="getSortedResults(obj.title, obj.type)"
              :key="obj.title"
            />
            <th class="font-medium" v-else>{{obj.title}}</th>
          </span>
        </tr>
        <tr class="cust-table-row w-full border border-gray-200 bg-gray-100" v-else>
          <th class="font-medium">No Data Available</th>
        </tr>
      </thead>
      <tbody>
        <DataTableRow
          v-for="(row, i) in currentPageResults"
          :rowData="row"
          :index="i"
          :rowCSS="tableSetup.tableCSS"
          :keysToFilter="keysToFilter"
          :key="i"
        >
          <td
            v-if="row.actions.length > 0"
            class="w-full h-full flex justify-around items-center text-center"
            slot="actions"
          >
            <span
              :data-cy="action.text"
              v-for="action in row.actions"
              class="cursor-pointer"
              @click="$emit(action.eventEmitter, row)"
              :key="action.text"
            >
              <i :class="action.faClass"></i>
              {{action.text}}
            </span>
          </td>
        </DataTableRow>
      </tbody>
      <tfoot class="flex justify-center items-center w-full h-12 bg-gray-100 mt-2">
        <PaginationControls
          v-show="showPaginationControls"
          @start="moveToStart"
          @end="moveToEnd"
          @next="moveNext"
          @prev="movePrev"
          @pageChangeRequest="goToPage"
          :pages="paginationPages"
          :paginateBy="amountToPaginateBy"
          :totalResults="numRows"
        />
      </tfoot>
    </table>
  </div>
</template>

<script>
import _ from "lodash"
import TableHeaderSort from "./TableHeaderSort"
import PaginationControls from "./PaginationControls"
import DataTableRow from "./DataTableRow"
import sortingMethods from "../../mixins/sortingMethods"

export default {
  name: "DataTable",

  components: {
    DataTableRow,
    TableHeaderSort,
    PaginationControls
  },

  props: {
    rows: {
      type: Array,
      required: true
    },

    tableSetup: {
      type: Object,
      required: true
      /**
       * tableSetup prop object example
       * {
       *  headers: [{title: 'MANIFEST ID', type: 'manifestID'}],
       *  tableCSS: {
       *    minWidth: '700px',
       *    display: 'grid',
       *    "grid-template-columns": "repeat(3, 1fr)"
       *  }
       *
       * }
       */
    },

    explicitFilteringKeys: {
      type: Array,
      required: false
    },

    allowPagination: {
      type: Boolean,
      required: false,
      default: false
    }
  },

  mixins: [sortingMethods],

  data() {
    return {
      searchText: "",
      sortRequest: false,
      sortRequestType: null,
      activeHeader: null,
      ascending: false,
      currentPage: 1,
      amountToPaginateBy: 25
    }
  },

  methods: {
    isPaginationAmountValid(num) {
      return this.numRows % num !== this.numRows
    },

    moveToStart() {
      this.currentPage = 1
    },

    moveToEnd() {
      this.currentPage = this.lastPagePossible
    },

    movePrev() {
      if (this.currentPage > 1) {
        this.currentPage--
      }
    },

    moveNext() {
      if (this.currentPage < this.lastPagePossible) {
        this.currentPage++
      }
    },

    goToPage(pageIndex) {
      this.currentPage = this.paginationPages[pageIndex]
    },

    getSortedResults(title, type) {
      this.sortRequest = true
      this.ascending = !this.ascending
      this.sortRequestType = type
      this.activeHeader = title
    }
  },

  computed: {
    filteredRows() {
      const vm = this
      vm.currentPage = 1
      vm.amountToPaginateBy = 25
      if (vm.searchText) {
        return vm.rows.filter(obj => {
          for (const key of vm.keysToFilter) {
            let currentVal = obj[key]
            if (currentVal != null && currentVal != undefined) {
              currentVal = String(currentVal).toLowerCase()
              let searchTerm = vm.searchText.toLowerCase()
              if (currentVal.indexOf(searchTerm) >= 0) {
                return obj
              }
            }
          }
        })
      }
      return vm.rows
    },

    sortedRows() {
      const key = this.sortRequestType
      const asc = this.ascending
      let data = _.cloneDeep(this.filteredRows)
      switch (key) {
        case "order_id":
        case "id":
        case "reference":
        case "document_count":
          data = this.sortByNum(data, key, asc)
          break
        case "submitted_at":
          data = this.sortByDate(data, key, asc)
          break
        default:
          data = this.sortByString(data, key, asc)
      }
      return data
    },

    numRows() {
      return this.filteredRows.length
    },

    rowsMoreThan25() {
      return this.numRows > 25
    },

    showPaginationControls() {
      return this.rowsMoreThan25 && this.allowPagination
    },

    displayEndPoint() {
      let endPoint = this.amountToPaginateBy * this.currentPage
      if (endPoint <= this.numRows) {
        return endPoint
      } else {
        return this.numRows
      }
    },

    displayStartPoint() {
      return (
        this.amountToPaginateBy * this.currentPage - this.amountToPaginateBy + 1
      )
    },

    lastPagePossible() {
      let num = this.numRows / this.amountToPaginateBy
      if (this.numRows % this.amountToPaginateBy > 0) {
        num = String(num)
        const indexOfPeriod = num.indexOf(".")
        const lastPage = Number(num.slice(0, indexOfPeriod))
        return lastPage + 1
      } else {
        return num
      }
    },

    paginationPages() {
      const num = this.currentPage
      if (num <= 1) {
        return ["", "", 1, 2, 3]
      } else if (num == 2) {
        return ["", 1, 2, 3, 4]
      } else {
        return [num - 2, num - 1, num, num + 1, num + 2]
      }
    },

    currentPageResults() {
      const start = this.displayStartPoint - 1
      const end = this.displayEndPoint
      let dataToShow = this.filteredRows.slice(start, end)
      if (this.sortRequest) {
        dataToShow = this.sortedRows.slice(start, end)
      }
      return dataToShow
    },

    keysToFilter() {
      if (!this.explicitFilteringKeys) {
        return this.tableSetup.headers.map(obj => obj.type)
      }
      return this.explicitFilteringKeys
    }
  }
}
</script>

<style lang="scss" scoped>
.pagination-amt-selection {
  display: flex;
  justify-content: space-between;
  align-items: center;
  left: 265px;
}
.top-pagination-controls {
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);
}
</style>