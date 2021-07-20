<template>
  <div
    ref="el"
    class="player"
  >
    <div class="first-line">
      <span class="title">{{ item.title }}</span>
      <Audio :audio-srcs="audioSrcs" />
    </div>
    <div v-if="audioReady">
      <!--
        These components need audio to be loaded because
        they need audio's duration to compute position
      -->
      <MainTrack
        class="main-track"
        :item-id="item.uuid"
        :selection="selection"
        @selection="onSelection"
      />
      <div>
        <Timer class="timer" />
      </div>
      <div class="menu">
        <div class="line">
          <div class="actions">
            <button
              :class="{ 'active': isAnalysisTrackFormOpen }"
              @click="isAnalysisTrackFormOpen = !isAnalysisTrackFormOpen; isAnnotationTrackFormOpen = false"
            >
              <span class="plus-picto">+</span>
              Analysis
            </button>
            <button
              :class="{ 'active': isAnnotationTrackFormOpen }"
              @click="isAnnotationTrackFormOpen = !isAnnotationTrackFormOpen; isAnalysisTrackFormOpen = false"
            >
              <span class="plus-picto">+</span>
              Annotation
            </button>
          </div>
        </div>
      </div>
      <div class="analysis-tab">
        <div v-if="isAnalysisTrackFormOpen" class="create-form-container">
          <CreateAnalysisTrack
            :item-id="item.uuid"
            class="create-form"
            @new-analysis-track="analysisTracks.add($event); isAnalysisTrackFormOpen = false"
            @close="isAnalysisTrackFormOpen = false"
          />
        </div>
        <div v-if="isAnnotationTrackFormOpen" class="create-form-container">
          <CreateAnnotationTrack
            :item-id="item.uuid"
            class="create-form"
            @new-annotation-track="annotationTracks.add($event); isAnnotationTrackFormOpen = false"
            @close="isAnnotationTrackFormOpen = false"
          />
        </div>
        <AnnotationTracks
          :item-id="item.uuid"
          :selection="selection"
          :annotation-tracks="annotationTracks"
        />
        <AnalysisTracks
          :item-id="item.uuid"
          :selection="selection"
          :analysis-tracks="analysisTracks"
        />
      </div>
    </div>
    <div v-else>
      Loading audio file...
    </div>
  </div>
</template>

<script lang="ts">
import {
  defineComponent,
  ref,
  PropType,
  computed
} from '@vue/composition-api'

import { useStore } from '@/store/index'
import { Item } from '@/utils/api'
import { assertIsDefined } from '@/utils/type-assert'
import { providePlayerRect } from '@/utils/use-player-rect'
import useBoundingClientRect from '@/utils/use-bounding-client-rect'
import { Region as RegionType } from '@/types/region'

import bindSelectionUrl from '@/utils/bind-selection-url'

import Controls from '@/components/Controls.vue'
import Timer from '@/components/Timer.vue'
import Audio from '@/components/Audio.vue'
import MainTrack from '@/components/tracks/MainTrack.vue'
import AnalysisTracks from '@/components/tracks/AnalysisTracks.vue'
import AnnotationTracks from '@/components/annotation/AnnotationTracks.vue'
import CreateAnalysisTrack from '@/components/analysis-tracks/CreateAnalysisTrack.vue'
import CreateAnnotationTrack from '@/components/annotation/CreateAnnotationTrack.vue'

import analysisTrackStore from '@/utils/analysis-track-store'
import annotationTrackStore from '@/utils/annotation-track-store'

enum Tab {
  Info,
  Analysis
}

export default defineComponent({
  name: 'Player',
  props: {
    item: {
      type: Object as PropType<Item>,
      required: true
    }
  },
  components: {
    Audio,
    Timer,
    MainTrack,
    AnalysisTracks,
    Controls,
    AnnotationTracks,
    CreateAnalysisTrack,
    CreateAnnotationTrack
  },
  setup (props) {
    // item.uuid should always be defined
    assertIsDefined(props.item.uuid)

    const store = useStore()
    const selection = ref<RegionType>()

    // Get playerRect and provide it for child components
    const el = ref<HTMLElement>()
    const playerSize = useBoundingClientRect(el)
    providePlayerRect(playerSize)

    const audioSrcs = computed(() => {
      assertIsDefined(props.item.uuid)
      return store.getters.items.getAudioSrcs(props.item.uuid)
    })
    const audioReady = computed(() => store.getters.audio.isReady)

    const onSelection = (val?: RegionType) => {
      selection.value = val
    }

    bindSelectionUrl(selection)

    const currentTab = ref(Tab.Analysis)
    function setTab (tab: Tab) {
      currentTab.value = tab
    }

    const isAnalysisTrackFormOpen = ref(false)
    const analysisTracks = analysisTrackStore(computed(() => props.item.uuid || ''))

    const isAnnotationTrackFormOpen = ref(false)
    const annotationTracks = annotationTrackStore(computed(() => props.item.uuid || ''))

    return {
      el,
      onSelection,
      selection,
      audioSrcs,
      audioReady,

      currentTab,
      setTab,
      Tab,

      isAnalysisTrackFormOpen,
      analysisTracks,

      isAnnotationTrackFormOpen,
      annotationTracks
    }
  }
})
</script>

