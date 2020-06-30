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
        <g>
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
            <title>{{ tooltipData[index - 1] }}</title>
          </polygon>
        </g>
        <g
          v-if="showGuideLines"
        >
          <polyline
            v-for="(guideLine, idx) in guideLines"
            :key="'g' + idx"
            :points="guideLine.join(' ')"
            class="tsc-guide-line"
            :style="{ stroke: guideColor }"
            @mouseover="onGuideLineMouseOver"
            @mouseout="onGuideLineMouseOut"
          ></polyline>
        </g>
        <g
          v-if="showValues"
        >
          <text
            v-for="(item, index) in textsPosition"
            :key="'t' + index"
            :x="item[0]"
            :y="item[1]"
            text-anchor="middle"
            dominant-baseline="middle"
            :fill="valueColor"
            :style="{
              transform: 'rotate(90deg)',
              transformOrigin: item[0] + 'px ' + item[1] + 'px',
              pointerEvents: 'none',
              fontSize: '0.95em',
              fontWeight: 'bold',
              stroke: valueStroke
            }"
          >{{ getFormattedValue(theData[index]) }}</text>
        </g>
        <g v-if="showGuideLines">
          <text
            v-for="gVal in guideValues"
            :key="'v' + gVal.id"
            :x="gVal.coord[0]"
            :y="gVal.coord[1] - 3"
            text-anchor="middle"
            dominant-baseline="top"
            :fill="guideColor"
            :style="{
              transform: 'rotate(' + (90 + angleDegree / 2) + 'deg)',
              transformOrigin: gVal.coord[0] + 'px ' + gVal.coord[1] + 'px',
              opacity: showGuideValue ? 0.5 : 0.1,
              fontWeight: 400,
              fontSize: '0.8em',
              cursor: 'default'
            }"
            @mouseover="onGuideLineMouseOver"
            @mouseout="onGuideLineMouseOut"
          >{{ gVal.value }}</text>
        </g>
        Sorry, your browser does not support inline SVG.
      </g>
    </svg>
    <div class="tsc-legends" v-if="showLegends">
      <div
        class="tsc-legend"
        v-for="index in itemCount"
        :key="index"
      >
        <div class="tsc-shape" :style="{ backgroundColor: defaultFills[index - 1] }"></div>
        <div class="tsc-title">{{ labelData[index - 1] + (showValues ? ' (' + getFormattedValue(theData[index - 1]) + ')' : '') }}</div>
      </div>
    </div>
  </div>
</template>

