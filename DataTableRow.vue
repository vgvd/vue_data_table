<template>
  <tr
    data-cy="tbl-row"
    class="w-full h-16 p-0 border"
    :class="{'bg-gray-100' : index % 2 !== 0}"
    :style="rowCSS"
  >
    <td
      data-cy="tbl-cell"
      class="w-full h-full flex justify-center items-center text-center"
      v-for="value in dataToShow"
      v-html="value"
    ></td>
    <slot name="actions">
      <!-- action items go here -->
    </slot>
  </tr>
</template>

<script>
export default {
  name: "DataTableRow",

  props: {
    rowData: {
      type: Object,
      required: true
    },

    index: {
      type: Number,
      required: true
    },

    rowCSS: {
      type: Object,
      required: true
    },

    keysToFilter: {
      type: Array,
      required: true
    }
  },

  computed: {
    dataToShow() {
      let obj = {};
      for (let key of this.keysToFilter) {
        if (key == "lockedBy") {
          obj["lockedHTML"] = this.rowData["lockedHTML"];
        } else {
          obj[key] = this.rowData[key];
        }
      }
      return obj;
    }
  }
};
</script>