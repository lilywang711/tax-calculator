<template>
  <table  border="1">
    <thead>
      <tr>
        <th v-for="item in columns" :key="item.prop">{{item.label}}</th>
      </tr>
    </thead>
    <tbody ref="tbody">
      <tr v-for="(row, index) in data" :key="index">
        <td v-for="(cell, index) in row" :key="index">{{paddingZero(cell)}}</td>
      </tr>
      <tr v-if="summaryMethod">
        <td v-for="(column, cellIndex) in columns" :key="cellIndex">{{sums[cellIndex]}}</td>
      </tr>
    </tbody>
  </table>
</template>
<script>
const paddingZero = (num) => {
  return ('' + num).split('.')[1] ? num.toFixed(2) : num
}
export default {
  props: {
    columns: Array,
    data: Array,
    showSummary: Boolean,
    summaryMethod: Function
  },
  data () {
    return {
      paddingZero
    }
  },
  computed: {
    columnsProps () {
      return this.columns.map(item => item.prop)
    },
    sums () {
      return this.summaryMethod({ columns: this.columns, data: this.data })
    }
  },
  render () {
    return ''
  },
  created () {
  },
  methods: {
  }
}
</script>
<style lang='scss' >
table{
  border-collapse: collapse;
}
table,th, td{
  border: 1px solid black;
}
th {
  height: 30px;
  min-width: 80px;
  width: auto;
}
td {
  height: 24px;
  text-align: center;
}
table {
  margin: 0 auto 30px;
}
</style>