<script>
const numeral = require('numeral');
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
    chartLabelData: {
      type: Array,
      default: null
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
      showGuideValue: false,
      hoverings: -1,
      selected: -1,
      default: {
        strokeWidth: 3,
        strokeColor: 'white',
        tooltip: '${label}: ${value}',
        guideColor: '#656565',
        valueColor: '#000000',
        valueStroke: '#FFFFFF'
      }
    };
  },
  computed: {
    numberFormat () {
      return this.options && this.options.numberFormat ? this.options.numberFormat : false;
    },
    showLegends () {
      return this.options && this.options.showLegends ? true : false;
    },
    showGuideLines () {
      return this.options && this.options.showGuideLines ? true : false;
    },
    showValues () {
      return this.options && this.options.showValues ? true : false;
    },
    guideColor () {
      return this.options && this.options.guideColor ? this.options.guideColor : this.default.guideColor;
    },
    valueColor () {
      return this.options && this.options.valueColor ? this.options.valueColor : this.default.valueColor;
    },
    valueStroke () {
      return this.options && this.options.valueStroke ? this.options.valueStroke : this.default.valueStroke;
    },
    center () {
      return this.size / 2.0;
    },
    rotation () {
      return 'rotate(-90 ' + this.size / 2 + ' ' + this.size / 2 + ')';
    },
    maxValue () {
      return !this.max ? Math.max.apply(Math, this.theData) : this.max;
    },
    angleDegree () {
      return 360 / this.itemCount;
    },
    angle () {
      return this.angleDegree * (Math.PI / 180);
    },
    theData () {
      let data = [];
      for (let i = 0; i < this.chartData.length; i++) {
        if (this.chartData[i] instanceof Object) {
          data.push(this.chartData[i].value);
        } else {
          data.push(this.chartData[i]);
        }
      }
      return data;
    },
    labelData () {
      let labels = [];
      let unamedCounter = 0;
      if (this.chartLabelData && Array.isArray(this.chartLabelData)) {
        for (let i = 0; i < this.itemCount; i++) {
          if (this.chartLabelData[i]) {
            labels.push(this.chartLabelData[i]);
          } else {
            labels.push('Unnamed ' + unamedCounter++);
          }
        }
      } else {
        for (let i = 0; i < this.itemCount; i++) {
          labels.push('Unnamed ' + unamedCounter++);
        }
      }
      for (let i = 0; i < this.chartData.length; i++) {
        if (this.chartData[i] instanceof Object) {
          labels[i] = this.chartData[i].label;
        }
      }
      return labels;
    },
    values () {
      let values = [];
      for (let i = 0; i < this.theData.length; i++) {
        values[i] = this.theData[i] * this.center / this.maxValue;
      }
      return values;
    },
    textsPosition () {
      let textsPos = [];
      for (let i = 0; i < this.theData.length; i++) {
        textsPos.push([
          (this.coords[i][1][0] + this.coords[i][2][0] + this.coords[i][0][0]) / 3,
          (this.coords[i][1][1] + this.coords[i][2][1] + this.coords[i][0][1]) / 3
        ]);
      }
      return textsPos;
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
    guideLines () {
      let coords = [];

      let coord = [];
      for (let i = 0; i < this.itemCount; i++) {
        coord.push(this.getCoord(this.center, i * this.angle));
      }
      coord.push(this.getCoord(this.center, 0));
      coords.push(coord);

      let step = parseInt(this.center / 3);
      for (let i = 1; i >= 0; i--) {
        let coord = [];
        for (let j = 0; j < this.itemCount; j++) {
          coord.push(this.getCoord(step * (i + 1), j * this.angle));
        }
        coord.push(this.getCoord(step * (i + 1), 0));
        coords.push(coord);
      }
      
      return coords;
    },
    guideValues () {
      let values = [];

      let coord1 = this.getCoord(this.center, 0);
      let coord2 = this.getCoord(this.center, this.angle);
      values.push({
        id: 0,
        value: this.maxValue,
        coord: [(coord1[0] + coord2[0]) / 2 - 17, (coord1[1] + coord2[1]) / 2]
      });

      let step = parseInt(this.maxValue / 3);
      let coordStep = parseInt(this.center / 3);
      for (let i = 1, j = 1; i >= 0, j < 3; i--, j++) {
        coord1 = this.getCoord(coordStep * (i + 1), 0);
        coord2 = this.getCoord(coordStep * (i + 1), this.angle);
        values.push({
          id: j,
          value: step * (i + 1),
          coord: [(coord1[0] + coord2[0]) / 2, (coord1[1] + coord2[1]) / 2]
        });
      }

      return values;
    },
    itemCount () {
      return this.chartData.length;
    },
    defaultFills () {
      let colors = [];
      let fillsIsArray = Array.isArray(this.theOptions.fills || null);
      let fillsIsString = this.theOptions.fills ? typeof this.theOptions.fills === 'string' : false;
      for (let i = 0; i < this.itemCount; i++) {
        if (this.chartData[i] instanceof Object && this.chartData[i].fill) {
          colors.push(this.chartData[i].fill);
        } else if (fillsIsString) {
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
        if (this.chartData[i] instanceof Object && this.chartData[i].hover) {
          colors.push(this.chartData[i].hover);
        } else if (fillsIsString) {
          colors.push(this.theOptions.hoverFills);
        } else if (fillsIsArray && this.theOptions.hoverFills[i]) {
          colors.push(this.theOptions.hoverFills[i]);
        } else {
          colors.push(0.75);
        }
      }
      return colors;
    },
    tooltipData () {
      let tooltip = [];
      let tooltipStr = this.theOptions.tooltip || this.default.tooltip;
      for (let i = 0; i < this.itemCount; i++) {
        tooltip.push(
          tooltipStr
            .replace('${label}', this.labelData[i])
            .replace('${value}', this.getFormattedValue(this.theData[i]))
        );
      }
      return tooltip;
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
    onGuideLineMouseOver () {
      this.showGuideValue = true;
    },
    onGuideLineMouseOut () {
      this.showGuideValue = false;
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
    },
    getFormattedValue (value) {
      let frmValue = numeral(value);
      if (this.numberFormat) {
        return frmValue.format(this.numberFormat);
      }
      return value;
    }
  },
  mounted () {
    this.selected = this.selectedIndex;
  }
};
</script>
<style scoped>
.tsc-container {
  padding: 16px;
}
.tsc-legends {
  display: flex;
  flex-direction: row;
  flex-wrap: wrap;
  justify-content: center;
  align-items: center;
}
.tsc-legend {
  margin: 16px 6px 6px;
}
.tsc-shape {
  border: 1px solid #565656;
  display: inline-block;
  width: 20px;
  height: 16px;
  line-height: 16px;
}
.tsc-title {
  margin-left: 4px;
  line-height: 16px;
  display: inline-block;
  vertical-align: text-top;
  font-size: 1rem;
}
.tsc-guide-line {
  stroke: #656565;
  stroke-width: 1;
  stroke-dasharray: 5;
  fill: transparent;
  opacity: 0.3;
  pointer-events: visibleStroke
}
</style>
