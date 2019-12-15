<template>
  <v-stage :config="configKonva()">
    <v-layer>
      <v-arc v-for="(e, i) in Array(regions)" :key="i" :config="configArc(i)"></v-arc>
      <v-arc v-for="(e, i) in Array(regions + 1)" :key="i+10" :config="configArcStep(i)"></v-arc>
      <v-text v-if="showStepText" v-for="(e, i) in Array(regions + 1)" :key="i+100" :config="configStepText(i)"></v-text>
      <v-shape v-if="false" :config="configShape()"></v-shape>
      <v-arrow :config="configArrow()"></v-arrow>
      <v-circle :config="configArrowCircle()"></v-circle>
      <v-text :config="configValueText()"></v-text>
    </v-layer>
  </v-stage>
</template>

<script>
export default {
  name: "Gauge",
  props: {
    width: {type: Number, default: 200},
    height: {type: Number, default: 150},
    xOffset: {type: Number, default: 10},
    unit: {type: String, default: "%"},
    innerRadius: {type: Number, default: 40},
    outerRadius: {type: Number, default: 50},
    stepTextOffset: {type: Number, default: 60},
    showStepText: {type: Boolean, default: true},
    arcAngle: {type: Number, default: 200},
    value: {type: Number, default: 1.25},
    valueGreen: {type: Number, default: 1.2},
    colorRange: {type: Number, default: 0.1},
    minValue: {type: Number, default: 0.5},
    maxValue: {type: Number, default: 1.5},
    limitType: {type: String, default: 'upper'}
  },
  computed: {
    getMiddlePoint: function() {
      return { x: (this.width ) / 2 + this.xOffset, y: this.height / 2 };
    },
    getRange: function() {
      return Math.abs(this.maxValue - this.minValue);
    },
    regions: function() {
      if (this.limitType === "both") return 5;
      else return 3;
    },
    arrowLength: function() {
      return this.innerRadius;
    },
    stepTextLength: function() {
      return this.outerRadius + 15;
    },
    arrowAngel: function() {
      return this.getArrowAngelPoint(this.value, this.arrowLength);
    }
  },
  methods: {
    getValuePercOfRange: function(value) {
      let calcValue = value;

      if (calcValue < this.minValue) calcValue = this.minValue;
      if (calcValue > this.maxValue) calcValue = this.maxValue;

      calcValue += (this.minValue > 0 ? -1 : 1) * Math.abs(this.minValue);

      return calcValue / this.getRange;
    },
    getArrowAngelPoint: function(value, length) {
      let angle0 = this.arcAngle - (this.arcAngle - 180) / 2;

      let angle = this.arcAngle * this.getValuePercOfRange(value);

      if (angle > angle0) {
        angle = 360 - (angle - angle0);
      } else {
        angle = angle0 - angle;
      }

      let y = Math.sin((angle / 180) * Math.PI) * length;
      let x = Math.cos((angle / 180) * Math.PI) * length;

      let point = { x: x, y: -y };

      return point;
    },
    generateRange: function(elements) {
      let range = [];
      for (let i = 0; i < elements; i++) {
        range.push(i);
      }
      // console.log(range)
      return range;
    },
    getArcAngle: function(arc) {
      let range = this.getRange;
      let arcRange = 0;

      if (this.limitType === "both") {
        if (arc === 0) {
          arcRange = Math.abs(
            this.minValue - (this.valueGreen - this.colorRange * 2)
          );
        } else if (arc === 4) {
          arcRange =
            range -
            Math.abs(this.minValue - (this.valueGreen - this.colorRange * 2)) -
            this.colorRange * 4;
        } else if (arc === 1 || arc === 3) {
          arcRange = this.colorRange;
        } else if (arc === 2) {
          arcRange = this.colorRange * 2;
        }
      } else if (this.limitType === "lower") {
        if (arc === 0) {
          arcRange = Math.abs(
            this.minValue - (this.valueGreen - this.colorRange)
          );
        } else if (arc === 1) {
          arcRange = this.colorRange;
        } else if (arc === 2) {
          arcRange =
            range -
            Math.abs(this.minValue - (this.valueGreen - this.colorRange)) -
            this.colorRange;
        }
      } else if (this.limitType === "upper") {
        if (arc === 0) {
          arcRange = Math.abs(this.minValue - this.valueGreen);
        } else if (arc === 1) {
          arcRange = this.colorRange;
        } else if (arc === 2) {
          arcRange =
            range - Math.abs(this.minValue - this.valueGreen) - this.colorRange;
        }
      }

      let result = (arcRange / range) * this.arcAngle;

      return result;
    },
    configKonva: function() {
      return {
        width: this.width,
        height: this.height
      };
    },
    configValueText: function() {
      return {
        x: 0 + this.xOffset,
        y: this.getMiddlePoint.y + 10,
        fontSize: 14,
        width: this.width,
        align: "center",
        text: this.value + " " + this.unit
        // offsetX: 30
      };
    },
    configArrowCircle: function() {
      return {
        x: this.getMiddlePoint.x,
        y: this.getMiddlePoint.y,
        radius: 5,
        fill: "black",
        stroke: "black",
        strokeWidth: 0,
        shadowColor: "black",
        shadowBlur: 0,
        shadowOffset: { x: 2, y: 2 },
        shadowOpacity: 0.5
      };
    },
    configArrow: function() {
      return {
        x: this.getMiddlePoint.x,
        y: this.getMiddlePoint.y,
        points: [0, 0, this.arrowAngel.x, this.arrowAngel.y],
        pointerLength: 20,
        pointerWidth: 10,
        fill: "black",
        stroke: "black",
        strokeWidth: 3,
        shadowColor: "black",
        shadowBlur: 0,
        shadowOffset: { x: 2, y: 2 },
        shadowOpacity: 0.5
      };
    },
    getColor: function(index) {
      if (this.limitType === "both") {
        if (index === 0 || index === 4) return "red";
        if (index === 1 || index === 3) return "yellow";
        if (index === 2) return "green";
      } else if (this.limitType === "lower") {
        if (index === 0) return "red";
        if (index === 1) return "yellow";
        if (index === 2) return "green";
      } else if (this.limitType === "upper") {
        if (index === 2) return "red";
        if (index === 1) return "yellow";
        if (index === 0) return "green";
      }
    },
    configArc: function(index) {
      return {
        x: this.getMiddlePoint.x,
        y: this.getMiddlePoint.y,
        innerRadius: this.innerRadius,
        outerRadius: this.outerRadius,
        setRotationDeg:
          180 +
          (180 - this.arcAngle) / 2 +
          (index > 0
            ? this.generateRange(index).reduce(
                (x, y) => this.getArcAngle(y) + x,
                0
              )
            : 0),
        angle: this.getArcAngle(index), // this.arcAngle / 5,
        fill: this.getColor(index),

        stroke: "black",
        strokeWidth: 1,
        shadowColor: "black",
        shadowBlur: 0,
        shadowOffset: { x: 2, y: 2 },
        shadowOpacity: 0.5,
        opacity: 0.9
      };
    },
    configArcStep: function(index) {
      return {
        x: this.getMiddlePoint.x,
        y: this.getMiddlePoint.y,
        innerRadius: this.outerRadius,
        outerRadius: this.outerRadius + 5,
        setRotationDeg:
          180 +
          (180 - this.arcAngle) / 2 +
          (index > 0
            ? this.generateRange(index).reduce(
                (x, y) => this.getArcAngle(y) + x,
                0
              )
            : 0),
        angle: 1,
        fill: "black",
        // stroke: "black",
        strokeWidth: 0
      };
    },
    configStepText: function(index) {
      let value = 0;

      if (this.limitType === "both") {
        if (index === 0) value = this.minValue;
        else if (index === 1) value = this.valueGreen - this.colorRange * 2;
        else if (index === 2) value = this.valueGreen - this.colorRange;
        else if (index === 3) value = this.valueGreen + this.colorRange;
        else if (index === 4) value = this.valueGreen + this.colorRange * 2;
        else if (index === 5) value = this.maxValue;
      } else if (this.limitType === "lower") {
        if (index === 0) value = this.minValue;
        else if (index === 1) value = this.valueGreen - this.colorRange;
        else if (index === 2) value = this.valueGreen;
        else if (index === 3) value = this.maxValue;
      } else if (this.limitType === "upper") {
        if (index === 0) value = this.minValue;
        else if (index === 1) value = this.valueGreen;
        else if (index === 2) value = this.valueGreen + this.colorRange;
        else if (index === 3) value = this.maxValue;
      }

      let point = this.getArrowAngelPoint(value, this.stepTextLength);
      // let width = 60;

      // console.log(point)

      let xOffset = this.getValuePercOfRange(value) <= 0.5 ? this.stepTextOffset : 0;

      return {
        x: this.getMiddlePoint.x + point.x - xOffset,
        y: this.getMiddlePoint.y + point.y,
        fontSize: 14,
        width: this.stepTextOffset,
        align: this.getValuePercOfRange(value) <= 0.5 ? "right" : "left",
        // text: value + " " + this.unit
        text: value
        // offsetX: 30
      };
    }
  }
};
</script>
