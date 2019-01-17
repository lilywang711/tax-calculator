<template>
  <div id="app">
    <div class="inputs-group">
      <div class="cell">
        <label>工资：</label>
        <input type="number" placeholder="工资" v-model="wage">
      </div>
      <div class="cell">
        <label>五险一金：</label>
        <input type="number" placeholder="五险一金" v-model="welfare">
      </div>
      <div class="cell">
        <label>专项扣除总额：</label>
        <input type="number" placeholder="专项扣除总额" v-model="deduct">
      </div>
      <div class="cell">
        <button @click="handleCalcu">计算</button>
      </div>
    </div>
    <h1 class="saving_tips">
      采用「专项附加累计预扣法」为您节约了 <span>{{savingMoney}}</span> 元
    </h1>
    <div class="v__table">
      <h2 class="v__table_title">专项附加年度工资表(2019)</h2>
      <v-table
       :columns="columns"
       :data="newTableData"
       :summary-method="handleSummary"
      />
    </div>
    <div class="v__table">
      <h2 class="v__table_title">无专项附加年度工资表(旧)</h2>
      <v-table
       :columns="columns"
       :data="oldTableData"
       :summary-method="handleSummary"
      />
    </div>
  </div>
</template>

<script>
import VTable from '@/components/VTable'

/* util function */
const isNumber = (val) => {
  return val && !isNaN(val)
}
export default {
  name: 'app',
  components: {
    VTable
  },
  data () {
    return {
      wage: 25000,
      welfare: 3000,
      taxPoint: 5000,
      deduct: 1500,
      columns: [{
        prop: 'month',
        label: '月份'
      }, {
        prop: 'wage',
        label: '工资'
      }, {
        prop: 'taxPoint',
        label: '起征点'
      }, {
        prop: 'welfare',
        label: '五险一金'
      }, {
        prop: 'deduct',
        label: '专项附加'
      }, {
        prop: 'payTaxMoney',
        label: '需扣税工资'
      }, {
        prop: 'taxRate',
        label: '适用税率'
      }, {
        prop: 'taxConst',
        label: '速算扣除数'
      }, {
        prop: 'totalTax',
        label: '当月税额'
      }, {
        prop: 'income',
        label: '到手工资'
      }],
      newTableData: [],
      oldTableData: []
    }
  },
  computed: {
    savingMoney () {
      let newIncome = 0
      let oldIncome = 0
      this.newTableData.forEach(item => {
        newIncome += item.income
      })
      this.oldTableData.forEach(item => {
        oldIncome += item.income
      })
      let totalSaving = newIncome - oldIncome
      return totalSaving && totalSaving.toFixed(2)
    }
  },
  created () {
  },
  mounted () {
    this.handleCalcu()
  },
  methods: {
    getData () {
      this.newTableData = this.calculateData()
      this.oldTableData = this.calculateData({ type: 'old' })
    },
    calculateData ({ type } = {}) {
      let result = []
      let count = type === 'old' ? 1 : 0
      for (let i = 1; i <= 12; i++) {
        let wage = this.wage * (count || i)
        let taxPoint = this.taxPoint * (count || i)
        let welfare = this.welfare * (count || i)
        let deduct = count ? 0 : this.deduct * (count || i) // 旧算法中没有专项附加
        let payTaxMoney = wage - taxPoint - welfare - deduct
        let { totalTax, taxRate, taxConst } = this.getTaxRate(payTaxMoney, type)
        if (!count) {
          result.forEach(item => {
            totalTax -= item.totalTax
          })
        }
        let income = this.wage - totalTax - this.welfare
        result.push({
          month: i,
          wage,
          taxPoint,
          welfare,
          deduct: count ? '-' : deduct,
          payTaxMoney,
          taxRate,
          taxConst,
          totalTax,
          income
        })
      }
      return result
    },
    getTaxRate (income, type) {
      let totalTax = 0
      let rate = 0
      let taxConst = 0
      let taxInterval = []
      let taxConstInterval = []
      if (type === 'old') {
        taxInterval = [3000, 12000, 25000, 35000, 55000, 80000]
        taxConstInterval = [0, 210, 1410, 2660, 4410, 7160, 15160]
      } else {
        taxInterval = [36000, 144000, 300000, 420000, 660000, 960000]
        taxConstInterval = [0, 2520, 16920, 31920, 52920, 85920, 181920]
      }
      if (income <= taxInterval[0]) {
        rate = 0.03
        totalTax = income * rate
        taxConst = taxConstInterval[0]
      } else if (income <= taxInterval[1]) {
        rate = 0.1
        taxConst = taxConstInterval[1]
        totalTax = income * rate - taxConst
      } else if (income <= taxInterval[2]) {
        rate = 0.2
        taxConst = taxConstInterval[2]
        totalTax = income * rate - taxConst
      } else if (income <= taxInterval[3]) {
        rate = 0.25
        taxConst = taxConstInterval[3]
        totalTax = income * rate - taxConst
      } else if (income <= taxInterval[4]) {
        rate = 0.3
        taxConst = taxConstInterval[4]
        totalTax = income * rate - taxConst
      } else if (income <= taxInterval[5]) {
        rate = 0.35
        taxConst = taxConstInterval[5]
        totalTax = income * rate - taxConst
      } else {
        rate = 0.45
        taxConst = taxConstInterval[6]
        totalTax = income * rate - taxConst
      }
      return {
        totalTax: Number(totalTax.toFixed(2)),
        taxRate: rate * 100 + '%',
        taxConst
      }
    },
    handleCalcu () {
      if (isNumber(this.wage) && isNumber(this.welfare) && isNumber(this.deduct)) {
        if (this.wage < this.taxPoint) {
          alert('您的工资未超过起征点，不需交税')
        } else if (this.wage - this.taxPoint - this.welfare - this.deduct > 0) {
          this.getData()
        } else {
          console.log(this.taxPoint, Number(this.welfare), this.deduct)
          alert(`出错了！工资应大于起征点、社保、专项扣除之和(${Number(this.taxPoint) + Number(this.welfare) + Number(this.deduct)})`)
        }
      } else {
        alert('请输入正确的格式(数字)')
      }
    },
    handleSummary ({ columns, data }) {
      let totalIncome = 0
      data.forEach(item => {
        totalIncome += item.income
      })
      return columns.map((column, index) => {
        if (index === 0) {
          return '合计'
        }
        if (column.prop !== 'income') {
          return '-'
        } else {
          return totalIncome
        }
      })
    }
  }
}
</script>

<style lang="scss">
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  margin-top: 20px;
  text-align: center;
}
h2 {
  margin: 10px auto;
  font-size: 18px;
}
.saving_tips {
  font-size: 20px;
  margin: 20px 0;
  text-align: center;
  span {
    color: red;
  }
}
.inputs-group {
  display: flex;
  justify-content: center;
  width: 100%;
}
.inputs-group input {
  margin-right: 16px;
}
input, button {
  border: 1px solid #ddd;
  appearance: none;
}
.v__table_title {
  text-align: center;
}
</style>
