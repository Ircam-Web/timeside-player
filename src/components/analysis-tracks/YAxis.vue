<template>
  <FluidSVG
    class="y-axis-container"
    @resized="svgSize = $event"
  >
    <!-- y-axis will be filled by d3 -->
    <g ref="group" />
  </FluidSVG>
</template>

<script lang="ts">
import {
  defineComponent,
  computed,
  onMounted,
  watchEffect,
  Ref,
  ref
} from '@vue/composition-api'

import { scaleLinear, ScaleLinear } from 'd3-scale'
import { select } from 'd3-selection'
import { axisLeft } from 'd3-axis'

import FluidSVG from '@/components/utils/FluidSVG.vue'

import { assertIsDefined } from '@/utils/type-assert'

// See https://github.com/DefinitelyTyped/DefinitelyTyped/blob/master/types/d3-axis/index.d.ts
type d3TickFormat = (domainValue: number | { valueOf(): number }, index: number) => string

export default defineComponent({
  props: {
    min_data: {
      type: Number,
      required: true
    },
    max_data: {
      type: Number,
      required: true
    }
  },
  components: {
    FluidSVG
  },
  setup (props) {
    const svgSize: Ref<ClientRect | undefined> = ref()
    const group = ref<SVGSVGElement>()

    const yScale = computed(() => {
      const height = svgSize.value ? svgSize.value.height : 0
      // divide to convert ms to seconds
      const minData = props.min_data
      const maxData = props.max_data

      // Use scaleLinear because scaleTime() is for dates
      return scaleLinear<number>()
        .domain([ minData, maxData ])
        .range([ height, 0 ])
    }) as Readonly<Ref<ScaleLinear<number, number>>>

    // Axis
    onMounted(() => watchEffect(() => {
      assertIsDefined(group.value)
      // Select element
      const d3axis = select(group.value)

      // Define axis
      const yAxis = axisLeft(yScale.value)

      // Render axis
      d3axis.attr('transform', 'translate(50, 0)').call(yAxis)
    }))

    return {
      group,
      svgSize
    }
  }
})
</script>

<style lang="less" scoped>
.y-axis-container {
  height: 100%;
  width: 100%;
  margin-left: -50px;
}
</style>
