<template>
  <div ref="knob" :class="[className, 'knob']" :style="{'width': width + 'px', 'height': height + 'px'}">
    <canvas ref="knobCanvas" class="knobCanvas"
                            :width="width"
                            :height="height"></canvas>
    <input ref="knobInput" type="text" 
                            :style="{'top': inputTop, 'left': inputLeft, 'width': inputWidth, 'font': inputFontSize, 'color': fgColor}"
                            v-model="knobValue"
                            class="knobInput" 
                            v-if="displayInput"
                            :disabled="readOnly"
                            @change="changeInput">
  </div>
</template>
<script>
export default {
  name: 'qhKnob',
  props: {
    value: {
      type: Number,
      default: 10
    },
    min: {
      type: Number,
      default: 0
    },
    max: {
      type: Number,
      default: 255
    },
    step: {
      type: Number,
      default: 1
    },
    cursor: {
      type: Boolean,
      default: false
    },
    thickness: {
      type: Number,
      default: 0.35
    },
    lineCap: {
      type: String,
      default: 'butt'
    },
    readOnly: {
      type: Boolean,
      default: false
    },
    displayInput: {
      type: Boolean,
      default: true
    },
    width: {
      type: Number,
      default: 100
    },
    height: {
      type: Number,
      default: 100
    },
    bgColor: {
      type: String,
      default: '#EEEEEE'
    },
    fgColor: {
      type: String,
      default: '#87CEEB'
    },
    angleOffset: {
      type: Number,
      default: -125
    },
    angleArc: {
      type: Number,
      default: 250
    },
    className: {
      type: String,
      default: ''
    }
  },
  data () {
    return {
      knobValue: this.value,
      animating: false,
      renderedknobValue: 0
    }
  },
  watch: {
    value () {
      this.draw(this.value)
      this.knobValue = this.value
    }
  },
  computed: {
    inputTop () {
      return (this.width / 2) - (this.width / 7) + 'px'
    },
    inputLeft () {
      return this.getLineWidth() + 'px'
    },
    inputWidth () {
      return this.width - (this.getLineWidth() * 2) + 'px'
    },
    inputFontSize () {
      let fontScale = Math.max(String(Math.abs(this.max)).length, String(Math.abs(this.min)).length, 2) + 2
      return 'bold ' + ((this.width / fontScale) >> 0) + 'px Arial'
    }
  },
  mounted () {
    this.$nextTick(function () {
      this.initData()
    })
  },
  methods: {
    initData () {
      const canvas = this.$refs.knobCanvas
      this.context2d = canvas.getContext('2d')
      this.renderedknobValue = this.knobValue
      this.draw()
      if (!this.readOnly) {
        this.handleChange()
      }
    },
    changeInput () {
      this.setknobValue(this.knobValue)
      this.$emit('changeKnob', this.knobValue)
    },
    getOffset () {
      const knob = this.$refs.knob
      return {
        x: knob.getBoundingClientRect().left,
        y: knob.getBoundingClientRect().top
      }
    },
    refreshCanvas () {
      if (this.renderedknobValue === this.knobValue) {
        this.animating = false
      } else {
        let requestAnimationFrame = window.requestAnimationFrame || window.mozRequestAnimationFrame ||
        window.webkitRequestAnimationFrame || window.msRequestAnimationFrame
        this.animating = true
        this.renderedknobValue = this.knobValue
        this.context2d.clearRect(0, 0, this.width, this.height)
        this.draw()
        requestAnimationFrame.call(window, this.refreshCanvas)
      }
    },
    setknobValue (knobValue, event) {
      knobValue = Math.min(this.max, Math.max(this.min, knobValue))
      this.knobValue = knobValue
      if (!this.animating) {
        this.refreshCanvas()
      }
      if (event === true && this.$refs.knob.onchange) {
        this.$refs.knob.onchange()
      }
    },
    handleChange () {
      const documentEvents = window.document
      const vm = this
      const canvas = this.$refs.knobCanvas
      canvas.addEventListener('mousedown', function (e) {
        e.preventDefault()

        let offset = vm.getOffset()
        function mouseMove (e) {
          vm.setknobValue(vm.xyToknobValue(e.clientX, e.clientY, offset), true)
        }

        function mouseUp (e) {
          documentEvents.removeEventListener('mousemove', mouseMove)
          documentEvents.removeEventListener('mouseup', mouseUp)
          vm.$emit('changeKnob', vm.knobValue)
        }

        documentEvents.addEventListener('mousemove', mouseMove)
        documentEvents.addEventListener('mouseup', mouseUp)

        mouseMove(e)
      })
      canvas.addEventListener('touchstart', function (e) {
        e.preventDefault()

        let touchIndex = e.touches.length - 1
        let offset = vm.getOffset()

        function touchMove (e) {
          vm.setknobValue(
            vm.xyToknobValue(e.touches[touchIndex].clientX, e.touches[touchIndex].clientY, offset), true
          )
        }

        function touchEnd () {
          documentEvents.removeEventListener('touchmove', touchMove)
          documentEvents.removeEventListener('touchend', touchEnd)
          vm.$emit('changeKnob', vm.knobValue)
        }

        documentEvents.addEventListener('touchmove', touchMove)
        documentEvents.addEventListener('touchend', touchEnd)

        touchMove(e)
      })
    },
    xyToknobValue (x, y, offset) {
      let PI2 = 2 * Math.PI
      let w2 = this.width / 2
      let angleArc = this.angleArc * Math.PI / 180
      let angleOffset = this.angleOffset * Math.PI / 180
      let angle = Math.atan2(x - (offset.x + w2), -(y - offset.y - w2)) - angleOffset

      if (angleArc !== PI2 && (angle < 0) && (angle > -0.5)) {
        angle = 0
      } else if (angle < 0) {
        angle += PI2
      }

      let result = ~~(0.5 + (angle * (this.max - this.min) / angleArc)) + this.min
      return Math.max(Math.min(result, this.max), this.min)
    },
    getLineWidth () {
      let xy = this.width / 2
      return xy * this.thickness
    },
    draw (newValue) {
      let currentValue = newValue !== undefined ? newValue : this.knobValue
      let angleOffset = this.angleOffset * Math.PI / 180
      let angleArc = this.angleArc * Math.PI / 180

      let angle = (currentValue - this.min) * angleArc / (this.max - this.min)

      let xy = this.width / 2
      let lineWidth = xy * this.thickness
      let radius = xy - lineWidth / 2

      let startAngle = 1.5 * Math.PI + angleOffset
      let endAngle = 1.5 * Math.PI + angleOffset + angleArc

      let startAt = startAngle
      let endAt = startAt + angle

      if (this.cursor) {
        let cursorExt = (this.cursor / 100) || 1
        startAt = endAt - cursorExt
        endAt = endAt + cursorExt
      }

      this.context2d.lineWidth = lineWidth
      this.context2d.lineCap = this.lineCap

      this.context2d.beginPath()
      this.context2d.strokeStyle = this.bgColor
      this.context2d.arc(xy, xy, radius, endAngle, startAngle, true)
      this.context2d.stroke()

      this.context2d.beginPath()
      this.context2d.strokeStyle = this.fgColor
      this.context2d.arc(xy, xy, radius, startAt, endAt, false)
      this.context2d.stroke()
    }
  }
}
</script>
<style scoped>
.knob {
  position: relative;
  display: inline-block;
}
.knobInput {
  position: absolute;
  vertical-align: middle;
  border: 0;
  background: none;
  text-align: center;
  -webkit-appearance: none;
}
.knobCanvas {
  position: absolute;
  top: 0;
  left: 0;
}
</style>
