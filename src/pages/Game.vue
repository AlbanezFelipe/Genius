<template>
  <q-page class="flex flex-center column bg">
    <div class="game">
      <div class="flex">
        <div ref="red" class="btn red" @click="press('r')"></div>
        <div ref="green" class="btn green" @click="press('g')"></div>
      </div>
      <div class="flex">
        <div ref="blue" class="btn blue" @click="press('b')"></div>
        <div ref="yellow" class="btn yellow" @click="press('y')"></div>
      </div>
      <div class="menu">
        <template v-if="state === 1">
          <span class="score">{{ score }}</span>
        </template>
        <template v-else>
          <div class="flex-column">
            <div class="flex justify-center play" @click="start">
              <q-icon name="play_arrow"/>
            </div>
            <div class="flex justify-center items-baseline">
              <q-icon name="settings" class="settings" @click="settings" />
              <span class="flex justify-center items-center record" @click="records"><q-icon name="emoji_events" /><span class="record-text">60</span></span>
            </div>
          </div>
        </template>
      </div>
    </div>
  </q-page>
</template>

<style lang="stylus">

// #333333 : gray
// #ECE7EE : center
// #32050C : counter-color

red           = #9F0F17
red-active    = #FF4C4C

green         = #00A74A
green-active  = #13FF7C

blue          = #094A8F
blue-active   = #1C8CFF

yellow        = #CCA707
yellow-active = #FED93F

bg-color =  #313131

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

.game > .flex
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

.play
  cursor pointer
  font-size 8vmin

.settings
  cursor pointer
  margin-right 4px

.record
  cursor pointer
  background-color #242424
  border-radius 16px
  padding 2px 3.5px 2px 2px

  .record-text
    font-size 3.7vmin

</style>

<script>

import { defineComponent } from 'vue'

const randomColor = () => ['r', 'g', 'b', 'y'][Math.floor(Math.random() * 4)]

const colors = {
  r: 'red',
  g: 'green',
  b: 'blue',
  y: 'yellow'
}

const audio = {
  red: new Audio('../audio/red.mp3'),
  green: new Audio('../audio/green.mp3'),
  blue: new Audio('../audio/blue.mp3'),
  yellow: new Audio('../audio/yellow.mp3')
}

export default defineComponent({
  name: 'Game',
  data: () => ({
    seq: [],
    state: 0, // 0 = menu; 1 = game; 2 = gameover;
    playing: false,
    pressIndex: 0,
    f: 400, // keep f + d >= 600 until fix song crack
    d: 200
  }),
  computed: {
    score () {
      return this.seq.length
    }
  },
  methods: {
    start () {
      this.seq = [...this.seq, randomColor()]
      // this.seq = ['r', 'r', 'r', 'r', 'r', 'r', 'r', 'r', 'r', 'r', 'r']
      this.playSeq()
      this.playing = true
      this.pressIndex = 0
      // console.log(this.seq)
    },
    press (color) {
      if (!this.playing) {
        if (this.seq[this.pressIndex] === color) {
          // this.activate(colors[color])
          this.playAudio(colors[color])
          this.pressIndex += 1
          if (this.pressIndex === this.seq.length) {
            setTimeout(() => this.start(), this.d)
          }
          return
        }
        console.log('lose')
      }
    },
    playSeq (i = 0) {
      console.log('called')
      const c = this.seq[i]
      this.deactivateAll()
      if (c) {
        setTimeout(() => {
          // console.log(c)
          this.activate(colors[c])
          this.playAudio(colors[c])
          setTimeout(() => this.playSeq(i + 1), this.f)
        }, this.d)

        return
      }
      return this.finish()
    },
    finish () {
      this.playing = false
    },
    activate (color) {
      this.$refs[color].classList.add(color + '-active')
    },
    playAudio (color) {
      audio[color].pause()
      audio[color].currentTime = 0
      audio[color].play()
    },
    deactivate (color) {
      this.$refs[color].classList.remove(color + '-active')
    },
    deactivateAll () {
      Object.values(colors).forEach(c => this.deactivate(c))
    },
    settings () {
      console.log('open settings')
    },
    records () {
      console.log('open records')
    }
  }
})
</script>
