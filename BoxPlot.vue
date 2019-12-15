<template>
  <v-stage :config="configKonva">
    <v-layer>
      <v-rect :config="configP" />
      <v-circle :config="configExtreme(ymax)" />
      <v-circle :config="configExtreme(ymin)" />
      <v-line :config="configMedian" />
      <v-line :config="configQuartile(yp25, yq25)" />
      <v-line :config="configQuartile(yp75, yq75)" />
      <v-rect :config="configMean" />
    </v-layer>
  </v-stage>
</template>

<script>
export default {
  name: "BoxPlot",
  props: {
    width: {type: Number, default: 20},
    height: {type: Number, default: 150},
    elementSize: {type: Number, default: 5},
    mean: {type: Number, required: true},
    median: {type: Number, required: true},
    p25: {type: Number, required: true},
    p75: {type: Number, required: true},
    q25: {type: Number, required: true},
    q75: {type: Number, required: true},
    min: {type: Number, required: true},
    max: {type: Number, required: true},
    minTotal: {type: Number, required: true},
    maxTotal: {type: Number, required: true},
  },
  data: () => ({
    xOffset: 1,
  }),
  computed: {
    yOffset: function() {
      return Math.trunc(this.elementSize / 2) + 1
    },
    drawWith: function() {
      return this.width - this.xOffset * 2;
    },
    drawHeight: function() {
        return this.height - this.yOffset * 2
    },
    centerdX: function () {
      return this.xOffset + this.drawWith / 2
    },
    configKonva: function() {
      return {
        width: this.width,
        height: this.height
      };
    },
    ymax: function() {
      return this.calculateY(this.max);
    },
    ymin: function() {
      return this.calculateY(this.min);
    },
    ymedian: function() {
      return this.calculateY(this.median);
    },
    ymean: function() {
      return this.calculateY(this.mean);
    },
    yp75: function() {
      return this.calculateY(this.p75);
    },
    yp25: function() {
      return this.calculateY(this.p25);
    },
    yq75: function() {
      return this.calculateY(this.q75);
    },
    yq25: function() {
      return this.calculateY(this.q25);
    },
    pHeight: function() {
      return this.calculateHeight(this.p75 - this.p25)
    },
    configMedian: function() {
      return {
        points: [this.xOffset, this.ymedian, this.drawWith, this.ymedian],
        stroke: "black",
        strokeWidth: 2
      };
    },
    configP: function() {
      return {
        x: this.xOffset,
        y: this.yp75,
        width: this.drawWith,
        height: this.pHeight,
        stroke: "black",
        fill: "gray",
        opacity: 0.7,
        strokeWidth: 1
      };
    },
    configMean: function() {
      let size = this.elementSize * 1.5
      return {
        x: this.centerdX,
        y: this.ymean - size / 2,
        width: size,
        height: size,
        stroke: "black",
        rotation: 45,
        fill: "black",
        // opacity: 0.7,
        strokeWidth: 1
      };
    },
  },
  methods: {
    calculateY: function(value) {
      let result = this.yOffset + this.drawHeight - this.calculateHeight(value - this.minTotal)
      
      // console.log(`calcY: value: ${value}; result: ${Math.trunc(result)}`)
      return result
    },
    calculateHeight: function(value) {
      let result = 
        ((value) /
          (this.maxTotal - this.minTotal)) *
        this.drawHeight
      
      // console.log(`calcHeight: value: ${value}; result: ${Math.trunc(result)}`)
      return result;
    },
    configExtreme: function(value) {
      return {
        x: this.centerdX,
        y: value,
        radius: this.elementSize,
        stroke: "black",
        // fill: 'gray',
        // opacity: 0.7,
        strokeWidth: 1
      };
    },
    configQuartile: function(start, end) {
      return {
        points: [this.centerdX, start, this.centerdX, end,
        this.centerdX / 2, end, this.centerdX / 2 * 3, end],
        stroke: "black",
        strokeWidth: 1,
        // dash: [5, 5]
      };
    },
  }
};
</script>
