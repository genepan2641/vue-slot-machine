<template>
  <div id="app" class="app-container">
    <div class="flex-box">
      <div class="flex-1">
        <div>
          <input type="range" v-model.number="seconds" min="4" max="10" step="0.5">
          {{ seconds }} 秒後停止
        </div>
        <label v-for="(prize, key) in prizeSet" :for="`prize${prize}`">
          <input :id="`prize${prize}`" v-model="targetPrize" type="radio" :value="Number(key)">
          {{ prize }}
        </label>
      </div>
      <div class="flex-1">
        <game 
          :allow-play="allowPlay"
          :prize-length="prizeLength"
          :target-prize="targetPrize"
          :seconds-to-stop="secondsToStop"
          :seconds-to-end="secondsToEnd"
          :prize-set="prizeSet">
        </game>
      </div>
    </div>
  </div>
</template>

<script>
import Game from './components/Game.vue'

export default {
  data() {
    return {
      seconds: 4,
      allowPlay: true,
      secondsToStop: 1500, // ms 後執行停止
      secondsToEnd: 3000, // 停止後 動畫持續幾秒
      targetPrize: 1,
      prizeSet: {
        1: '一獎',
        2: '二獎',
        3: '三獎',
        4: '四獎'
      },
    }
  },
  computed: {
    prizeLength() {
      return Object.keys(this.prizeSet).length;
    }
  },
  watch: {
    seconds: {
      handler(val) {
        console.log(Number(val))
        this.secondsToStop = Number(val) * 1000 - this.secondsToEnd; // transform to ms
      },
      immediate: true
    }
  },
  name: 'App',
  components: {
    Game
  }
}
</script>

<style>
* {
  box-sizing: border-box;
}

.w100 {
  width: 100%;
}

.app-container {
  max-width: 700px;
  margin: 0 auto;
}

.flex-box {
  display: flex;
  flex-wrap: wrap;
}

.flex-1 {
  flex: 1;
}

.flex-2 {
  flex: 2;
}
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
