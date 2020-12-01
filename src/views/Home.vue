<template>
  <div ref="infoBox" class="d-flex flex-column justify-space-between align-center" style="background-color: 'red'">
    <v-row>
      <v-col cols="12">
        <v-menu ref="menu" v-model="menu" :close-on-content-click="false" :return-value.sync="date" transition="scale-transition" offset-y min-width="290px">
          <template v-slot:activator="{ on, attrs }">
            <v-text-field v-model="date" label="Picker in menu" prepend-icon="mdi-calendar" readonly v-bind="attrs" v-on="on"></v-text-field>
          </template>
          <v-date-picker v-model="date" no-title scrollable>
            <v-spacer></v-spacer>
            <v-btn text color="primary" @click="menu = false"> Cancel </v-btn>
            <v-btn text color="primary" @click="save(date)"> OK </v-btn>
          </v-date-picker>
        </v-menu>
      </v-col>
      <v-spacer></v-spacer>
    </v-row>
    <trading-vue :data="chart" :width="width" :height="height" title-txt="BTCUSD" :toolbar="true" :overlays="overlays"></trading-vue>
  </div>
</template>
<script>
import TradingVue from "trading-vue-js";
import { DataCube } from "trading-vue-js";
import Overlays from 'tvjs-overlays'
 
export default {
  name: "app",
  components: { TradingVue },
  data() {
    return {
      api: "https://iss.moex.com/iss/engines/stock/markets/shares/boards/tqbr/securities/SBERP/candles.json?from=",
      height: 500,
      width: 500,
      chart: new DataCube([]),
      overlays: [Overlays['Histogram']],
      menu: false,
      date: new Date(new Date().setDate(new Date().getDate() - 1)).toISOString().substr(0, 10),
    };
  },
  mounted() {
    this.matchHeight();
    let ohlcv = [];
    this.$http.get(this.api + this.date).then((response) => {
      for (let arr of response.data.candles.data) {
        ohlcv.push(new Array(new Date(arr[6]).getTime(), arr[0], arr[2], arr[3], arr[1], arr[5]));
      }
      this.chart = new DataCube({ chart: { data: ohlcv } });
    });
  },
  methods: {
    matchHeight() { // 1
      this.height = this.$refs.infoBox.clientHeight * 1.3;
      this.width = this.$refs.infoBox.clientWidth;
      console.log(this.$refs.infoBox.clientWidth);
    },
    save(date) {
      this.$refs.menu.save(date);
      this.date = date;
      let ohlcv = [];
      this.$http.get(this.api + this.date).then((response) => {
        for (let arr of response.data.candles.data) {
          ohlcv.push(new Array(new Date(arr[6]).getTime(), arr[0], arr[2], arr[3], arr[1], arr[5]));
        }
        this.chart = new DataCube({ chart: { data: ohlcv } });
      });
    },
  },
};
</script>
