<template>
  <div class="maintrack-and-axis-container">
    <div class="maintrack-container">
      <WaveformContainer
        class="maintrack"
        :item-id="itemId"
      />
      <TrackPluginsContainer class="track-plugins-container">
        <Region v-model="innerSelection" />
        <InteractivePlayCursor />
      </TrackPluginsContainer>
    </div>
    <Axis
      :first-time="0"
      :last-time="lastTime"
    />
  </div>
</template>

<script lang="ts">
import {
  defineComponent,
  ref,
  watch,
  PropType,
  computed
} from '@vue/composition-api'

import { useStore } from '@/store/index'
import { Region as RegionType } from '@/types/region'

import TrackPluginsContainer from '@/components/track-elements/TrackPluginsContainer.vue'
import InteractivePlayCursor from '@/components/track-elements/InteractivePlayCursor.vue'
import Region from '@/components/track-elements/Region.vue'
import WaveformContainer from '@/components/track-elements/WaveformContainer.vue'
import Axis from '@/components/track-elements/Axis.vue'

export default defineComponent({
  props: {
    itemId: {
      type: String,
      required: true
    },
    selection: {
      type: Object as PropType<RegionType>,
      required: false
    }
  },
  components: {
    WaveformContainer,
    TrackPluginsContainer,
    InteractivePlayCursor,
    Region,
    Axis
  },
  setup (props, { emit }) {
    const store = useStore()
    const selection = ref<RegionType>()

    // two-way data binding
    watch([ () => props.selection ], () => {
      selection.value = props.selection
    }, { immediate: true })
    watch([ selection ], () => {
      emit('selection', selection.value)
    }, { immediate: true })

    return {
      // Use a different name to avoid namespace conflicts with `selection` props
      innerSelection: selection,
      lastTime: computed(() => store.state.audio.duration)
    }
  }
})
</script>

<style lang="less" scoped>

.maintrack-container {
  position: relative;
}

.maintrack {
  height: 50px;
}
</style>
