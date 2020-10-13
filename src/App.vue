<template>
  <div id="app">
    <div class="inputs-group">
      <div class="cell">
        <label>工资：</label>
        <input type="number" placeholder="工资" v-model="wage" />
      </div>
      <div class="cell">
        <label>五险一金：</label>
        <input type="number" placeholder="五险一金" v-model="welfare" />
      </div>
      <div class="cell">
        <label>专项扣除总额：</label>
        <input type="number" placeholder="专项扣除总额" v-model="deduct" />
      </div>
      <div class="cell">
        <button @click="handleCalculate">计算</button>
      </div>
    </div>
    <h1 class="saving_tips">
      采用「专项附加累计预扣法」为您节约了 <span>{{ savingMoney }}</span> 元
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
import VTable from '@/components/VTable';

/* util function */

const isNumber = val => {
  return val && !isNaN(val);
};

const OLD = 'OLD';
const NEW = 'NEW';

const getTaxRate = (income, type) => {
  const getTaxPoint = (income, strategies) => {
    let taxInterval = Object.keys(strategies);
    return taxInterval.find(level => income <= level);
  };
  let oldConfig = {
    3000: {
      rate: 0.03,
      taxConst: 0
    },
    12000: {
      rate: 0.1,
      taxConst: 210
    },
    25000: {
      rate: 0.2,
      taxConst: 1410
    },
    35000: {
      rate: 0.25,
      taxConst: 2660
    },
    55000: {
      rate: 0.3,
      taxConst: 4410
    },
    80000: {
      rate: 0.35,
      taxConst: 7160
    },
    Infinity: {
      rate: 0.45,
      taxConst: 15160
    }
  };

  let newConfig = {
    36000: {
      rate: 0.03,
      taxConst: 0
    },
    144000: {
      rate: 0.1,
      taxConst: 2520
    },
    300000: {
      rate: 0.2,
      taxConst: 16920
    },
    420000: {
      rate: 0.25,
      taxConst: 31920
    },
    660000: {
      rate: 0.3,
      taxConst: 52920
    },
    960000: {
      rate: 0.35,
      taxConst: 85920
    },
    Infinity: {
      rate: 0.45,
      taxConst: 181920
    }
  };
  const config = type === OLD ? oldConfig : newConfig;
  const { rate, taxConst } = config[getTaxPoint(income, config)];
  return {
    rate: rate,
    taxRate: rate * 100 + '%',
    totalTax: income * rate - taxConst,
    taxConst: taxConst
  };
};

export default {
  name: 'app',
  components: {
    VTable
  },
  data() {
    return {
      wage: 25000,
      welfare: 3000,
      taxPoint: 5000,
      deduct: 1500,
      columns: [
        {
          prop: 'month',
          label: '月份'
        },
        {
          prop: 'wage',
          label: '工资'
        },
        {
          prop: 'taxPoint',
          label: '起征点'
        },
        {
          prop: 'welfare',
          label: '五险一金'
        },
        {
          prop: 'deduct',
          label: '专项附加'
        },
        {
          prop: 'payTaxMoney',
          label: '需扣税工资'
        },
        {
          prop: 'taxRate',
          label: '适用税率'
        },
        {
          prop: 'taxConst',
          label: '速算扣除数'
        },
        {
          prop: 'totalTax',
          label: '当月税额'
        },
        {
          prop: 'income',
          label: '到手工资'
        }
      ],
      newTableData: [],
      oldTableData: []
    };
  },
  computed: {
    savingMoney() {
      let newIncome = 0;
      let oldIncome = 0;
      this.newTableData.forEach(item => {
        newIncome += item.income;
      });
      this.oldTableData.forEach(item => {
        oldIncome += item.income;
      });
      let totalSaving = newIncome - oldIncome;
      return totalSaving && totalSaving.toFixed(2);
    }
  },
  created() {},
  mounted() {
    this.handleCalculate();
  },
  methods: {
    getData() {
      this.newTableData = this.getTableData(NEW);
      this.oldTableData = this.getTableData(OLD);
    },
    getTableData(type) {
      let result = [];
      let count = type === OLD ? 1 : 0;
      for (let i = 1; i <= 12; i++) {
        getData.call(this, i);
      }
      function getData(i) {
        let wage = this.wage * (count || i);
        let taxPoint = this.taxPoint * (count || i);
        let welfare = this.welfare * (count || i);
        let deduct = count ? 0 : this.deduct * (count || i); // 旧算法中没有专项附加
        let payTaxMoney = wage - taxPoint - welfare - deduct;
        let { totalTax, taxRate, taxConst } = getTaxRate(payTaxMoney, type);
        if (!count) {
          result.forEach(item => {
            totalTax -= item.totalTax;
          });
        }
        let income = this.wage - totalTax - this.welfare;
        result.push({
          month: i,
          wage,
          taxPoint,
          welfare,
          deduct: count ? '-' : deduct,
          payTaxMoney,
          taxRate,
          taxConst,
          totalTax: totalTax,
          income: income
        });
      }
      return result;
    },
    handleCalculate() {
      if (
        isNumber(this.wage) &&
        isNumber(this.welfare) &&
        isNumber(this.deduct)
      ) {
        if (this.wage < this.taxPoint) {
          alert('您的工资未超过起征点，不需交税');
        } else if (this.wage - this.taxPoint - this.welfare - this.deduct > 0) {
          this.getData();
        } else {
          alert(
            `出错了！工资应大于起征点、社保、专项扣除之和(${
              Number(this.taxPoint) + Number(this.welfare) + Number(this.deduct)
            })`
          );
        }
      } else {
        alert('请输入正确的格式(数字)');
      }
    },
    handleSummary({ columns, data }) {
      let totalIncome = 0;
      data.forEach(item => {
        totalIncome += item.income;
      });
      return columns.map((column, index) => {
        if (index === 0) {
          return '合计';
        }
        if (column.prop !== 'income') {
          return '-';
        } else {
          return parseInt(totalIncome);
        }
      });
    }
  }
};
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
input,
button {
  border: 1px solid #ddd;
  appearance: none;
}
.v__table_title {
  text-align: center;
}
</style>
