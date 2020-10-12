<template>
  <div id="app">
    <ImageReader v-model="text" :config="config" />
    This is text:     {{ text }}
  </div>
</template>

<script>
/* eslint-disable */
import { createWorker, PSM, OEM } from "tesseract.js";
const worker = createWorker({
  logger: (m) => console.log(m),
});

import ImageReader from "./components/ImageReader";
export default {
  name: "App",
  components: { ImageReader },
  data() {
    return {
      text: "",
      config: {
        src: "",
      },
    };
  },
};
</script>

<style>
</style>
