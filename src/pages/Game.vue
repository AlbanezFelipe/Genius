<template>
  <q-page class="column flex-center bg">

    <!-- Game -->
    <div class="game">
      <div class="row">
        <div ref="red" class="btn red" @click="press('r')"></div>
        <div ref="green" class="btn green" @click="press('g')"></div>
      </div>
      <div class="row">
        <div ref="blue" class="btn blue" @click="press('b')"></div>
        <div ref="yellow" class="btn yellow" @click="press('y')"></div>
      </div>

      <!-- Menu -->
      <div class="menu">

        <!-- In Game -->
        <template v-if="state === 1">
          <span class="score">{{ score }}</span>
        </template>

        <!-- Game Over -->
        <template v-else-if="state === 2">
          <div class="column">
            <div class="row justify-center">
              <span class="score">{{ score }}</span>
            </div>
            <div class="row justify-center items-center">
              <q-icon @click="start" class="retry-btn" name="refresh"/>
              <q-icon @click="menu" class="home-btn" name="home"/>
            </div>
          </div>
        </template>

        <!-- Main Menu -->
        <template v-else>
          <div class="column">
            <div class="row justify-center play" @click="start">
              <q-icon name="play_arrow"/>
            </div>
            <div class="row justify-center items-baseline">
              <q-icon name="settings" class="settings" @click="settings" />
              <span class="row justify-center items-center record" @click="records"><q-icon name="emoji_events" /><span class="record-text">{{ bestScore }}</span></span>
            </div>
          </div>
        </template>

      </div>
    </div>

    <!-- Dialog Settings -->
    <q-dialog v-model="dialogSettings" transition-show="rotate" transition-hide="rotate">
      <q-card class="card-dialog">
        <q-card-section>
          <div class="text-h6">Settings</div>
        </q-card-section>

        <!-- Frequency -->
        <q-card-section class="q-pt-none">
          <div class="row items-center">
            <span class="q-mr-sm">Frequency (ms) {{ f }}</span>
            <q-badge rounded>
              <span class="help">?</span>
              <q-tooltip anchor="center right" self="center left" :offset="[4, 4]">
                Duration in milliseconds of color active
              </q-tooltip>
            </q-badge>
          </div>
          <q-slider v-model="f" :min="50" :max="1000" :step="50" label @change="saveSettings"/>

          <!-- Delay -->
          <div class="row items-center">
            <span class="q-mr-sm">Delay (ms) {{ d }}</span>
            <q-badge rounded>
              <span class="help">?</span>
              <q-tooltip anchor="center right" self="center left" :offset="[4, 4]">
                Duration in milliseconds of interval until next color plays
              </q-tooltip>
            </q-badge>
          </div>
          <q-slider v-model="d" :min="50" :max="1000" :step="50" label @change="saveSettings"/>

          <!-- Round Delay -->
          <div class="row items-center">
            <span class="q-mr-sm">Round Delay (ms) {{ rd }}</span>
            <q-badge rounded>
              <span class="help">?</span>
              <q-tooltip anchor="center right" self="center left" :offset="[4, 4]">
                Duration in milliseconds of interval until next round starts when player has finished the sequence<br>
                (recommended is be slightly greater than frequency + delay)
              </q-tooltip>
            </q-badge>
          </div>
          <q-slider v-model="rd" :min="50" :max="2000" :inner-min="f + d" :step="50" label @change="saveSettings"/>
        </q-card-section>

        <q-card-actions align="right">
          <q-btn flat label="OK" color="primary" v-close-popup />
        </q-card-actions>
      </q-card>
    </q-dialog>

    <!-- Dialog Records -->
    <q-dialog v-model="dialogRecords" transition-show="rotate" transition-hide="rotate">
      <q-card class="card-dialog">
          <q-card-section>
            <div class="text-h6">High Scores</div>
          </q-card-section>

          <q-card-section class="q-pt-none">
            <div class="q-pa-md">
              <q-list>
                <template v-for="(game, index) in highScores" :key="index">
                  <q-item>
                    <q-item-section avatar>
                      <template v-if="index < 3">
                        <q-icon :class="'highscore-' + index" name="emoji_events" />
                      </template>
                      <template v-else>
                        <span class="highscore">{{ index + 1 }}</span>
                      </template>
                    </q-item-section>

                    <q-item-section>
                    <div class="row items-end">
                      <span class="highscore-value">{{ game.score }}</span>
                      <div class="column">
                        <span class="highscore-info">{{ game.date }}</span>
                        <span class="highscore-info">({{ game.settings }})</span>
                      </div>
                    </div>
                    </q-item-section>
                  </q-item>
                </template>
              </q-list>
            </div>
          </q-card-section>

        <q-card-actions align="right">
          <q-btn flat label="OK" color="primary" v-close-popup />
        </q-card-actions>
      </q-card>
    </q-dialog>

  </q-page>
