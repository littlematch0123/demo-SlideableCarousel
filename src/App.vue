<template>
  <section :class="$style.wrap">
    <ul
      :class="{
        [$style.panelBox]:true,
        [$style.transitionByPanelBox]:needTransition
      }"
      :style="{
        width: `calc(100% * ${this.len})`,
        transform: transformByPanelBox
      }"
      @touchstart="startDrag"
      @touchmove="onDrag"
      @touchend="stopDrag"
    >
      <li
        v-for="(el,i) in imgDatas" :key="i"
        :style="{width:`calc(100% / ${len})`}"
        :class="[
          {[$style['panel-active']]:(i === index)},
          $style['panel']
        ]"
        @click="$router.push(el.url)"
      >
        <img
          :class="$style['panel-img']"
          :src="el.src" :alt="el.name"
        />
      </li>
    </ul>
    <ul :class="$style.dotBox">
      <li
        v-for="(el,i) in dotLen" :key="i"
        :class="[
          {[$style['dot-active']]:(i === dotIndex)} ,
          $style.dot
        ]"
      ></li>
    </ul>
  </section>
</template>

<script>
const imgDatas = [
  {
    id: '1',
    name: 'closure',
    src: require('./img/closure_big.png'),
    url: '/closure'
  },
  {
    id: '2',
    name: 'transition',
    src: require('./img/transition_big.png'),
    url: '/transition'
  },
  {
    id: '3',
    name: 'regexp',
    src: require('./img/regexp_big.jpg'),
    url: '/regexp'
  }
]

export default {
  mounted() {
    // 复制一次imgDatas，用于无缝滚动
    this.imgDatas = this.imgDatas.concat(this.imgDatas)
    // 设置轮播图的长度为图片的个数
    this.len = this.imgDatas.length
    // 开启自动轮播
    this.autoSwipe()
    // 设置轮播图的tranform属性
    this.$_setTransformByPanelBox_g()
  },
  data() {
    return {
      imgDatas,
      index: 0,
      len: 0,
      needTransition: true,
      slideDis: 0,
      transformByPanelBox: 'translate3d(0,0,0)',
      timer: 0,
      timerDuration: 3000
    }
  },
  computed: {
    // 显示点的数量为复制后的图像数量的一半
    dotLen() {
      return this.len / 2
    },
    // 显示点的索引值为图像索引值 % 显示点的数量
    dotIndex() {
      return this.index % this.dotLen
    }
  },

  methods: {
    /* 检测滑动 */
    $_checkSwipe(changeX, duration) {
      // 最小滑动距离为10px
      const THRESHOLD = 10
      // 最大滑动时间为300ms
      const SHORTESTTIME = 300
      // 距离大于10，且时间小于300ms，才算做一次滑动
      return Boolean(Math.abs(changeX) > THRESHOLD && duration < SHORTESTTIME)
    },

    /* 设置index */
    $_setIndex_g() {
      // 当index为原始列的第0个时， 将其设置为复制列的第0个
      if (this.index === 0) {
        this.index = this.len / 2
      }
      // 当index为复制列的最后一个时，将其设置为原始列的最后一个
      if (this.index === this.len - 1) {
        this.index = this.len / 2 - 1
      }
    },

    /* 设置元素的transform属性 */
    $_setTransformByPanelBox_g() {
      // 如果过渡被开启，表示已经结束滑动
      if (this.needTransition) {
        this.transformByPanelBox = `translate3d(calc(-${this.index}*100%/${
          this.len
        }) ,0,0)`
        // 否则，正在滑动
      } else {
        this.transformByPanelBox = `translate3d(calc(${-this.index /
          this.len *
          100 +
          '%'} + ${this.slideDis}px),0,0)`
      }
    },

    /* 自动轮播 */
    autoSwipe() {
      let autoSetTimeoutFn = () => {
        // 如果是复制列的最后一个，则将其设置为原始列的最后一个。并取消transition，设置transform
        if (this.index === this.len - 1) {
          this.index = this.len / 2 - 1
          this.needTransition = false
          this.$_setTransformByPanelBox_g()
        }
        // 设置index的自增
        this.index = (this.index + 1) % this.len
        // 设置50ms的延时，为元素渲染留出时间
        setTimeout(() => {
          this.needTransition = true
          this.$_setTransformByPanelBox_g()
        }, 50)
        this.timer = setTimeout(autoSetTimeoutFn, this.timerDuration)
      }
      this.timer = setTimeout(autoSetTimeoutFn, this.timerDuration)
    },

    /* 开始滑动 */
    startDrag(e) {
      // 记录手指按下时的时刻及位置
      this.startTime = new Date().getTime()
      this.startX = e.changedTouches[0].pageX
      // 清除定时器，并将其值置0
      clearInterval(this.timer)
      this.timer = 0
      // 设置index
      this.$_setIndex_g()
      // 关闭过渡
      this.needTransition = false
    },

    /* 滑动过程 */
    onDrag(e) {
      // 记录手指的滑动距离
      this.slideDis = e.changedTouches[0].pageX - this.startX
      // 设置元素的transform属性
      this.$_setTransformByPanelBox_g()
    },

    /* 滑动结束 */
    stopDrag(e) {
      // 记录手指的滑动时间及滑动距离
      const duration = new Date().getTime() - this.startTime
      const moveX = e.changedTouches[0].pageX - this.startX
      // 检测是否是一次成功的滑动
      if (this.$_checkSwipe(moveX, duration)) {
        // 检测方向，如果moveX>0，表示向右滑动，则index减小
        if (moveX > 0) {
          this.index--
          // 否则，表示向左滑动，index增大
        } else {
          this.index++
        }
      }
      // 开启过渡
      this.needTransition = true
      // 设置元素的transform属性[注意：transform属性的设置依赖于是否开启过渡]
      this.$_setTransformByPanelBox_g()
      // 开启自动轮播
      this.autoSwipe()
    }
  }
}
</script>
<style module lang="postcss">
body{
  margin: 0;
}
ul{
  margin: 0;
  padding: 0;
  list-style: none;
}
.wrap {
  position: relative;
  overflow: hidden;
  height: 140px;
}
.panelBox {
  display: flex;
  height: 100%;
}
.transitionByPanelBox {
  transition: transform 0.5s;
}
.panel {
  height: 100%;
}
.panel-img {
  width: 100%;
  height: 100%;
}
.dotBox {
  position: absolute;
  bottom: 10px;
  left: 40%;
  right: 40%;
  display: flex;
  justify-content: space-between;
}
.dot {
  display: inline-block;
  width: 10px;
  height: 10px;
  border-radius: 50%;
  background: rgba(255, 255, 255, 0.6);
}
.dot-active {
  background: #fff;
}
</style>