<style lang="less" scoped>

.first-line {
  display: flex;
  justify-content: space-between;
  .title {
    font-size: 18px;
    text-align: left;
  }
}

.description {
  color: grey;
  font-size: 12px;
}

.line {
  justify-content: space-between;
}

.timer {
  text-align: right;
  font-size: 12px;
}

.menu {
  background: gainsboro;

  button {
    font-size: 16px;
    display: block;
    background: gainsboro;
    color: black;
    border: none;
    cursor: pointer;
    padding: 10px 15px;
  }
}

.tabs {
  display: flex;

  .tab {
    &.active {
      background: white;
      color: #606062;
    }
  }
}

.actions {
  display: flex;

  & button {
    display: flex;
    align-items: center;

    &.active {
      background-color: silver;
    }

    & .plus-picto {
      font-size: 30px;
      font-weight: 600;
      height: 15px;
      margin-top: -23px;
      margin-right: 5px;
    }
  }
}

.info-tab, .analysis-tab {
  padding: 20px 0;
}

.info-tab {
  text-align: left;
  font-size: 18px;
}

.info-table {
  tr {
    margin-bottom: 5px;
  }
  th {
    padding-left: 5px;
    padding-right: 20px;
    color: #444;
    background-color: #f9f9f9;
    border-right: .3em solid #ddd;
  }
  td {
    padding-top: 5px;
    padding-left: 20px;
  }
}

.create-form-container {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 200px;
}

.error {
  color: red;
}
</style>

<!-- Global style for the player -->
<style lang="less" scoped>
::v-deep {

  .form-title {
    font-weight: bold;
    font-size: 20px;
    margin-top: 0;
  }

  .generic-form {
    .select-wrapper {
      select {
        appearance: none;

        background: none;
        border: 1px solid grey;
        padding: 7px 10px;
        width: 100%;
        font-size: 16px;
      }

      position: relative;
      &:after {
        content: '<>';
        font: 17px "Consolas", monospace;
        transform: rotate(90deg);

        position: absolute;
        right: 7px;
        top: 7px;
        pointer-events: noneet
      }
    }

    input[type="text"], textarea {
      border: 1px solid grey;
      padding: 7px 10px;
    }

    input[type="checkbox"] {
      margin: 0;
      margin-right: 10px;
    }

    label {
      margin-right: 10px;
    }

    .field {
      margin-bottom: 10px;

      &.left {
        text-align: left;
      }
    }
  }

  .btn {
    padding: 7px 10px;
    font-size: 20px;
    font-weight: bold;
    border: none;
    width: 100%;
    cursor: pointer;
    margin-bottom: 5px;
    border-radius: 5px;
  }

  .green-btn {
    background: #32762F;
    color: white;
  }

  .grey-btn {
    background-color: #E9EAEC;
    color: black;
    font-weight: normal;
  }

  .red-btn {
    background-color: #DB4D27;
    color: white;
  }

  .info-box {
    position: absolute;
    top: 5px;
    right: 5px;
    z-index: 5;

    background-color: #c4c4c494;
    padding: 10px;
    width: 160px;
    border-radius: 5px;

    & .info-type {
      font-weight: bold;
      margin-top: 5px;
      margin-bottom: 10px;
      font-size: 18px;
    }

    & .info-title {
      margin-top: 5px;
      margin-bottom: 10px;
      font-size: 18px;
    }

    .delete {
      background-color: #DB4D27;
      color: white;
      border: none;
      font-size: 16px;
      padding: 7px 15px;
      border-radius: 5px;
      cursor: pointer;
    }
  }
}
</style>
