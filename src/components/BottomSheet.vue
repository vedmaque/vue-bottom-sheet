<template>
  <div
    class="bs-wrapper"
    ref="bwrapper"
    :style="wrapperStyle"
    :class="{
      _locked: isMoving
    }"
  >
    <div class="bs-over" ref="bover">
      <slot name="over"></slot>
    </div>

    <div class="bs-under" ref="bunder">
      <slot name="under"></slot>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    topOffset: {
      type: Number,
      required: false,
      default: 70 // минимальный отступ от верхней части сайта
    }
  },
  data: () => ({
    top: null, // текущее значение translateY элемента
    lastTop: null, // последнее значение translateY при движении блока (для вычислений)
    boverTop: null, // значение translateY для свёрнутого состояния, вычисляется один раз
    boverHeight: null, // высота верхнего видимого блока, вычисляется один раз
    maxHeight: null, // максимальная высота всего блока (равна или высоте блока или доступному месту)
    initialTouchY: null, // начальная позиция тапа (нужна для расчётов)
    isMoving: false, // флажок, двигает юзер или не двигает блок
    isOpened: false // флажок, в каком состоянии находится блок, полностью открыт или нет
  }),
  computed: {
    wrapperStyle() {
      // в начальном положении высота автоматом, чтобы можно было её посчитать
      // и скрываем блок за пределы экрана
      const { top, maxHeight } = this
      return {
        height: maxHeight === null ? 'auto' : `${maxHeight}px`,
        transform: top === null ? 'translateY(100%)' : `translateY(${top}px)`
      }
    }
  },
  mounted() {
    const bwrapperHeight = this.$refs.bwrapper.clientHeight
    const boverHeight = this.$refs.bover.clientHeight
    const availableHeight = window.innerHeight - this.topOffset
    const maxHeight =
      availableHeight < bwrapperHeight ? availableHeight : bwrapperHeight
    const boverTop = maxHeight - boverHeight

    this.maxHeight = maxHeight
    this.boverHeight = boverHeight
    this.boverTop = boverTop
    this.top = boverTop

    this.$refs.bwrapper.addEventListener('touchstart', this.onTouchStart)
    this.$refs.bwrapper.addEventListener('touchmove', this.onTouchMove)
    this.$refs.bwrapper.addEventListener('touchend', this.onTouchEnd)

    this.$refs.bwrapper.addEventListener('mousedown', this.onTouchStart)
    this.$refs.bwrapper.addEventListener('mousemove', this.onTouchMove)
    this.$refs.bwrapper.addEventListener('mouseup', this.onTouchEnd)
  },
  beforeDestroy() {
    this.$refs.bwrapper.removeEventListener('touchstart', this.onTouchStart)
    this.$refs.bwrapper.removeEventListener('touchmove', this.onTouchMove)
    this.$refs.bwrapper.removeEventListener('touchend', this.onTouchEnd)

    this.$refs.bwrapper.removeEventListener('mousedown', this.onTouchStart)
    this.$refs.bwrapper.removeEventListener('mousemove', this.onTouchMove)
    this.$refs.bwrapper.removeEventListener('mouseup', this.onTouchEnd)

    this.$refs.bwrapper.removeEventListener(
      'transitionend',
      this.onLastTransitionEnd
    )
  },
  methods: {
    onTouchStart(e) {
      const isMouse = e instanceof MouseEvent
      const touchY = isMouse ? e.clientY : e.changedTouches[0].clientY
      this.isMoving = true
      this.initialTouchY = touchY
      this.lastTop = this.top
    },
    onTouchMove(e) {
      // убираем просто mouse hover
      if (!this.isMoving) {
        return
      }
      const isMouse = e instanceof MouseEvent
      // если там есть что скроллить, то пусть скроллится
      const touchY = isMouse ? e.clientY : e.changedTouches[0].clientY
      const diff = this.initialTouchY - touchY
      // убираем рефреш хрома при свайпе вниз, когда скроллить некуда
      if (diff < 0 && !this.$refs.bwrapper.scrollTop) {
        e.preventDefault()
      }
      // если есть куда скроллить и юзер листает вниз, то начинаем двигать блок вниз только лишь тогда, когда доскроллил
      if (this.$refs.bwrapper.scrollTop && diff < 0) {
        this.initialTouchY = e.changedTouches[0].clientY
        return
      }
      // если листает вверх и блок закрыт, тогда убираем скролл, иначе он и вверх едет и скроллится одновременно, что безумно
      if (diff > 0 && !this.isOpened) {
        e.preventDefault()
      }
      // если открыт и листает вверх, то просто скроллим
      if (diff > 0 && this.isOpened) {
        this.initialTouchY = e.changedTouches[0].clientY
        return
      }

      this.top = this.lastTop - diff
    },
    onTouchEnd(e) {
      const isMouse = e instanceof MouseEvent
      const touchY = isMouse ? e.clientY : e.changedTouches[0].clientY
      this.isMoving = false
      const diff = this.initialTouchY - touchY
      // если было движение вниз на расстоение в два раза больше блока ставки, то закрываем
      if (Math.abs(diff) > this.boverHeight / 3) {
        if (diff <= 0) {
          // если было открытым, то сворачиваем до элемента ставки
          if (this.isOpened) {
            this.isOpened = false
            this.top = this.boverTop
          } else {
            this.top = null
            this.$refs.bwrapper.addEventListener(
              'transitionend',
              this.onLastTransitionEnd
            )
          }
        }
        if (diff > 0) {
          this.top = 0
          this.isOpened = true
        }
      } else {
        this.top = this.isOpened ? 0 : this.boverTop
      }
    },
    onLastTransitionEnd() {
      this.$emit('close')
    },
    externalClose() {
      this.top = null
      this.$refs.bwrapper.addEventListener(
        'transitionend',
        this.onLastTransitionEnd
      )
    }
  }
}
</script>

<style lang="scss" scoped>
.bs-wrapper {
  position: fixed;
  width: 100%;
  left: 0;
  bottom: 0;
  overflow-y: auto;
  transform: translateY(100%);
  box-shadow: 0px 0px 10px #ccc;
  border-radius: 6px;
  box-sizing: border-box;
  transition: transform 0.3s cubic-bezier(0.4, 0, 0.2, 1);

  &._locked {
    transition: none;
  }
}
</style>
