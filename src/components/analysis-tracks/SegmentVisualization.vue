<template>
  <div class="segment-container">
    <div
      v-for="{ t, d } of points"
      :key="t"
      :style="{ left: `${t}%`, height: `100%`, width: `${d}%` }"
      class="event"
    />
  </div>
</template>

<script lang='ts'>
import {
  defineComponent,
  PropType,
  computed
} from '@vue/composition-api'

import { HDF5 } from './HDF5Segment'

export default defineComponent({
  props: {
    hdf5: {
      type: Object as PropType<HDF5>,
      required: true
    },
    start: {
      type: Number,
      required: false
    },
    stop: {
      type: Number,
      required: false
    }
  },
  setup (props) {
    const duration = computed(() => props.hdf5.audio_metadata.duration)

    const start = computed(() => props.start || 0)
    const stop = computed(() => props.stop || duration.value * 1000)
    const durations = computed(() => props.hdf5.data_object.duration.numpyArray)

    const points = computed(() => props.hdf5.data_object.time.numpyArray
      .map((x, idx) => {
        const t = x * 1000
        const d = durations.value[idx] * 1000

        if (t >= start.value && t <= stop.value) {
          const dur = (t + d >= stop.value) ? stop.value - t : d
          return {
            t: t * 100 / (stop.value - start.value),
            d: dur * 100 / (stop.value - start.value)
          }
        }
        if (t + d >= start.value && t + d <= stop.value) {
          return {
            t: 0,
            d: (t + d - start.value) * 100 / (stop.value - start.value)
          }
        }
        if (t < start.value && t + d > stop.value) {
          return {
            t: 0,
            d: 100
          }
        }
      })
    )

    return {
      points
    }
  }
})
</script>

<style lang="less" scoped>
.event-container {
  position: relative;
  height: 100%;
}
.event {
  background-color: black;
  position: absolute;
}

</style>