</template>

<style lang="stylus">

red           = #9F0F17
red-active    = #FF4C4C

green         = #00A74A
green-active  = #13FF7C

blue          = #094A8F
blue-active   = #1C8CFF

yellow        = #CCA707
yellow-active = #FED93F

bg-color =  #313131

gold   = #C9B037
silver = #B4B4B4
bronze = #AD8A56

border-size = 2.5vmin
border-color = #101010

game-size = 95vmin

menu-size = 25%
menu-size-calc = "calc((%s - %s) / 2)" % (game-size menu-size)

menu-bg-color = #181818

font-color = #B7B7B7

@font-face
  font-family Font
  font-style normal
  src url('../assets/Roboto_Mono/RobotoMono-VariableFont_wght.ttf')

.red
  background-color red
  border-style solid
  border-width border-size (border-size / 2) (border-size / 2) border-size
  border-color border-color
  border-top-left-radius 100%

.red-active
  background-color red-active

.green
  background-color green
  border-style solid
  border-width border-size border-size (border-size / 2) (border-size / 2)
  border-color border-color
  border-top-right-radius 100%

.green-active
  background-color green-active

.blue
  background-color blue
  border-style solid
  border-width (border-size / 2) (border-size / 2) border-size border-size
  border-color border-color
  border-bottom-left-radius 100%

.blue-active
  background-color blue-active

.yellow
  background-color yellow
  border-style solid
  border-width (border-size / 2) border-size border-size (border-size / 2)
  border-color border-color
  border-bottom-right-radius 100%

.yellow-active
  background-color yellow-active

.bg
  background-color bg-color

.game
  width game-size
  height game-size
  position relative

.game > .row
  width 100%
  height 50%

.btn
  width 50%

.menu
  position absolute
  top menu-size-calc
  left menu-size-calc
  width menu-size
  height menu-size
  border-radius 100%
  background-color menu-bg-color
  border border-size solid border-color
  font-family Font
  display flex
  align-items center
  justify-content center
  font-size 4vmin
  // color #04F404
  color font-color
  line-height 1

.score
  font-size 6vmin

.play, .settings, .retry-btn, .record, .home-btn
  cursor pointer

.play
  font-size 8vmin

.settings, .retry-btn
  margin-right 0.638vmin

.record
  background-color #242424
  border-radius 2.55vmin
  padding 0.319vmin 0.558vmin 0.319vmin 0.319vmin

  .record-text
    font-size 3.7vmin

.q-badge
  padding 4px
  width 20px
  height 20px

.help
  font-size 14px
  margin 0 auto

.card-dialog
  min-width 30%
  font-family Font
  color white
  background-color #121212

// 1st Gold
.highscore-0
  color gold
  transform scale(1.5)

// 2nd Silver
.highscore-1
  color silver
  transform scale(1.2)

// 3rd Bronze
.highscore-2
  color bronze

// > 3rd
.highscore
  font-weight bold
  text-align center
  width 24px

.highscore-value
  font-size 28px
  margin-right 16px
  min-width (28*1.2)px

.highscore-info
  font-size 12px
  color rgba(255,255,255,.5)

</style>

<script>

import { defineComponent } from 'vue'

import { Howl } from 'howler'

const randomColor = () => ['r', 'g', 'b', 'y'][Math.floor(Math.random() * 4)]

const colors = {
  r: 'red',
  g: 'green',
  b: 'blue',
  y: 'yellow'
}

const audio = {
  red: '../audio/red.mp3',
  green: '../audio/green.mp3',
  blue: '../audio/blue.mp3',
  yellow: '../audio/yellow.mp3',
  gameover: '../audio/gameover.mp3'
}

