<template>
  <div id="app">
    <img alt="Vue logo" src="./assets/logo.png" />
    <HelloWorld msg="Welcome to Your Vue.js App" />
     <button v-on:click="recognize">recognize</button>
    <img id="text-img" alt="Vue logo" src="./assets/testocr.png">
  </div>
</template>

<script>
import HelloWorld from "./components/HelloWorld.vue";

/* eslint-disable */
import { createWorker, PSM, OEM } from "tesseract.js";
const worker = createWorker({
  logger: (m) => console.log(m),
});

export default {
  name: "App",
  data(){
    return {
      text:''
    }
  },
  components: {
    HelloWorld,
  },
  mounted() {

  },

  methods: {
    recognize: async () => {
      let that = this;
      const img = document.getElementById("text-img");
      await worker.load();
      await worker.loadLanguage("eng");
      await worker.initialize("eng", OEM.LSTM_ONLY);
      await worker.setParameters({
        tessedit_pageseg_mode: PSM.SINGLE_BLOCK,
      });
      const {
        data: { text },
      } = await worker.recognize(img);
      console.log(text);
    },
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
