<script>
import * as d3 from 'd3'
import ToolTip from '../components/ToolTip'
import TreeMap from '../components/TreeMap'

export default {
  components: {
    ToolTip,
    TreeMap
  },
  data() {
    return {
      data: [],
      groupOrder: ['area', 'ticker'],
      selected: 'treemap',
      height: 600,
      width: 600
    }
  },
  computed: {
    hierarchy() {
      let h = d3.hierarchy(this.nestedData, v => v.values)
      
      // stock size
      h.sum(v => v.size)
      h.sort((a, b) => d3.ascending(a, b))
      // https://github.com/d3/d3-hierarchy/blob/v1.1.9/README.md#node_each
      return h.each(function(d) {
        console.log(d)
      })
    },
    treemap() {
      return this.treemapLayout(this.hierarchy)
    },
    nestedData() {
      return {
        name: 'Stock Market Map',
        values: this.nester.entries(this.data)
      }
    },
    // https://github.com/d3/d3-collection/blob/v1.0.7/README.md#nest
    nester() {
      const n = d3.nest()
      this.groupOrder.forEach(v => {
        n.key(node => node[v])
      })
      return n
    },
    treemapLayout() {
      return d3.treemap().size([this.width, this.height])
    }
  },
  // https://github.com/thegoodideaco/visualizing-hierarchies/blob/master/datasets/populations.json
  async mounted() {
    const url = await d3.json('http://127.0.0.1:5000/api/v1/getGraphInfo?type=1week')
    this.data = Object.freeze(url)
  }
}
</script>

<template>
  <div :class="$style.stock">
    <ToolTip />
    <h1>Stock Market Map</h1>
    <TreeMap
      v-bind="{ data: treemap, width, height }"
    />
  </div>
</template>

<style module>
.stock {
  max-width: 950px;
  margin: 0 auto;
  display: flex;
  flex-direction: column;
}
</style>
