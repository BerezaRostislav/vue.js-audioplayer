<template>
  <div v-hotkey="keymap">
    <v-bottom-sheet full-width persistent hide-overlay :value="showPlayer">
      
      <v-card dark tile color="grey darken-3 text-xs-center">
        <v-layout row>
          <v-flex xs9>
            <div id="waveform"></div>
          </v-flex>

          <v-flex xs2 class="pl-5 py-2">
            <v-tooltip left>
              <v-slider
                slot="activator"
                hide-details
                :prepend-icon-cb="() => { volume = volume ? 0 : 0.5 }"
                :prepend-icon="muted ? 'mdi-volume-off' : 'mdi-volume-high'"
                step="0.05"
                v-model="volume"
                max="1"
                min="0"
              ></v-slider>
              <span>ГУЧНІСТЬ<br/><span>NumPad *</span> - додати<br/><span> NumPad /</span> - знизити<br/><span>ALT + M</span> - вимкнути</span>
            </v-tooltip>

              <v-tooltip left>
                <v-slider
                  slot="activator"
                  hide-details
                  :prepend-icon-cb="() => { speed = 1 }"
                  prepend-icon="mdi-transfer"
                  step="0.05"
                  v-model="speed"
                  max="2"
                  min="0.25"
                ></v-slider>
                <span>ШВИДКІСТЬ<br/><span>NumPad 3</span> - додати<br/><span> NumPad 1</span> - знизити<br/><span>NumPad 2</span> - зкинути</span>
              </v-tooltip>

              <v-tooltip left>
                <v-slider
                  slot="activator"
                  hide-details
                  :prepend-icon-cb="() => { zoom = 1 }"
                  prepend-icon="mdi-cursor-move"
                  step="5"
                  v-model="zoom"
                  max="200"
                  min="1"
                ></v-slider>
              <span>МАШТАБ<br/><span>NumPad 8</span> - додати<br/><span>NumPad 5</span> - знизити</span>
            </v-tooltip>
          </v-flex>

          <v-flex class="text-xs-right">
            <v-tooltip left>
              <v-btn
                slot="activator"
                icon flat large
                class="pa-0 ma-0"
                @click="quit(); playerToggle()"
              >
                <v-icon>mdi-close</v-icon>
              </v-btn>
              <span>ПЛЕЄР<br/><span>ALT + P</span></span>
            </v-tooltip>
            <br/>
            <v-tooltip left>
              <v-btn
                slot="activator"
                icon flat large
                class="pa-0 ma-0"
                @click="autoplayToggle"
              >
                <v-icon>{{ autoplay ? 'mdi-playlist-play' : 'mdi-playlist-remove' }}</v-icon>
              </v-btn>
              <span>АВТОПЛЄЙ<br/><span>ALT + O</span></span>
            </v-tooltip>
            <br/>
            <v-tooltip left>
              <v-btn
                slot="activator"
                icon flat large
                class="pa-0 ma-0"
                @click="clear"
              >
                <v-icon>mdi-recycle</v-icon>
              </v-btn>
              <span>СБРОС<br/><span>ALT + С</span></span>
            </v-tooltip>
          </v-flex>
        </v-layout>

        <v-progress-linear height="3" :value="progress" class="my-0"></v-progress-linear>
       
        <div id="wave-timeline"></div>
        <div id="minimap"></div>

        <v-list>
          <v-list-tile>
            <v-list-tile-content v-if="showPlayer">
              <v-list-tile-title class="title">
                Сессія //
                <span class="body-1">{{ now }} з {{ duration | duration }}</span>
              </v-list-tile-title>
              <v-list-tile-sub-title v-if="currentSession">{{ currentSession.fileLinkWav }}</v-list-tile-sub-title>
            </v-list-tile-content>
            
            <v-spacer></v-spacer>

            <v-list-tile-action>
              <v-tooltip top>
                <v-btn
                  slot="activator"
                  icon
                  @click="skipBackward(5)"
                >
                  <v-icon>mdi-skip-backward</v-icon>
                </v-btn>
                <span>Назад 5 сек.<br/><span>NumPad 7</span></span>
              </v-tooltip>
            </v-list-tile-action>
            
            <v-list-tile-action>
              <v-tooltip top>
                <v-btn
                  slot="activator"
                  icon
                  @click="skipBackward(2)"
                >
                  <v-icon>mdi-skip-previous</v-icon>
                </v-btn>
                <span>Назад 2 сек.<br/><span>NumPad 4</span></span>
              </v-tooltip>
            </v-list-tile-action>

            <v-list-tile-action :class="{ 'mx-3': $vuetify.breakpoint.mdAndUp }">
              <v-tooltip top>
                <v-btn
                  icon
                  slot="activator"
                  @click="playPause">
                  <v-icon>mdi-play-pause</v-icon>
                </v-btn>
                <span>Пауза-Плєй<br/><span>NumPad 0 / CTRL + SPACE</span></span>
              </v-tooltip>
            </v-list-tile-action>

            <v-list-tile-action :class="{ 'mx-3': $vuetify.breakpoint.mdAndUp }">
              <v-tooltip top>
                <v-btn
                  slot="activator"
                  icon
                  @click="stop"
                >
                  <v-icon>mdi-stop</v-icon>
                </v-btn>
                <span>Стоп<br/><span>ALT + S</span></span>
              </v-tooltip>
            </v-list-tile-action>

            <v-list-tile-action :class="{ 'mr-1': $vuetify.breakpoint.mdAndUp }">
              <v-tooltip top>
                <v-btn
                  slot="activator"
                  icon
                  @click="skipForward(2)"
                >
                  <v-icon>mdi-skip-next</v-icon>
                </v-btn>
                <span>Вперед 2 сек.<br/><span>NumPad 6</span></span>
              </v-tooltip>
            </v-list-tile-action>
              
            <v-list-tile-action :class="{ 'mr-1': $vuetify.breakpoint.mdAndUp }">
              <v-tooltip top>
                <v-btn
                  slot="activator"
                  icon
                  @click="skipForward(5)"
                >
                  <v-icon>mdi-skip-forward</v-icon>
                </v-btn>
                <span>Вперед 5 сек.<br/><span>NumPad 9</span></span>
              </v-tooltip>
            </v-list-tile-action>

          </v-list-tile>
        </v-list>

      </v-card>
    </v-bottom-sheet>
  </div>
