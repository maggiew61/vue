<template>
  <div class="currencyList ">
    <span class="flow-text"> {{title}}</span>

    <div class="col s12 card-tabs" >
      <ul class=" tabs tabs-fixed-width blue_tabs">
        <li class="tab mytab" v-for="tab in listindex"  :class="{active: activeIndex === tab}">
          <a :href="'#'+tab"
             :class="{active: activeIndex === tab}"
             @click="setActive(tab)">{{tab}}</a>
        </li>
      </ul>
    </div>
    <div class="clearfix"></div>
    <div class="listWrap row">
      <div class="list" v-for="(li , index) in listcontent">
        <div class="listContent">
            <div class="listTop col s12">
              <div class="title col s6">{{li.title}}</div>
              <div class="hiliter col s6 text-right" :title="li.now">
                {{Math.round(li.now * 10000)/10000}}
              </div>
            </div>
            <div class="clearfix"></div>
            <div class="listBottom col s12">
              <div class="col s6 chartBox">
                <currency-sparklines v-if="li.growth" :data="li.hist" :id="li.title+'-'+activeIndex+'-'+ index "></currency-sparklines>
              </div>
              <div v-if="li.growth" 
                  class="col s6 text-right growth"
                  :class="{incrs: li.growth >  0, decrs: li.growth <  0}">
                <span :title="li.growth" class="growth">{{Math.round(li.growth * 10000)/10000}}%</span>
                <i class="material-icons iconsVa ">{{li.icon}}</i>
              </div>
              <div v-if="!li.growth" class="col s6 text-right" title="無資料"> --- </div>
            </div>
            <div class="clearfix"></div>
        </div>
      </div>
    </div>
    <div class="d3tooltip">d3tooltip1d3tooltip1</div>
  </div>
</template>

<script>

import { mapGetters, mapActions } from 'vuex';
import currencySparklines from './currencySparklines.vue';

export default {
  name: 'currencyList',
  components: {
    currencySparklines,
  },
  data() {
    return {
      activeIndex: '',
      tooltip: '',
    };
  },
  props: {
    data: {
      type: Array,
      default: [],
    },
  },
  computed: {
    ...mapGetters(['CurrencyData']),
    title() {
      return '貨幣';
    },
    listindex() {
      const list = [];
      for (let i = 0; i < this.CurrencyData.length; i += 1) {
        for (let j = 0; j < this.CurrencyData[i].data.length; j += 1) {
          if (list.indexOf(this.CurrencyData[i].data[j].title) < 0) {
            list.push(this.CurrencyData[i].data[j].title);
          }
        }
      }
      this.activeIndex = list[0];
      return list;
    },
    listcontent() {
      return this.activeIndex ? this.CurrencyData.map((d) => {
        const AcrtiveData = d.data.find(active => active.title === this.activeIndex);
        const icon = AcrtiveData.growth > 0 ? 'arrow_drop_up' : 'arrow_drop_down';
        return {
          title: d.title,
          now: AcrtiveData.now,
          hist: AcrtiveData.hist || [],
          growth: AcrtiveData.growth,
          icon,
        };
      }) : [];
    },
  },
  methods: {
    setActive(tab) {
      this.activeIndex = tab;
    },
  },
};
</script>
<style scoped lang="scss">
  .currencyList {
    min-width: 360px;
  }
  .tabs .tab a {
    color: #91aec5;
    transition: .3s .1s;
  }
  .mytab.active {
    border-bottom: 4px solid #2c67c3;
  }
  .tabs .tab  a.active {
    color: #2c67c3;
    font-weight: 600;
  }
  .tabs .tab  a:hover {
    background: rgba(0,0,0,0.05);
  }
  .incrs{
    color : #557555;
  }
  .decrs{
    color: #f76565;
  }
  .growth{
    transform: translateX(5px);
    display: inline-block;
    vertical-align: middle;
  }
  .iconsVa{
    vertical-align: middle;
    font-size: 20px;
  }
  .growth{
    font-size: .95em;
  }
  .listWrap .list:nth-child(2n){
    border-bottom: 1px solid rgba(0,0,0,0.05);
  }

  .listWrap .list:first-child{
    border-top: 1px solid rgba(0,0,0,0.05);
  }
  .listWrap .list:last-child {
    border-top: 1px solid rgba(0,0,0,0);
  }

  .listWrap .list:nth-child(2n){
    background: rgba(0,0,0,0.05);
  }
  .listContent{
    display: block;
    padding: 5px 0 ;
  }
  .listTop{
    .title{
      color: #4e79bd;
    }
  }
  .d3tooltip{
    position: fixed;
    background: rgba(0,0,0,0.5);
    color: #fff;
    padding: 2px 4px;
    font-size: .4em;
    border-radius: 3px;
    box-shadow: 1px 1px 2px 0px rgba(0, 0, 0, 0.5)
  }
 
</style>
