<template>
  <th
    :data-cy="title"
    class="font-medium relative pr-4 py-4"
    :class="{activate: isHeaderActive, border: isHeaderActive}"
  >
    {{title}}
    <span
      class="text-gray-500 absolute right-0"
      v-show="!isHeaderActive || isAscending"
    >&#9650</span>
    <span class="text-gray-500 absolute right-0" v-show="!isHeaderActive || !isAscending">&#9660</span>
  </th>
</template>
<script>
export default {
  name: "TableHeaderSort",
  props: {
    title: {
      type: String,
      required: true
    },
    isAscending: {
      type: Boolean,
      required: true
    },
    isActive: {
      type: String,
      required: false
    }
  },

  computed: {
    isHeaderActive() {
      return this.isActive == this.title
    }
  }
}
</script>
<style scoped>
th {
  color: #4a4a4a;
  border-left: none;
  border-right: none;
  border-bottom: none;
}

th:hover {
  cursor: default;
}

.activate {
  border-top-color: rgb(0, 136, 206);
  color: rgb(0, 136, 206);
}

span {
  font-size: 10px;
}

span:first-child {
  top: 16px;
}

span:last-child {
  bottom: 16px;
}
</style>