</template>

<!--  <v-flex xs10>
        <div id="waveform"></div>
        <div id="equalizer" style="margin-top: 10px"></div>
      </v-flex>
    -->

<script>
  import { mapState, mapGetters } from 'vuex'

  import WaveSurfer from 'wavesurfer.js'
  import RegionPlugin from 'wavesurfer.js/dist/plugin/wavesurfer.regions.min.js'
  import TimelinePlugin from 'wavesurfer.js/dist/plugin/wavesurfer.timeline.min.js'
  import MinimapPlugin from 'wavesurfer.js/dist/plugin/wavesurfer.minimap.min.js'
// import CursorPlugin from 'wavesurfer.js/dist/plugin/wavesurfer.cursor.min.js'

export default {
    name: 'player',

    data () {
      return {
        player: true,
        autoplay: false,
        wavesurfer: null,
        audio: null,
        zoom: 1,
        filters: null,
        volume: 0.5,
        speed: 1,
        now: '',
        EQ: require('~/assets/seeds/equalizer').default
      }
    },

    created () {
      this.$bus.$on('play', this.play)
      this.$bus.$on('playPause', this.playPause)
    },

    mounted () {
      const self = this
  
      this.$nextTick(function () {
        this.wavesurfer = WaveSurfer.create({
          backend: 'MediaElement',
          container: '#waveform',
          // barWidth: 1,
          // barHeight: 1.4,
          height: '100',
          // scrollParent: true,
          hideScrollbar: true,
          waveColor: 'lightBlue',
          progressColor: 'orange',
          plugins: [
            RegionPlugin.create({
              dragSelection: true,
              loop: true,
              resize: true
            }),
            TimelinePlugin.create({
              container: '#wave-timeline'
            }),
            // CursorPlugin.create(),
            MinimapPlugin.create()
          ]
        })

        this.wavesurfer.on('audioprocess', function (time) {
          const minutes = ('00' + Math.floor((time % 3600) / 60)).slice(-2)
          const seconds = ('00' + Math.floor(time % 60)).slice(-2)

          self.now = minutes + ':' + seconds
        })
  
        this.wavesurfer.on('ready', function () {

        })
      })
    },

    watch: {
      audio (newValue) {
        newValue ? this.play(newValue) : this.quit()
      },

      volume (newValue) {
        this.setVolume(newValue)
      },

      speed (newValue) {
        this.setPlaybackRate(newValue)
      },

      zoom (newValue) {
        this.setZoom(newValue)
      },

      currentSession (newValue) {
        if (newValue && newValue.fileLinkWav && this.basePath) {
          this.audio = `${this.basePath}${newValue.fileLinkWav}`
        }
      }
    },

    methods: {
      // onChange ($event, filter) {
      //   filter.gain.value = ~~$event.target.value
      // },

      playerToggle () {
        this.player = !this.player
      },

      autoplayToggle () {
        this.autoplay = !this.autoplay
      },

      setVolume (value) {
        if (!this.wavesurfer || value < 0 || value > 1) return
        this.wavesurfer.setVolume(value)
      },

      setPlaybackRate (value) {
        if (!this.wavesurfer || value < 0.25 || value > 2) return
        this.wavesurfer.setPlaybackRate(value)
      },
  
      setSpeed (rate) {
        this.speed = rate
      },

      setZoom (zoom) {
        if (!this.wavesurfer || zoom < 1 || zoom > 200) return
        this.wavesurfer.zoom(Number(zoom))
      },

      play (audio) {
        if (!audio || !this.wavesurfer) return
        this.wavesurfer.load(audio)
        if (this.autoplay) {
          this.player = true
          this.wavesurfer.play()
        }
      },
  
      stop ($event) {
        if (!this.wavesurfer) return
        $event.stopPropagation()
        $event.preventDefault()
        this.wavesurfer.stop()
      },

      playPause () {
        if (!this.wavesurfer) return

        if (!this.wavesurfer.isPlaying()) {
          this.wavesurfer.skipBackward(0.5)
        }
        this.wavesurfer.playPause()
      },

      skipForward (seconds) {
        if (!this.wavesurfer) return
        this.wavesurfer.skipForward(seconds)
      },
  
      skipBackward (seconds) {
        if (!this.wavesurfer) return
        this.wavesurfer.skipBackward(seconds)
      },

      quit () {
        if (!this.wavesurfer) return
        this.audio = ''
        this.player = false
        this.wavesurfer.stop()
        this.wavesurfer.empty()
      },

      clear () {
        if (!this.wavesurfer) return
        this.volume = 0.5
        this.speed = 1
        this.zoom = 1
        this.wavesurfer.clearRegions()
      },

      toggleMute () {
        this.wavesurfer.toggleMute()
      },

      volumeUp () {
        if (this.volume >= 1) return
        this.volume += 0.1
      },

      volumeDown () {
        if (this.volume <= 0) return
        this.volume -= 0.1
      },

      createEQFilters (EQ) {
        return EQ.map((band) => {
          var filter = this.wavesurfer.backend.ac.createBiquadFilter()
          filter.type = band.type
          filter.frequency.value = band.f
          filter.gain.value = 0
          filter.Q.value = 1
          return filter
        })
      }
    },

    computed: {
      ...mapGetters({ currentSession: 'session/getCurrent' }),
      ...mapState('config', { mediaPath: 'mediaPath' }),

      basePath () {
        return this.mediaPath + 'testbox/'
        // return 'https://athena-test.corvax.kiev.ua:20143/data/'
      },

      showPlayer () {
        // const result =
        //   (this.$route.name === 'session-id' && this.currentSession && this.currentSession.fileLinkWav) ||
        //   (this.player && this.currentSession && this.currentSession.fileLinkWav) ||
        //   (this.wavesurfer && this.wavesurfer.isPlaying())
        // return result
        return true
      },
  
      muted () {
        if (!this.wavesurfer) return
        return !this.volume || this.wavesurfer.getMute()
      },

      duration () {
        if (!this.wavesurfer) return
        const duration = this.wavesurfer.getDuration()
        return duration
      },

      progress: {
        cache: false,
        get () {
          if (!this.wavesurfer) return
          const proc = this.wavesurfer.getCurrentTime() * 100 / this.wavesurfer.getDuration()
          return proc
        }
      },

      keymap () {
        return {
          'alt+s': this.stop,
          'alt+q': this.quit,
          'alt+c': this.clear,
          'alt+p': this.playerToggle,
          'alt+o': this.autoplayToggle,
          'alt+m': this.toggleMute,
          'numpad 6': () => { this.skipForward(2) },
          'numpad 9': () => { this.skipForward(5) },
          'numpad 4': () => { this.skipBackward(2) },
          'numpad 7': () => { this.skipBackward(5) },
          'numpad 1': () => { this.speed -= 0.15 },
          'numpad 2': () => { this.setSpeed(1) },
          'numpad 3': () => { this.speed += 0.15 },
          'numpad 8': () => { this.zoom += 5 },
          'numpad 5': () => { this.zoom -= 5 },
          'numpad 0': this.playPause,
          'numpad *': this.volumeUp,
          'numpad /': this.volumeDown
        }
      }
    }
  }
</script>

<style>
  .input-group.input-group--slider {
    padding-top: 4px;
  }
</style>