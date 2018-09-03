<template>
    <div class="slot-machine">
        <div class="slot-machine__prize-frame">
            <div class="slot-machine__prize-list" 
                :style="[listTransition(), listPositionStyle()]">
                
                <div v-for="n in itemSetCount" :key="`setNo${n}`">
                    <div v-for="(set, key) in prizeSet" 
                        :key="`setItem${Object.keys(prizeSet).length + 1 - key}`"
                        :style="prizeItemStyle()"
                        class="prize"
                        :class="{
                            'prize--target': isTargetPrize((itemSetCount - n ) * prizeLength + Object.keys(prizeSet).length + 1 - key  )
                        }">
                        <span>{{ prizeSet[Object.keys(prizeSet).length + 1 - key] }} 
                            <span class="hint">(第 {{ (itemSetCount - n ) * prizeLength + Object.keys(prizeSet).length + 1 - key }} 個 item)</span>
                        </span>
                    </div>
                </div>
            </div>

            <div v-if="gameStatus == 0" class="slot-machine__mask">
                <span v-if="allowPlay">點擊按鈕開始!</span>
            </div>
        </div>

        <div class="slot-machine__handle-wrapper">
            <button v-if="gameStatus == 0" @click="go">start game!</button>
            <button v-if="gameStatus == 3" @click="reset">reset game</button>
        </div>
    </div>
</template>

<script>
export default {
    props: {
        secondsToStop: { type: Number, default: 3500 }, // 幾秒後開始變慢
        secondsToEnd: { type: Number, default: 3000 }, // 變慢後，幾秒後真的停住
        targetPrize: { type: Number }, // 目標獎項
        allowPlay: { type: Boolean }, // 可以繼續下一次抽獎
        prizeLength: { type: Number }, // 獎品長度
        prizeSet: { type: Object }, // 獎項池
    },
    data() {
        return {
            itemSetCount: 1, // 總共有幾組獎品 
            gameStatus: 0, // 0 -> before start, 1 -> when machine is runing, 2 -> slow down and is about to end, 3 -> when game ends
            currentItem: 1, // 現在移動到第幾個獎品
            speed: 0, // 一次移動多少格子
            itemHeight: 82, // 每一個格子的高度
        }
    },
    computed: {
        // 目標獎品的位子
        targetItem() {
            let mul = Math.ceil(this.currentItem / this.prizeLength); // 最接近目前的獎品組別
            return (mul + 2) * this.prizeLength + this.targetPrize;
        }
    },
    watch: {
        // 目前顯示的位子
        currentItem() {
            if (this.currentItem > this.prizeLength * this.itemSetCount - 1) {
                this.itemSetCount += 4;
            }
        }
    },
    mounted() {
        // DOM 建立好時決定好每一格的高度
        // 因為一次要出現三格，所以每格高度是 mask / 3
        setTimeout(() => {
            console.log(document.querySelector('.slot-machine__prize-frame').clientHeight)
            this.itemHeight = document.querySelector('.slot-machine__prize-frame').clientHeight / 3;
        }, 100);
    },
    methods: {
        stop() {
            clearInterval(this.animate);
            this.gameStatus = 2;
            this.currentItem = this.targetItem;
            setTimeout(() => {
                this.gameStatus = 3;
            }, this.secondsToEnd + 200);
            setTimeout(() => {
                this.$emit('game-ends');
            }, this.secondsToEnd + 500);
        },
        go() {
            clearInterval(this.animate);
            this.gameStatus = 1;
            this.animate = setInterval(() => {
                this.currentItem += this.speed;
                if (this.speed < 1) {
                    this.speed += 2;
                }
            }, 60);
            setTimeout(() => {
                if (this.gameStatus == 1) {
                    this.stop();
                }
            }, this.secondsToStop);
        },
        reset() {
            if (this.allowPlay) {
                this.currentItem = 1;
                this.gameStatus = 0;
                this.speed = 0;
                this.itemSetCount = 1;
            }
        },
        // 獎品列表的 style
        listPositionStyle() {
            return {
                transform: this.currentItem < 2
                    ? 'translateY(0px)'
                    : `translateY(${(this.currentItem - 1) * this.itemHeight - this.itemHeight}px)`
            }
        },
        // 獎品列表的漸變樣式
        listTransition() {
            switch (this.gameStatus) {
                case 1:
                    return {
                        transition: '1000ms all ease-out'
                    }
                    break;
                case 2:
                    return {
                        transition: `${this.secondsToEnd}ms all cubic-bezier(.25,1.31,.33,.95)`
                    }
                    break;
                default:
                    break;
            }
        },
        // 每格獎品的高度
        prizeItemStyle() {
            return {
                height: `${this.itemHeight}px`
            }
        },
        // 遊戲開始前遮罩
        prizeMaskStyle() {
            return {
                height: `${this.itemHeight * 3}px`
            }
        },
        // 目標獎品
        isTargetPrize(key) {
            return this.gameStatus == 3 && key == this.currentItem;
        }
    }
}
</script>

<style lang="scss" scoped>
.slot-machine {
  width: 350px;
  position: relative;
  text-align: center;

  &__machine-img {
    width: 100%;
  }

  &__prize-frame {
    width: 100%;
    height: 200px;
    margin: auto;
    overflow: hidden;
    position: relative;
    border: 1px solid blue;
  }

  &__prize-list {
    width: 100%;
    position: absolute;
    bottom: 0;

    .hint {
      font-size: 12px;
      color: gray;
    }
    .prize {
      border: 1px solid lightgray;
      display: inline-flex;
      width: 100%;

      &--target {
        animation: targetPrize 250ms infinite alternate-reverse linear;
      }

      @keyframes targetPrize {
        0% {
          border: 2px solid rgb(242, 114, 114);
        }

        100% {
          border: 2px solid pink;
        }
      }

      * {
        margin: auto;
      }
    }
  }

  &__mask {
    position: absolute;
    width: 100%;
    height: 100%;
    background: #feef35;
    display: flex;

    * {
      margin: auto;
    }
  }

  &__handle-wrapper {
    display: flex;
    width: 100%;
    height: 50px;
    justify-content: center;
    margin-top: 10px;
    > button {
      border-radius: 5px;
      background: blue;
      color: white;
      outline: none;
      padding: 10px 20px;
    }
  }
}
</style>
