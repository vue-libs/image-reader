<template>
  <div class="container">
    <ImageReader v-model="text" :config="config" />
    This is text: {{ text }}
  </div>
</template>

<script>
import ImageReader from "./components/ImageReader";
export default {
  name: "App",
  components: { ImageReader },
  data() {
    return {
      text: "",
      config: {
        browse: true,
        src: "https://tesseract.projectnaptha.com/img/eng_bw.png",
        showImage: true,
      },
    };
  },
};
</script>