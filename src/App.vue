<template>
  <div id="app">
    <img alt="Vue logo" src="./assets/logo.png" />

    <p>
      BottomSheet
    </p>
    <button @click="toggleBottomSheet" class="actionButton">Toggle</button>
    <bottom-sheet
      v-if="isBottomSheetShown"
      @close="closeBottomSheet"
      ref="bottomSheet"
    >
      <template #over>
        <div class="bbet">
          <p>Over</p>
          <p>
            Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do
            eiusmod tempor incididunt ut labore et dolore magna aliqua. Enim
            lobortis scelerisque fermentum dui.
          </p>
        </div>
      </template>
      <template #under>
        <div class="bsettings">
          <p>Under</p>
          <p>
            Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do
            eiusmod tempor incididunt ut labore et dolore magna aliqua. Enim
            lobortis scelerisque fermentum dui. Sed turpis tincidunt id aliquet
            risus feugiat. Purus ut faucibus pulvinar elementum integer enim
            neque. Adipiscing tristique risus nec feugiat in fermentum. Amet
            dictum sit amet justo donec enim diam vulputate ut. Consectetur a
            erat nam at lectus urna. Aliquet enim tortor at auctor urna nunc id.
            Ullamcorper malesuada proin libero nunc. Arcu cursus euismod quis
            viverra. Enim diam vulputate ut pharetra sit amet aliquam id. At in
            tellus integer feugiat scelerisque varius.
          </p>
          <p>
            Non nisi est sit amet facilisis magna etiam. Cursus risus at
            ultrices mi tempus imperdiet. Eu scelerisque felis imperdiet proin.
            Nibh venenatis cras sed felis eget velit aliquet sagittis id. Mi
            eget mauris pharetra et ultrices neque ornare. Volutpat est velit
            egestas dui id. Purus gravida quis blandit turpis cursus in hac.
            Tellus at urna condimentum mattis pellentesque id nibh tortor id.
            Purus sit amet luctus venenatis lectus magna fringilla.
          </p>
        </div>
      </template>
    </bottom-sheet>
  </div>
</template>

<script>
import BottomSheet from '@/components/BottomSheet.vue'
export default {
  name: 'app',
  components: {
    BottomSheet
  },
  data: () => ({
    isBottomSheetShown: false
  }),
  methods: {
    toggleBottomSheet() {
      const newValue = !this.isBottomSheetShown
      if (!newValue) {
        this.$refs.bottomSheet.externalClose()
      } else {
        this.isBottomSheetShown = newValue
      }
    },
    closeBottomSheet() {
      this.isBottomSheetShown = false
    }
  }
}
</script>

<style lang="scss">
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
.actionButton {
  background-color: #42b983;
  padding: 8px 16px;
  border: 0;
  color: white;
  border-radius: 4px;
  &:active {
    background-color: #36966a;
  }
}
.bbet {
  padding: 8px;
  background-color: #efebe1;
  display: flex;
  flex-direction: column;
}
.bsettings {
  padding: 8px;
  background-color: wheat;
  display: flex;
  flex-direction: column;
}
</style>
