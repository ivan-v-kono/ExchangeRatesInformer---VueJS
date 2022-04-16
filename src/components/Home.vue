<template>

    <a-row justify="start">
      <a-col :offset="2">
        <h1>Currencies today</h1>
      </a-col>
    </a-row>
    <!--<h4>{{new Date()}}</h4>--><!--:gutter="[16,16]"-->
    <h4 class="date">{{new Date().toJSON().slice(0,10).replace(/-/g,'/')}}</h4>
    <a-row class="currency"
        v-for="(currency,index) in userCurrenciesData"
        :key="index"
        justify="space-around"
    >

      <a-col
          :xs={span:12,offset:4}
          :sm={span:10,offset:6}
          :md={span:8,offset:7}
          :lg={span:6,offset:8}
      ><!---->
        <a-button
            type="primary"
            size="small"
            shape="circle"
            @click="removeCurrency(index)"
        >
          X
        </a-button>
        {{ currency.Cur_Abbreviation }} for {{ currency.Cur_Scale }}
        <a-divider />
      </a-col>
      <a-col :span="8" :md="9" :lg="10"><!---->
        {{ currency.Cur_OfficialRate }}
      </a-col>
    </a-row>
  <a-row>
    <a-col
        :xs={span:4,offset:4}
        :sm={offset:6}
        :md={offset:7}
        :lg={offset:8}
    >
      <a-button
          class="add"
          @click="this.visibilitySelect = true"
      >
        + Add currency
      </a-button>
      <select
          :xs={span:2,offset:1}
          v-model="selected"
          multiple
          v-if="visibilitySelect"
          @click="addCurrency">
        <option
            v-for="currency in allCurrenciesData"
            :key="currency.Cur_ID"
        >
          {{ currency.Cur_Abbreviation }}
        </option>
      </select>
    </a-col>
  </a-row>

  <!--<a-col
      :xs={offset:4}
      :sm={offset:6}
      :md={offset:7}
      :lg={offset:8}
  >

  </a-col>-->

</template>

<script>
export default {
  name: 'TheHome',
  data: () =>({
    allCurrenciesData: [],
    userCurrencies: ["RUB", "USD", "EUR"],
    userCurrenciesData: [],
    visibilitySelect: false,
    selected: []
  }),
  methods: {
    async getApiData() {
      const response = await fetch('https://www.nbrb.by/api/exrates/rates?periodicity=0')
      this.allCurrenciesData = ( await response.json() )
      .map(item => {
        return {
          Cur_ID: item.Cur_ID,
          Cur_Abbreviation: item.Cur_Abbreviation,
          Cur_Scale: item.Cur_Scale,
          Cur_Name: item.Cur_Name,
          Cur_OfficialRate: item.Cur_OfficialRate
        }
      })
    },
    removeCurrency(index) {
      this.userCurrencies.splice(index,1)
      this.userCurrenciesData.splice(index,1)
    },
    addCurrency() {
      if(!this.userCurrencies.includes(this.selected[0])) {
        this.userCurrencies.push(this.selected[0])
        this.addCurrencyData(this.selected[0])
      }
        else alert('currency already selected')
      this.visibilitySelect = false
    },
    addCurrencyData(currency) {
      this.userCurrenciesData.push(
        this.allCurrenciesData.find(item => item.Cur_Abbreviation === currency)
      )
    },
  },
  watch: {
    userCurrencies: {
      handler(newValue) {
        localStorage.userCurrencies = newValue
      },
      deep: true
    }
  },
  async created() {
    await this.getApiData();
    if(localStorage.userCurrencies) {
      this.userCurrencies = localStorage.userCurrencies.split(',')
    }
    for(let currency of this.userCurrencies) {
      this.addCurrencyData(currency)
    }
  },
}
</script>

<style scoped>
/*.currency{
  margin-top: 4px;
  margin-bottom: 4px;
}*/
.ant-divider {
  margin-bottom: 4px;
  margin-top: 4px;
}
.add{
  border-radius: 5px;
  margin-bottom: 0.5rem;
}
.date{
  text-align: end;
  margin-right: 2rem;
  margin-bottom: 1rem;
}
</style>
