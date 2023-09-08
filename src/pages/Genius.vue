<template>
  <div class="column flex-center bg">

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
              <q-icon @click="showSeq" class="question-btn" name="question_mark"/>
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
    <q-dialog v-model="dialogSettings" transition-show="scale" transition-hide="scale">
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

          <div class="row items-center">
            <span class="q-mr-sm">Click Limit (ms) {{ clickLimit }}</span>
            <q-badge rounded>
              <span class="help">?</span>
              <q-tooltip anchor="center right" self="center left" :offset="[4, 4]">
                For avoid accidentally click twice<br>
                (if you do not want this just set to 0)
              </q-tooltip>
            </q-badge>
          </div>
          <q-slider v-model="clickLimit" :min="0" :max="300" :step="10" label @change="saveSettings"/>
        </q-card-section>

        <q-card-actions align="right">
          <q-btn flat label="OK" color="primary" v-close-popup />
        </q-card-actions>
      </q-card>
    </q-dialog>

    <!-- Dialog Records -->
    <q-dialog v-model="dialogRecords" transition-show="scale" transition-hide="scale">
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

    <!-- Dialog Sequence -->
    <q-dialog v-model="dialogSequence" transition-show="scale" transition-hide="scale">
      <q-card class="card-dialog">

          <q-card-section class="q-pt-none">
            <div class="q-pa-md">
              <template v-for="(color, index) in seq" :key="index">
                <span :class="'text-' + color">{{ color.toUpperCase() }}</span>
                <span v-if="seqIndex === index"> (<span :class="'text-' + blunder">{{ blunder.toUpperCase() }}</span> ??)</span>
                <span v-if="index + 1 < seq.length"> - </span>
              </template>
            </div>
          </q-card-section>

        <q-card-actions align="right">
          <q-btn flat label="OK" color="primary" v-close-popup />
        </q-card-actions>
      </q-card>
    </q-dialog>

  </div>
</template>

<style lang="stylus">
  @import '../css/genius.styl'
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
  name: 'Genius',
  data: () => ({
    seq: [],        // Game sequence
    state: 0,       // 0 = menu; 1 = game; 2 = gameover;
    playing: false, // true = is playing sequence; false = waiting player finish sequence
    seqIndex: 0,    // Player index in game sequence
    blunder: '',    // Blunder color when player misses, used in dialog sequence
    f: initSettings.f || 350,   // Active time per color
    d: initSettings.d || 200,   // Delay between colors in sequence
    rd: initSettings.rd || 750, // Round delay (time for play sequence again when player has finished it)
    clickLimit: initSettings.clickLimit || 100, // For avoid accidentally click twice
    scores: initScores,    // Records data
    dialogSettings: false, // Dialog Settings
    dialogRecords: false,  // Dialog Records
    dialogSequence: false, // Dialog Sequence
    lastTimeoutColor: null, // Last setTimeout id of a color pressed
    clickLimitBlocked: false // For click limit control
  }),
  computed: {
    // Number of rounds beat
    score () {
      return (this.seq.length || 1) - 1
    },
    // Higher score in records
    bestScore () {
      return (this.scores.reduce((s, g) => g.score > s ? g.score : s, 0) + '').padStart(2, '0')
    },
    // Records in descending by score
    highScores () {
      return [...this.scores].sort((a, b) => a.score < b.score ? 1 : -1)
    }
  },
  methods: {
    // Start a new game
    start () {
      this.state = 1
      this.seq = []
      // this.seq = new Array(25).fill().map(randomColor) // autofill
      this.deactivateAll()
      this.next()
    },
    // Start game or next round
    next () {
      this.seq = [...this.seq, randomColor()] // generate sequence
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
      // ignore if sequence is playing or in gameover state
      if (this.playing || this.state === 2) {
        return
      }

      // ignore if clickLimitBlocked is activated
      if (this.clickLimitBlocked) return

      // click limit manager
      this.clickLimitBlocked = true
      setTimeout(() => {
        this.clickLimitBlocked = false
      }, this.clickLimit)

      // menu is free to play color
      if (this.state === 0) {
        this.playColor(colors[color])
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

      // otherwise gameover
      this.blunder = color
      this.gameover()
    },
    // Go to Gameover state when miss a color
    gameover () {
      this.state = 2
      this.gameoverSequence()
      this.saveRecord()
    },
    // Gameover blink all colors twice and play gameover buzzer
    gameoverSequence () {
      this.playAudio('gameover')
      for (let i = 0; i < 4; i++) {
        setTimeout(() => {
          if (this.state !== 2) return
          if (i % 2) this.deactivateAll(); else this.activateAll()
        }, i * 500)
      }
    },
    // Play color audio and highlight it
    playColor (color) {
      this.deactivateAll() // deactivate other colors
      this.activate(color) // highlight color button
      this.playAudio(color) // play color audio

      // clear timeout from previous event
      if (this.lastTimeoutColor) clearTimeout(this.lastTimeoutColor)

      // deactivate highlight after frequency time
      this.lastTimeoutColor = setTimeout(() => { if (this.state !== 2) this.deactivate(color) }, this.f)
    },
    // Play a audio
    playAudio (id) {
      new Howl({ src: audio[id], html5: true }).play()
    },
    // Highlight a color button
    activate (color) {
      this.$refs[color].classList.add(color + '-active')
    },
    // Highlight all color buttons
    activateAll () {
      Object.values(colors).forEach(c => this.activate(c))
    },
    // Remove highlight from a color button
    deactivate (color) {
      this.$refs[color].classList.remove(color + '-active')
    },
    // Clear all highlights
    deactivateAll () {
      Object.values(colors).forEach(c => this.deactivate(c))
    },
    // Go to menu state
    menu () {
      this.deactivateAll()
      this.state = 0
    },
    // Open dialog settings
    settings () {
      this.dialogSettings = true
    },
    // Store settings in localStorage
    saveSettings () {
      this.rd = this.f + this.d > this.rd ? this.f + this.d : this.rd // Fix Round Delay offset

      localStorage.settings = JSON.stringify({
        f: this.f,
        d: this.d,
        rd: this.rd,
        clickLimit: this.clickLimit
      })
    },
    // Open dialog records
    records () {
      this.dialogRecords = true
    },
    // Save current game record in localStorage
    saveRecord () {
      this.scores = [...this.scores, {
        score: this.score,                             // Game Score
        settings: [this.f, this.d, this.rd].join('|'), // Current Settings in "n|n|n" format
        date: new Date().toISOString().split('T')[0]   // Current Date
      }]
      localStorage.scores = JSON.stringify(this.highScores)
    },
    // Open dialog sequence
    showSeq () {
      this.dialogSequence = true
    }
  }
})
</script>
