<template>
  <div class="tsc-container">
    <svg
      preserveAspectRatio="xMidYMid meet"
      :height="size"
      :width="size"
    >
      <g
        :transform="rotation"
        :width="size"
        :height="size"
      >
        <polygon
          v-for="index in itemCount"
          :key="index"
          :id="'item' + index"
          :class="'p' + (index - 1)"
          :points="coordString(index - 1)"
          :style="pStyle(index - 1)"
          @mouseover="hoverings = index - 1; onMouseOver(index - 1)"
          @mouseout="hoverings = -1; onMouseOut(index - 1)"
          @click="onItemClick(index - 1)"
        >
          <title>Value: {{ chartData[index - 1] }}</title>
        </polygon>
        Sorry, your browser does not support inline SVG.
      </g>
    </svg>
  </div>
</template>

<script>
export default {
  name: 'triangled-spider-chart',
  props: {
    size: {
      type: Number,
      default: 300
    },
    max: {
      type: Number,
      default: null
    },
    chartData: {
      type: Array,
      required: true
    },
    options: {
      type: Object,
      default: null
    },
    selectedIndex: {
      type: Number,
      default: -1
    }
  },
  data () {
    return {
      hoverings: -1,
      selected: -1,
      default: {
        strokeWidth: 3,
        strokeColor: 'white'
      }
    };
  },
  computed: {
    center () {
      return this.size / 2.0;
    },
    rotation () {
      return 'rotate(-90 ' + this.size / 2 + ' ' + this.size / 2 + ')';
    },
    maxValue () {
      return !this.max ? Math.max.apply(Math, this.chartData) : this.max;
    },
    angle () {
      return (360 / this.itemCount) * (Math.PI / 180);
    },
    values () {
      let values = [];
      for (let i = 0; i < this.chartData.length; i++) {
        values[i] = this.chartData[i] * this.center / this.maxValue;
      }
      return values;
    },
    theOptions () {
      return !this.options ? {} : this.options;
    },
    coords () {
      let coords = [];
      for (let i = 0; i < this.itemCount; i++) {
        let coord = [];
        coord.push([0 + this.center, 0 + this.center]);
        coord.push(this.getCoord(this.values[i], i * this.angle));
        coord.push(this.getCoord(this.values[i], (i + 1) * this.angle));
        coords[i] = coord;
      }
      return coords;
    },
    itemCount () {
      return this.chartData.length;
    },
    defaultFills () {
      let colors = [];
      let fillsIsArray = Array.isArray(this.theOptions.fills || null);
      let fillsIsString = this.theOptions.fills ? typeof this.theOptions.fills === 'string' : false;
      for (let i = 0; i < this.itemCount; i++) {
        if (fillsIsString) {
          colors.push(this.theOptions.fills);
        } else if (fillsIsArray && this.theOptions.fills[i]) {
          colors.push(this.theOptions.fills[i]);
        } else {
          colors.push(this.getRandomColor());
        }
      }
      return colors;
    },
    defaultHovers () {
      let colors = [];
      let fillsIsArray = Array.isArray(this.theOptions.hoverFills || null);
      let fillsIsString = this.theOptions.hoverFills ? typeof this.theOptions.hoverFills === 'string' : false;
      for (let i = 0; i < this.itemCount; i++) {
        if (fillsIsString) {
          colors.push(this.theOptions.hoverFills);
        } else if (fillsIsArray && this.theOptions.hoverFills[i]) {
          colors.push(this.theOptions.hoverFills[i]);
        } else {
          colors.push(0.75);
        }
      }
      return colors;
    }
  },
  methods: {
    getCoord (len, angle) {
      return [this.center + len * Math.cos(angle), this.center + len * Math.sin(angle)];
    },
    pStyle (index) {
      let fill = this.hoverings === index || this.selected === index
        ? (typeof this.defaultHovers[index] === 'number' ? this.defaultFills[index] : this.defaultHovers[index])
        : this.defaultFills[index];
      let strokeColor = this.theOptions.strokeColor || this.default.strokeColor;
      let strokeWidth = this.theOptions.strokeWidth || this.default.strokeWidth;
      let alpha = this.hoverings === index || this.selected === index
        ? (typeof this.defaultHovers[index] === 'number' ? this.defaultHovers[index] : 1)
        : 1;
      return 'fill:' + fill + ';stroke:' + strokeColor + ';stroke-width:' + strokeWidth + ';opacity:' + alpha;
    },
    onMouseOver (index) {
      this.$emit('onItemMouseOver', index);
    },
    onMouseOut (index) {
      this.$emit('onItemMouseOut', index);
    },
    onItemClick (index) {
      if (this.selected === index) {
        this.setSelected(-1);
      } else {
        this.setSelected(index);
      }
      this.$emit('onItemClick', index);
    },
    setSelected (index) {
      this.selected = index;
    },
    coordString (index) {
      if (this.coords[index]) {
        return this.coords[index].join(' ');
      }
      return '';
    },
    getRandomColor () {
      return '#' + (0x1000000 + (Math.random()) * 0xffffff).toString(16).substr(1,6);
    }
  },
  mounted () {
    this.selected = this.selectedIndex;
  }
};
</script>