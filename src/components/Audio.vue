<template>
  <div class="audio">
    <audio
      v-if="audioSrcs"
      ref="el"
      autoplay
      controls
      @timeupdate="onTimeUpdate"
      @pause="onPause"
      @playing="onPlaying"
      @ended="onEnded"
      @error="audioError = $event"
      @durationchange="onDurationChange"
      @seeked="onSeeked"
    >
      <source
        v-for="src of audioSrcs"
        :key="src.src"
        :src="src.src"
        :type="src.type"
      >
      <p class="error">
        Your browser does not support the
        <code>audio</code> element.
      </p>
    </audio>
    <p
      v-else
      class="error"
    >
      Your selection have no valid audio file
    </p>
    <p
      v-if="audioError"
      class="error"
    >
      Unable to load media file: {{ audioError }}
    </p>
  </div>
</template>

<script lang="ts">
import {
  defineComponent,
  Ref,
  ref,
  onMounted,
  onUnmounted,
  PropType,
  computed,
  watch
} from '@vue/composition-api'
import { useStore } from '@/store/index'
import { PlayState, CurrentTimeSource } from '@/store/audio'
import { AudioSrc } from '@/store/items'
import { assertIsDefined } from '@/utils/type-assert'

export default defineComponent({
  props: {
    audioSrcs: {
      type: Array as PropType<AudioSrc[]>,
      required: true
    }
  },
  setup () {
    const store = useStore()
    const el: Ref<HTMLAudioElement | undefined> = ref()
    const audioError = ref<Error | undefined>()

    /*
    * Audio/Media element events
    */
    const onTimeUpdate = (ev: Event) => {
      assertIsDefined(ev.target)
      const target = ev.target as HTMLAudioElement
      store.commit.audio.setCurrentTime({
        value: target.currentTime * 1000,
        source: CurrentTimeSource.TimeUpdate
      })
    }
    const onPause = () => {
      assertIsDefined(el.value)

      // Avoids an infinite loop between play/pause state (reproducible on Firefox 73)
      if (el.value.seeking) {
        return
      }

      store.commit.audio.setCurrentTime({
        value: el.value.currentTime * 1000,
        source: CurrentTimeSource.TimeUpdate
      })
      store.commit.audio.setPlayState(PlayState.Pause)
    }
    const onPlaying = () => {
      assertIsDefined(el.value)
      store.commit.audio.setCurrentTime({
        value: el.value.currentTime * 1000,
        source: CurrentTimeSource.TimeUpdate
      })
      store.commit.audio.setPlayState(PlayState.Play)
    }
    const onEnded = () => {
      assertIsDefined(el.value)
      store.commit.audio.setCurrentTime({
        value: el.value.currentTime * 1000,
        source: CurrentTimeSource.TimeUpdate
      })
      store.commit.audio.setPlayState(PlayState.Stop)
    }
    const onDurationChange = () => {
      assertIsDefined(el.value)
      store.commit.audio.setDuration(el.value.duration * 1000)
    }
    const onSeeked = () => {
      assertIsDefined(el.value)
      store.commit.audio.setCurrentTime({
        value: el.value.currentTime * 1000,
        source: CurrentTimeSource.Seek
      })
    }

    /*
    * Set currentTime of audioFile when store's currentTime is updated
    */
    const currentTime = computed(() => store.state.audio.currentTime)
    onMounted(() => watch([ currentTime ], () => {
      // Do not update audio's currentTime if update comes from audio
      if (currentTime.value.source !== CurrentTimeSource.Cursor) {
        return
      }
      const audio = el.value
      if (!audio) {
        return
      }
      audio.currentTime = currentTime.value.value / 1000
    }, {
      flush: 'sync',
      immediate: true
    }))

    /*
    * Set playState to stop when there is an audio error
    */
    onMounted(() => watch([ audioError ], () => {
      if (audioError.value !== undefined) {
        store.commit.audio.setPlayState(PlayState.Stop)
      }
    }, { immediate: true }))

    /*
    * Update playState according to store's value
    */
    const playStateInput = computed(() => store.state.audio.playState)
    onMounted(() => watch([ playStateInput ], () => {
      const audio = el.value
      if (!audio) {
        // Store edit playState but audio is not mounted
        return
      }

      // Set currentTimeOutput to to avoid cursor jump
      store.commit.audio.setCurrentTime({
        value: audio.currentTime * 1000,
        source: CurrentTimeSource.TimeUpdate
      })

      switch (playStateInput.value) {
        case PlayState.Play:
          audio.play()
          break
        case PlayState.Pause:
          audio.pause()
          break
        case PlayState.Stop:
          audio.pause()
          audio.currentTime = 0
          break
        default:
          console.error('Unknown PlayState', playStateInput.value)
      }
    }, {
      // do not wait for component's update to run the watcher
      flush: 'sync',
      immediate: true
    }))

    const playbackRate = computed(() => store.state.audio.playbackRate)
    watch([ playbackRate ], () => {
      // Silently returns for before ready
      if (!el.value) {
        return
      }
      el.value.playbackRate = playbackRate.value
    }, { immediate: true })

    // Force the browser to unload the audio element
    // Without this, switching item without restarting the page
    // plays all previously loaded audio elemenets
    onUnmounted(() => {
      const audio = el.value
      if (!audio) {
        return
      }
      audio.pause()
      audio.currentTime = 0
      audio.removeAttribute('src')
      while (audio.firstChild) {
        audio.removeChild(audio.firstChild)
      }
      audio.load()
    })

    return {
      el,
      onTimeUpdate,
      onPause,
      onPlaying,
      onEnded,
      audioError,
      onDurationChange,
      onSeeked
    }
  }
})
</script>

<style lang="less" scoped>
.error {
  color: red;
}
</style>
