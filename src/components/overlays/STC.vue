<script>
// STC renderer. (SMAs, EMAs, TEMAs...
// you know what I mean)
// TODO: make a real spline, not a bunch of lines...

// Adds all necessary stuff for you.
import Overlay from "../../mixins/overlay.js";

export default {
  name: "STC",
  mixins: [Overlay],
  methods: {
    meta_info() {
      return {
        author: "C451",
        version: "1.1.2",
        description: "SCHAFF TREND CYCLE",
        preset: {
          name: "STC",
          side: "offchart",
          settings: {
            lineWidth: 0.75,
            color: "#ff9669",
            bandColor: "#aaa",
            backColor: "#9b9ba316",
            upper: 0.75,
            lower: 0.25,
          },
        },
      };
    },

    // Here goes your code. You are provided with:
    // { All stuff is reactive }
    // $props.layout -> positions of all chart elements +
    //  some helper functions (see layout_fn.js)
    // $props.interval -> candlestick time interval
    // $props.sub -> current subset of candlestick data
    // $props.data -> your indicator's data subset.
    //  Comes "as is", should have the following format:
    //  [[<timestamp>, ... ], ... ]
    // $props.colors -> colors (see TradingVue.vue)
    // $props.cursor -> current position of crosshair
    // $props.settings -> indicator's custom settings
    //  E.g. colors, line thickness, etc. You define it.
    // $props.num -> indicator's layer number (of All
    // layers in the current grid)
    // $props.id -> indicator's id (e.g. EMA_0)
    // ~
    // Finally, let's make the canvas dirty!

    draw(ctx) {
      const layout = this.$props.layout;
      const upper = layout.$2screen(this.sett.upper || 70);
      const lower = layout.$2screen(this.sett.lower || 30);
      const data = this.$props.data;

      // RSI values

      ctx.lineWidth = this.line_width;
      ctx.strokeStyle = this.color;
      ctx.beginPath();

      for (var k = 0, n = data.length; k < n; k++) {
        let p = data[k];
        let x = layout.t2screen(p[0]);
        let y = layout.$2screen(p[1]);
        ctx.lineTo(x, y);
      }

      ctx.stroke();

      ctx.strokeStyle = this.band_color;
      ctx.setLineDash([5]); // Will be removed after draw()
      ctx.beginPath();

      // Fill the area between the bands
      ctx.fillStyle = this.sett.backColor;
      ctx.fillRect(0, upper, layout.width, lower - upper);

      // Upper band
      ctx.moveTo(0, upper);
      ctx.lineTo(layout.width, upper);

      // Lower band
      ctx.moveTo(0, lower);
      ctx.lineTo(layout.width, lower);

      ctx.stroke();
    },

    // For all data with these types overlay will be
    // added to the renderer list. And '$props.data'
    // will have the corresponding values. If you want to
    // redefine the default behviour for a prticular
    // indicator (let's say EMA),
    // just create a new overlay with the same type:
    // e.g. use_for() { return ['EMA'] }.
    use_for() {
      return ["userSTC", "userCMF", "userStochRSI", "userCCI"];
    },

    // Colors for the legend, should have the
    // same dimention as a data point (excl. timestamp)
    data_colors() {
      return [this.color];
    },
    // Will produce: "OverlayKing 10 v=2 [2,3]"
    legend(values) {
      return [
        {
          value: `${values[1]}`,
          color: this.sett.color,
        },
      ];
    },

    y_range(hi, lo) {
      return [
        Math.max(hi, this.sett.upper || 0.75), // new high
        Math.min(lo, this.sett.lower || 0.25), // new low
      ];
    },
  },
  // Define internal setting & constants here
  computed: {
    sett() {
      return this.$props.settings;
    },
    line_width() {
      return this.sett.lineWidth || 0.75;
    },
    color() {
      const n = this.$props.num % 5;
      return this.sett.color || this.COLORS[n];
    },
    data_index() {
      return this.sett.dataIndex || 1;
    },
    // Don't connect separate parts if true
    skip_nan() {
      return this.sett.skipNaN;
    },

    band_color() {
      return this.sett.bandColor || "#ddd";
    },
  },
  data() {
    console.log("STC data");
    console.log(this.color);
    console.log(this.$props.settings);
    return {
      COLORS: ["#42b28a", "#5691ce", "#612ff9", "#d50b90", "#ff2316"],
    };
  },
};
</script>