const initSettings = JSON.parse(localStorage.settings || '{}')
const initScores = JSON.parse(localStorage.scores || '[]').slice(0, 999)

export default defineComponent({
  name: 'Game',
  data: () => ({
    seq: [],
    state: 0, // 0 = menu; 1 = game; 2 = gameover;
    playing: false, // true = is playing sequence; false = waiting player finish sequence
    seqIndex: 0, // player index in sequence
    f: initSettings.f || 350, // active time per color
    d: initSettings.d || 200, // delay between colors in sequence
    rd: initSettings.rd || 750, // round delay (time for play sequence again when player has finished it)
    scores: initScores,
    dialogSettings: false,
    dialogRecords: false,
    lastTimeoutColor: null
  }),
  computed: {
    score () {
      return (this.seq.length || 1) - 1
    },
    bestScore () {
      return (this.scores.reduce((s, g) => g.score > s ? g.score : s, 0) + '').padStart(2, '0')
    },
    highScores () {
      return [...this.scores].sort((a, b) => a.score < b.score ? 1 : -1)
    }
  },
  methods: {
    // Start a new game
    start () {
      this.state = 1
      this.seq = []
      this.deactivateAll()
      this.next()
    },
    // Start game or next round
    next () {
      this.seq = [...this.seq, randomColor()] // generate sequence
      // this.seq = ['r', 'r']
      this.playing = true
      this.seqIndex = 0

      setTimeout(() => this.playSeq(), this.rd)
    },
    // Play color sequence
    playSeq (i = 0) {
      const c = this.seq[i]

      // if sequence finished
      if (!c) {
        this.playing = false
        return
      }

      // play color pressed
      this.playColor(colors[c])

      // frequency + delay to call next color
      setTimeout(() => this.playSeq(i + 1), this.f + this.d)
    },
    // Color pressed
    press (color) {
      // ignore if sequence is playing or game finished
      if (this.playing || this.state !== 1) {
        return
      }

      // if color is correct
      if (this.seq[this.seqIndex] === color) {
        this.playColor(colors[color])

        this.seqIndex += 1

        // if round finished
        if (this.seqIndex === this.seq.length) {
          this.next()
        }
        return
      }

      this.gameover() // otherwise gameover
    },
    // Gameover when miss a color
    gameover () {
      this.state = 2
      this.gameoverSequence()
      this.saveRecord()
    },
    gameoverSequence () {
      this.playAudio('gameover')
      for (let i = 0; i < 4; i++) {
        setTimeout(() => {
          if (this.state !== 2) return
          if (i % 2) this.deactivateAll(); else this.activateAll()
        }, i * 500)
      }
    },
    playColor (color) {
      this.deactivateAll() // deactivate other colors
      this.activate(color) // highlight color button
      this.playAudio(color) // play color audio

      // clear timeout from previous event
      if (this.lastTimeoutColor) clearTimeout(this.lastTimeoutColor)

      // deactivate highlight after frequency time
      this.lastTimeoutColor = setTimeout(() => { if (this.state === 1) this.deactivate(color) }, this.f)
    },
    playAudio (id) {
      new Howl({ src: audio[id] }).play()
    },
    activate (color) {
      this.$refs[color].classList.add(color + '-active')
    },
    activateAll () {
      Object.values(colors).forEach(c => this.activate(c))
    },
    deactivate (color) {
      this.$refs[color].classList.remove(color + '-active')
    },
    deactivateAll () {
      Object.values(colors).forEach(c => this.deactivate(c))
    },
    menu () {
      this.deactivateAll()
      this.state = 0
    },
    settings () {
      this.dialogSettings = true
    },
    saveSettings () {
      this.rd = this.f + this.d > this.rd ? this.f + this.d : this.rd

      localStorage.settings = JSON.stringify({
        f: this.f,
        d: this.d,
        rd: this.rd
      })
    },
    records () {
      this.dialogRecords = true
    },
    saveRecord () {
      this.scores = [...this.scores, {
        score: this.score,
        settings: [this.f, this.d, this.rd].join('|'),
        date: new Date().toISOString().split('T')[0]
      }]

      localStorage.scores = JSON.stringify(this.highScores)
    }
  }
})
</script>
