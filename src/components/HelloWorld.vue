<template>
  <v-row justify="center">
    <v-dialog v-model="dialog" fullscreen hide-overlay transition="dialog-bottom-transition" lazy>
      <template v-slot:activator="{ on, attrs }">
        <v-btn color="primary" dark v-bind="attrs" v-on="on"> Open Dialog </v-btn>
      </template>
      <v-card>
        <trading-vue v-if="value" :data="this.$data" :overlays="overlays" :width="width" :height="height"></trading-vue>
      </v-card>
    </v-dialog>
  </v-row>
</template>

<script>
import TradingVue from "trading-vue-js";
import Overlays from "tvjs-overlays";

export default {
  name: "HelloWorld",
  components: { TradingVue },
  props: ['value'],
  data: () => ({
    dialog: false,
    height: 500,
    width: 500,
    arr: [],
    overlays: [Overlays["CCI"]],
  }),
  updated() {
    this.arr = this.value;
    console.log(this.value);
  },
};
</script>
