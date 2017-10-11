<template>
  <div class="platform ">
    <div class="nav">
      <div class="center">
        <i class="material-icons ">trending_up</i>
        <div class="center">
          <span class="btn" @click="currency = true">currency-list</span>          
          <span class="btn" @click="currency = false">map</span>          
        </div>
      </div>
    </div>
    
    <hr>
    <div class="main">
      <div class="mainWarp"  v-if="currency">
        <div class="container" >
          <div class="row">
            <currency-list class="col s4 m4 card" :data="CurrencyData"></currency-list>
          </div>
        </div>
      </div>
      <div  v-if="!currency" >
        <map-box :data="TradingData"></map-box>
      </div>
     <div class="clearfiX"></div>
     </div>
  </div>
</template>

<script>

import { mapGetters, mapActions } from 'vuex';
import currencyList from './currencyList.vue';
import mapBox from './mapBox.vue';

export default {
  name: 'platform',
  data() {
    return {
      currency: true,
    };
  },
  created() {
    this.getCurrency_dashboard();
    this.getTrading_amt();
  },
  components: {
    currencyList,
    mapBox,
  },
  computed: {
    ...mapGetters(['showLoading', 'CurrencyData', 'TradingData']),
  },
  methods: {
    ...mapActions(['logout', 'getCurrency_dashboard', 'getTrading_amt']),
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
  .platform{
    height: 100vh;
    overflow: hidden;
  }
  hr{
    margin: 10px 0;
    background: #ccc;
  }
  .nav{
    height: 65px;
    overflow: hidden;
  }
  .main{
    height: calc(100vh - 100px);
    background: #ededed;
  }
  .mainWarp{
    height: calc(100vh - 100px);
    overflow: auto;
  }

</style>
