<template>
<div class="container">
  <!-- <modal name="hello-world">
    文字：
    <input type="text" v-model="itemText">
    <button @click="addItem">新增</button>
  </modal>
  <button @click="add">新增</button>
  <button @click="reset">清除</button> -->

    <div class="turnTable" v-show="true">
        <div class="turnTable__body">
          <canvas class="turnTable__canvas" :class="animitionType"
          :style="{'width': `${config.baseSize}px`, 'height': `${config.baseSize}px`}">
          </canvas>
        </div>
        <div class="turnTable__button" @click="animitionProc">
            <div class="turnTable__arrow"></div>
            <div class="turnTable__button__content">
              <span class="turnTable__button__text">{{ config.buttonText }}</span>
            </div>
        </div>
    </div>
</div>
</template>

<script>
export default {
  name: 'TurnTable',
  data () {
    return {
      isRunning: false,

      turnStatus: {
        currentDeg: 0,
        targetDeg: 0,
        rollBackDeg: 0
      },
      defaultGifts: [
        { chance: 10, text: '做', textColor: '', textSize: '', backgroundColor: '', edit: false },
        { chance: 10, text: '不做', textColor: '', textSize: '', backgroundColor: '', edit: false }
      ],
      /** 轉盤設定 */
      config: {},
      defaultConfig: {
        autoStop: true,
        runTime: 1,
        rollBackRange: 0.25,
        showAlert: true,
        baseSize: 500,
        singleColor: '#488d12',
        doubleColor: '#ce0235',
        borderWidth: 1,
        borderColor: '#ffffff',
        textColor: '#fff',
        buttonColor: '#f6d7d7',
        buttonText: 'GO'
      },
      itemText: ''
    }
  },
  watch: {
    isRunning (boolean) {
      // 手動模式要將動畫停止
      if (!this.config.autoStop) {
        const state = boolean ? 'running' : 'paused'
        document.documentElement.style.setProperty('--animitionState', state)
      }
    },
    turnStatus: {
      handler () {
        this.updateturnStatus()
      },
      deep: true
    }
  },
  computed: {
    /** 動畫Class判斷 */
    animitionType () {
      return {
        'turnTable__canvas--manualTrun': !this.config.autoStop,
        'turnTable__canvas--autoTrun': this.isRunning && this.config.autoStop
      }
    },
    /** 計算區塊機率值總和 */
    countDataChance () {
      let totalChance = 0
      this.defaultGifts.forEach((data) => {
        totalChance += data.chance
      })
      return totalChance
    },
    /** 預設區塊色 */
    defaultGiftBackgroundColor () {
      return this.getDefaultGiftBackgroundColor()
    },
    /** 取得裝置像素比例(至少用兩倍來避免canvas繪製模糊) */
    pixelRatio () {
      return window.devicePixelRatio * 2 || 2
    }
  },
  methods: {
    reset () {
      this.defaultGifts = this.defaultGifts.slice(0, 2)
      this.buideTurnTable()
    },
    add () {
      this.$modal.show('hello-world')
    },
    addItem () {
      this.$modal.hide('hello-world')
      this.defaultGifts.push({ chance: 10, text: this.itemText, textColor: '', textSize: '', backgroundColor: '', edit: false })
      // this.gift = Object.assign({}, this.defaultGift)
      // this.gifts = this.getDefaultGfits()
      this.buideTurnTable()
    },
    getDefaultGiftBackgroundColor (index) {
      // const number = index || this.defaultGifts.length
      // return number % 2 === 0 ? this.config.doubleColor : this.config.singleColor
      return index % 2 === 0 ? this.config.doubleColor : this.config.singleColor
    },
    getDefaultGfits () {
      return Array.from(this.defaultGifts)
    },
    getDefaultConfig () {
      // 計算當前視窗寬高，取低值*0.6當基準值設定轉盤大小
      const innerHeight = window.innerHeight
      const innerWidth = window.innerWidth
      this.defaultConfig.baseSize = innerHeight > innerWidth
        ? Math.floor(innerWidth * 0.8) : Math.floor(innerHeight * 0.6)
      return Object.assign({}, this.defaultConfig)
    },
    drawCanvas () {
      const centerPoint = this.config.baseSize * (this.pixelRatio / 2)
      const turnTable = document.querySelector('.turnTable__canvas')
      const ctx = turnTable.getContext('2d')
      turnTable.setAttribute('width', this.config.baseSize * this.pixelRatio)
      turnTable.setAttribute('height', this.config.baseSize * this.pixelRatio)
      this.giftDegs = []
      let lastAngle = 0
      // 內部區塊繪製
      this.defaultGifts.forEach((gift, index) => {
        console.log('in foreach, idnex:', index)
        // 計算角度(全部資料機率 / 單片機率 * 360)
        console.log('此時的機率總和:', this.countDataChance)
        const deg = (gift.chance / this.countDataChance) * 360
        // 計算弧度(角度 * PI / 180)，
        const angle = deg * (Math.PI / 180)
        // 儲存角度範圍
        this.giftDegs[index] = {
          from: index === 0 ? 0 : this.giftDegs[index - 1].to,
          to: index === 0 ? deg : this.giftDegs[index - 1].to + deg,
          name: gift.text
        }
        // 開始繪製
        ctx.save()
        ctx.beginPath()
        ctx.translate(centerPoint, centerPoint)
        ctx.moveTo(0, 0)
        // 旋轉弧度 = 上次的區塊弧度(初始0)
        ctx.rotate(lastAngle)
        // 繪製區塊，半徑-外框線 避免 框線擠到canvas邊框
        ctx.arc(0, 0, centerPoint - this.config.borderWidth, 0, angle, false)
        // 更新最後一次的結束角度
        lastAngle += angle
        // 區塊底色填充
        ctx.fillStyle = gift.backgroundColor || this.getDefaultGiftBackgroundColor(index)
        ctx.fill()
        /** 邊框繪製 */
        ctx.lineWidth = this.config.borderWidth * this.pixelRatio
        ctx.strokeStyle = this.config.borderColor
        ctx.closePath()
        ctx.stroke()
        // 內容文字繪製
        ctx.rotate(angle / 2)
        ctx.fillStyle = gift.textColor || this.config.textColor
        ctx.font = gift.textSize
          ? `${gift.textSize}px Microsoft JhengHei`
          : `${(this.config.baseSize / gift.text.length) * (this.pixelRatio / 4)}px Microsoft JhengHei`
        ctx.textBaseline = 'middle'
        ctx.fillText(gift.text, centerPoint / 2.25, 0)
        //
        ctx.restore()
      })
      // 自動停止模式動畫結束時要跑autoTurnStop function
      if (this.config.autoStop) {
        turnTable.addEventListener('animationend', this.autoTurnStop)
      } else {
        turnTable.removeEventListener('animationend', this.autoTurnStop)
      }
    },
    /** 轉動模式 */
    animitionProc () {
      if (this.config.autoStop) {
        this.autoTurnStart()
      } else {
        // this.manualTrun();
      }
    },
    /** 自動模式(自動停止) */
    autoTurnStart () {
      // 如果正在執行中不動作
      if (this.isRunning) return
      // 取得隨機角度(預設至少跑10圈)
      const randomDeg = Math.floor(Math.random() * (360 - 0)) + (360 * 4)
      // 取得隨機回彈角度
      // const randomRollBackDeg = (Math.random() * this.config.rollBackRange) + 1;
      // // 設定回彈角度
      // this.turnStatus.rollBackDeg = randomRollBackDeg < 1.01 ? 1 : randomRollBackDeg;
      // 設置動畫的目標角度
      this.turnStatus.targetDeg = randomDeg
      // 開始旋轉動畫
      this.isRunning = true
    },
    /** 自動模式停止事件(由繪製區的addEventListener('animationend')觸發) */
    autoTurnStop () {
      // 關閉動畫
      this.isRunning = false
      // 紀錄本次角度
      this.turnStatus.currentDeg = Math.floor(this.turnStatus.targetDeg % 360)
    },
    /** 建立轉盤 */
    buideTurnTable () {
      console.log('呼叫 build Turn table')
      // CSS值設定
      // document.documentElement 為 html 這個元素
      document.documentElement.style.setProperty('--turnTableSize', `${this.config.baseSize + 20}px`)
      document.documentElement.style.setProperty('--buttonSize', `${this.config.baseSize / 3.5}px`)
      document.documentElement.style.setProperty('--buttonFontSize', `${this.config.baseSize / 10}px`)
      document.documentElement.style.setProperty('--arrowHeight', `${this.config.baseSize / 8}px`)
      document.documentElement.style.setProperty('--arrowWidth', `${this.config.baseSize / 5}px`)
      document.documentElement.style.setProperty('--buttonColor', `${this.config.buttonColor}`)
      document.documentElement.style.setProperty('--runTime', `${this.config.runTime}s`)
      document.documentElement.style.setProperty('--animitionState', 'paused')
      // 清空轉出結果
      this.resultList = []
      // 繪製轉盤
      this.drawCanvas()
    },
    /** 更新轉盤角度資訊 */
    updateturnStatus () {
      document.documentElement.style.setProperty('--targetDeg', `${this.turnStatus.targetDeg}deg`)
      document.documentElement.style.setProperty('--currentDeg', `${this.turnStatus.currentDeg}deg`)
      document.documentElement.style.setProperty('--rollBackDeg', `${this.turnStatus.rollBackDeg}`)
    }
  },
  beforeMount () {
    this.config = this.getDefaultConfig()
    // this.gifts = this.getDefaultGfits()
  },
  mounted () {
    this.buideTurnTable()
  }
}
</script>

