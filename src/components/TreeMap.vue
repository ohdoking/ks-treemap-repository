<script>
import { eventBus } from '../main'
import * as d3 from 'd3'

export default {
  props: {
    data: { type: Object, required: true },
    width: { type: Number, required: true },
    height: { type: Number, required: true }
  },
  computed: {
    gparents() {
      return this.data.descendants().filter(d => d.depth === 1)
    },
    parents() {
      return this.data.descendants().filter(d => d.depth === 2)
    },
    children() {
      return this.data.descendants().filter(d => d.depth > 2)
    }
  },
  methods: {
    openTooltip(h, event) {
      const label = '주가 상승률 : '
      const item = {
        parent: h.parent.data.id,
        current: h.data.ticker || h.data.key,
        value: h.data.value,
        unit: h.data.unit
      }
      eventBus.$emit('openTooltip', { item, event, label })
      event.target.style.fill = 'rgba(0, 0, 0, .25)'
      event.target.style.stroke = 'rgba(255, 255, 255, .5)'
    },
    closeTooltip(event) {
      eventBus.$emit('closeTooltip', { event })
      event.target.style.fill = 'rgba(0, 0, 0, 0)'
      event.target.style.stroke = 'rgba(255, 255, 255, .3)'
    },
    setColor(item) {
      var value = Math.round(parseFloat(item.values[0].value)*100)/100

      if( value <= parseFloat(1.5) && value > parseFloat(0.25)){
        return "#35764e"
      }
      else if( value <= parseFloat(2.5) && value > parseFloat(1.5)){
        return "#2f9e4e"
      }
      else if(value > parseFloat(2.5)){
        return "#30cc5a"
      }
      else if( value < parseFloat(-0.25) && value > parseFloat(-1.5)){
        return "#8b444d"
      }
      else if( value <= parseFloat(-1.5) && value > parseFloat(-2.5)){
        return "#bf4044"
      }
      else if( value <= parseFloat(-2.5)){
        return "#f63538"
      }
      else{
        return "#414554"        
      }
    }
    ,
    setColorGrandParent() {
      return "#dfe4ed"
    }
  }
}
</script>

<template>
  <div class="visualization">
    <h2>Treemap Diagram (Hover to highlight each region)</h2>
    <svg
      :viewBox="`0, 0 ${width} ${height}`"
      :width="width"
      :height="height"
    >
      <rect
        v-for="(item, i) in parents"
        :key="`p${i}`"
        :x="item.x0"
        :y="item.y0"
        :width="(item.x1 - item.x0)"
        :height="(item.y1 - item.y0)"
        :fill="setColor(item.data)"
      />
      <rect
        v-for="(item, i) in gparents"
        :key="`gp${i}`"
        :x="item.x0"
        :y="item.y0"
        :width="(item.x1 - item.x0)"
        height="15px"
        :fill="setColorGrandParent()"
      />
      <rect
        v-for="(item, i) in children"
        :key="`c${i}`"
        :class="$style.child"
        :x="item.x0"
        :y="item.y0"
        :width="(item.x1 - item.x0)"
        :height="(item.y1 - item.y0)"
        @mouseover="openTooltip(item, $event)"
        @mouseout="closeTooltip($event)"
      />
       <!-- The visible square text element with the title and value of the child node -->
      <text 
        v-for="(item, i) in gparents"
        dy="0.4em"
        class="area"
        :key="'kt_' + i" 
        :x="item.x0 + 6" 
        :y="item.y0 + 6" 
        :width="(item.x1 - item.x0)"
        style="fill-opacity: 1;font-size: 1.3vw;font-weight: bold"
        >
        {{ item.data.key }}
      </text>
      <text 
        v-for="(item, i) in children"
        dy="2em"
        :key="'st_' + i" 
        :x="item.x0 + 6" 
        :y="item.y0 + 6" 
        :width="(item.x1 - item.x0)"
        :height="(item.y1 - item.y0)"
        style="fill-opacity: 1;"
        >
        {{ item.data.ticker }}
      </text>
      <text 
        v-for="(item, i) in children"
        dy="3.25em"
        :key="'sv_' + i" 
        :x="item.x0 + 6" 
        :y="item.y0 + 6" 
        :width="(item.x1 - item.x0)"
        :height="(item.y1 - item.y0)"
        style="fill-opacity: 1;"
        >
        {{ item.data.value }}{{ item.data.unit }}
      </text>
    </svg>
  </div>
</template>

<style module>
.child {
  fill: transparent;
  stroke: rgba(255, 255, 255, 0.3);
}
/*svg text {
  color: #000000;
  font-size: 1vw;
}*/
</style>
