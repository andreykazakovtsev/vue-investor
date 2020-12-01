<template>
  <v-container class="pa-0 mx-0">
    <v-navigation-drawer app permanent right width="100%" dark style="z-index: 121" value="true" :mini-variant.sync="mini" mini-variant-width="50">
      <v-list-item>
        <v-list-item-avatar>
          <v-icon>more_vert</v-icon>
        </v-list-item-avatar>
        <v-list-item-title class="title"> Настройки </v-list-item-title>
        <v-btn icon @click.stop="closeFilter">
          <v-icon>clear</v-icon>
        </v-btn>
      </v-list-item>

      <v-divider></v-divider>
      <v-list dense>
        <v-list-item>
          <v-list-item-icon>
            <v-icon>trending_up</v-icon>
          </v-list-item-icon>
          <v-list-item-content class="pa-0 ma-0">
            <v-autocomplete v-model="refItem" :items="refList" dense filled label="Акция" item-text="secname" return-object></v-autocomplete>
          </v-list-item-content>
        </v-list-item>
        <v-list-item>
          <v-list-item-icon>
            <v-icon>event</v-icon>
          </v-list-item-icon>
          <v-list-item-content class="pa-0 ma-0">
            <v-menu ref="menuBegDate" v-model="menuBegDate" :close-on-content-click="false" :return-value.sync="begDate" transition="scale-transition" offset-y min-width="290px">
              <template v-slot:activator="{ on, attrs }">
                <v-text-field v-model="begDate" label="Дата начала" readonly v-bind="attrs" v-on="on"></v-text-field>
              </template>
              <v-date-picker v-model="begDate" no-title scrollable locale="ru-Ru" :first-day-of-week="1">
                <v-btn text color="primary" @click="menuBegDate = false"> Отмена </v-btn>
                <v-btn text color="primary" @click="updatePeriod(0, begDate)"> Подтвердить </v-btn>
              </v-date-picker>
            </v-menu>
          </v-list-item-content>
        </v-list-item>
        <v-list-item>
          <v-list-item-icon>
            <v-icon>event</v-icon>
          </v-list-item-icon>
          <v-list-item-content class="pa-0 ma-0">
            <v-menu ref="menuEndDate" v-model="menuEndDate" :close-on-content-click="false" :return-value.sync="endDate" transition="scale-transition" offset-y min-width="290px">
              <template v-slot:activator="{ on, attrs }">
                <v-text-field v-model="endDate" label="Дата окончания" readonly v-bind="attrs" v-on="on"></v-text-field>
              </template>
              <v-date-picker v-model="endDate" no-title scrollable locale="ru-Ru" :first-day-of-week="1">
                <v-btn text color="primary" @click="menuEndDate = false"> Отмена </v-btn>
                <v-btn text color="primary" @click="updatePeriod(1, endDate)"> Подтвердить </v-btn>
              </v-date-picker>
            </v-menu>
          </v-list-item-content>
        </v-list-item>
      </v-list>
    </v-navigation-drawer>

    <trading-vue :data="chart" :title-txt="title" :toolbar="true" :overlays="overlays" :width="width" :height="height" ref="tradingVue"> </trading-vue>
  </v-container>
</template>
<script>
import TradingVue from "trading-vue-js";
import { DataCube } from "trading-vue-js";
import Overlays from "tvjs-overlays";

export default {
  name: "app",
  components: { TradingVue },
  data() {
    return {
      ref: "https://iss.moex.com/iss/engines/stock/markets/shares/boards/tqbr/securities.json",
      api: "https://iss.moex.com/iss/engines/stock/markets/shares/boards/tqbr/securities/",
      title: "",
      refList: [],
      refItem: null,
      height: window.innerHeight,
      width: window.innerWidth - 50,
      mini: true,
      chart: new DataCube({ chart: { type: "Candles", data: [], indexBased: true } }),
      overlays: [Overlays["Histogram"]],
      menuBegDate: false,
      menuEndDate: false,
      begDate: new Date(new Date().setDate(new Date().getDate() - 6)).toISOString().substr(0, 10),
      endDate: new Date(new Date().setDate(new Date().getDate())).toISOString().substr(0, 10),
      colors: {
        colorBack: "#fff",
        colorGrid: "#eee",
        colorText: "#333",
      },
    };
  },
  async mounted() {
    window.addEventListener("resize", this.onResize);
    this.refList = await this.initRef();
    this.refItem = this.refList[0];
    this.title = this.refItem.secname;
    let ohlcv = await this.getData();
    this.chart = new DataCube({ chart: { type: "Candles", data: ohlcv, indexBased: true } });
    this.$refs.tradingVue.resetChart();
  },
  beforeDestroy() {
    window.removeEventListener("resize", this.onResize);
  },
  methods: {
    async initRef() {
      let list = [];
      await this.$http.get(this.ref).then((response) => {
        for (let arr of response.data.securities.data) {
          let obj = new Object();
          obj.secid = arr[0];
          obj.shortname = arr[2];
          obj.secname = arr[9];
          list.push(obj);
        }
      });
      return list;
    },
    onResize() {
      this.width = window.innerWidth - 50;
      this.height = window.innerHeight;
    },
    async updatePeriod(i, date) {
      if (i == 0) {
        this.menuBegDate = false;
        this.begDate = date;
        this.$refs.menuBegDate.save(date);
      } else {
        this.menuEndDate = false;
        this.$refs.menuEndDate.save(date);
        this.endDate = date;
      }
    },
    async closeFilter() {
      let ohlcv = await this.getData();
      this.chart = new DataCube({ chart: { type: "Candles", data: ohlcv, indexBased: true } });
      this.$refs.tradingVue.resetChart();
      this.mini = !this.mini;
    },
    async getData() {
      let ohlcv = [];
      if (!this.refItem) {
        return ohlcv;
      }
      let str = this.api + this.refItem.secid + "/candles.json?from=" + this.begDate + "&til=" + this.endDate;
      await this.$http.get(str).then((response) => {
        for (let arr of response.data.candles.data) {
          let obj = new Object();
          obj.date = arr[6];
          obj.open = arr[0];
          obj.high = arr[2];
          obj.low = arr[3];
          obj.close = arr[1];
          obj.volume = arr[5];
          ohlcv.push(obj);
        }
      });
      // группировка
      const myMap = new Map();
      for (let obj of ohlcv) {
        let key = obj.date; //.substr(0, 18);
        if (!myMap.get(key)) {
          myMap.set(key, obj);
        } else {
          let item = myMap.get(key);
          item.high = Math.max(item.high, obj.high);
          item.low = Math.min(item.low, obj.low);
          item.close = obj.close;
          item.volume = item.volume + obj.volume;
          myMap.set(key, item);
        }
      }
      let result = [];
      for (var [key, value] of myMap.entries()) {
        result.push([new Date(key).getTime() - new Date().getTimezoneOffset() * 60 * 1000, value.open, value.high, value.low, value.close, value.volume]);
      }
      return result;
    },
  },
  watch: {
    refItem(val) {
      this.title = val.secname;
      this.refItem = val;
    }
  }
};
</script>
<style scoped>
.trading-vue-toolbar {
  z-index: 165;
  position: absolute;
}
</style>
