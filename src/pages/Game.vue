<template>
  <q-page class="flex flex-center column bg">
    <div class="game">
      <div class="flex">
        <div ref="red" class="btn red"></div>
        <div ref="green" class="btn green"></div>
      </div>
      <div class="flex">
        <div ref="blue" class="btn blue"></div>
        <div ref="yellow" class="btn yellow"></div>
      </div>
      <div class="menu">
        <span class="score">60</span>
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

border = 10px solid #101010

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
  border-top border
  border-left border
  border-top-left-radius 100%

.red-active
  background-color red-active

.green
  background-color green
  border-top border
  border-right border
  border-top-right-radius 100%

.green-active
  background-color green-active

.blue
  background-color blue
  border-bottom border
  border-left border
  border-bottom-left-radius 100%

.blue-active
  background-color blue-active

.yellow
  background-color yellow
  border-bottom border
  border-right border
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
  width 25%
  height 25%
  border-radius 100%
  background-color menu-bg-color
  border border
  font-family Font
  display flex
  align-items center
  justify-content center
  font-size 3em
  // color #04F404
  color font-color

</style>

<script>
import { defineComponent } from 'vue'

// const randomColor = () => ['red', 'green', 'blue', 'yellow'][Math.floor(Math.random() * 4)]

export default defineComponent({
  name: 'Game',
  data: () => ({
    seq: ['red', 'green', 'blue', 'yellow'],
    f: 2000
  }),
  computed: {
    test () {
      return this.seq.length
    }
  }
  /*
  methods: {
    emitSeq(){
      this.$refs[this.seq[i]] =
      setTimeout(, f)
    }
  }
  */
})
</script>
