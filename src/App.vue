
// eslint-disable-next-line vue/script-setup-uses-vars
<template>
  <div v-if="isLoaded">
    <trading-vue
      :data="chart"
      :overlays="overlays"
      :width="width"
      :height="height"
      :color-back="colors.colorBack"
      :color-grid="colors.colorGrid"
      :color-text="colors.colorText"
      :timezone="2"
      :titleTxt="titleTxtValue"
      :toolbar="true"
      :legendButtons="legendButtons"
      :extensions="ext"
    />
  </div>
  <div v-else>Loading...</div>
</template>

<script>
import TradingVue from "./TradingVue.vue";
// import chartData from "../data/data.json";
import DataCube from "../src/helpers/datacube.js";
import axios from "axios";
import Overlays from "tvjs-overlays";
import XP from "tvjs-xp";
import STC from "../src/components/overlays/STC.vue";

export default {
  name: "App",
  components: {
    TradingVue,
  },

  props: {
    titleTxt: {
      type: Boolean,
      default: true,
    },
  },
  data() {
    return {
      api: "https://get.coiningit.parsec.link/get/view",
      //   chartData: null,
      isLoaded: false,
      chart: {},
      indicatorData: {},
      overlays: [...Object.values(Overlays), STC],
      // overlays: [STC],
      titleTxtValue: "",
      ext: Object.values(XP),
      legendButtons: ["display", "up", "down", "add", "remove", "settings"],
      width: window.innerWidth,
      height: window.innerHeight,
      colors: {
        colorBack: "#fff",
        colorGrid: "#eee",
        colorText: "#333",
      },
    };
  },
  mounted() {
    window.addEventListener("resize", this.onResize);
    this.chartData = this.getList();
    window.dc = this.chart;
  },
  beforeDestroy() {
    window.removeEventListener("resize", this.onResize);
  },
  methods: {
    onResize() {
      this.width = window.innerWidth;
      this.height = window.innerHeight;
    },
    getList: function () {
      // get query params from url
      var urlParams = new URLSearchParams(window.location.search);
      var currencyPair = urlParams.get("currencyPair") || "BTCZAR";
      var candleInterval = urlParams.get("candleInterval") || "5m";
      var numberOfSets = urlParams.get("numberOfSets") || 2;

      let data = {
        currencyPair: currencyPair,
        candleInterval: candleInterval,
        numberOfSets: numberOfSets,
      };

      this.$emit("update-currency", currencyPair.toUpperCase());
      this.titleTxtValue = currencyPair.toUpperCase();

      let varToken =
        "$2a$08$0mnLSLW.CY8Hd1kV.L/couSfDAZQCRULAv8ozFhlrlSinATbsk8w6";

      var self = this;
      axios
        .post(self.api, data, {
          headers: {
            "Content-Type": "application/json",
            Authorization: "Bearer " + varToken,
          },
        })
        .then((response) => {
          self.chartData = response.data;
          self.isLoaded = true;
          self.chart = new DataCube(this.chartData);
          self.indicatorData = this.chartData;
        })
        .catch(function (error) {
          console.log(error);
        });
    },
    setCube: function () {
      var self = this;
      if (!this.isLoaded) {
        self.isLoaded = true;
      }
    },
  },
};
</script>

<style>
html,
body {
  margin: 0;
  padding: 0;
  overflow: hidden;
}
</style>