<style lang="scss">
  //圖形樣式變數
  $turnTableSize: var(--turnTableSize);
  $buttonSize: var(--buttonSize);
  $buttonColor: var(--buttonColor);
  $buttonFontSize: var(--buttonFontSize);
  $arrowWidth: var(--arrowWidth);
  $arrowHeight: var(--arrowHeight);
  //轉動執行用變數
  $runTime: var(--runTime);
  $targetDeg: var(--targetDeg);
  $currentDeg: var(--currentDeg);
  $rollBackDeg: var(--rollBackDeg);
  $animitionState: var(--animitionState);

  @mixin positionCenter($args:null) {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%) $args;
  }

  * {
    font-family: Microsoft JhengHei;
  }

  html {
    font-size: 14px;
  }

  .alert {
    font-size: 20px;
    &--show {
      z-index: 1;
      opacity: 0.7;
      transition: opacity 0.5s linear;
    }
  }

  .buttonArea {
    position: fixed;
    display: inline-block;
    right: 3vh;
    bottom: 3vh;
  }

  .turnTable {
    @include positionCenter();
    width: $turnTableSize;
    height: $turnTableSize;
    border-radius: 50%;
    box-shadow: 0px 2px 10px rgba(0, 0, 0, 0.5);
    user-select: none;
    &__body {
      @include positionCenter();
      font-size: 0px;
    }
    &__canvas {
      transform: rotate($targetDeg);
    }
    &__canvas--autoTrun {
      animation: AutoTrun $runTime forwards cubic-bezier(0.1, 0, 0, $rollBackDeg);
    }
    &__canvas--manualTrun {
      animation: ManualTrun $runTime infinite linear forwards;
      animation-play-state: $animitionState;
    }
    &__button {
      @include positionCenter();
      padding: 10px;
      border-radius: 50%;
      background-color: #fff;
      box-shadow: 0px 1px 5px rgba(0, 0, 0, 0.5);
      color: #fff;
      cursor: pointer;
    }
    &__button__content {
      width: $buttonSize;
      height: $buttonSize;
      border-radius: 50%;
      background-color: $buttonColor;
    }
    &__button__text {
      @include positionCenter();
      color: #2b1414;
      font-weight: 900;
      font-size: $buttonFontSize;
    }
    &__arrow {
      @include positionCenter(rotate(270deg));
      left: 100%;
      transform: translate(-113%, -107%) rotate(180deg);

      border: $arrowHeight solid transparent;
      border-top: $arrowWidth solid $buttonColor;
    }
  }

  @keyframes AutoTrun {
    from {
      transform: rotate($currentDeg);
    }
    to {
      transform: rotate($targetDeg);
    }
  }

  @keyframes ManualTrun {
    from {
      transform: rotate(0deg);
    }
    to {
      transform: rotate(7200deg);
    }
  }
</style>
