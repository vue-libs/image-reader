<template>
  <div>
    <ImageReader v-model="text" :config="config" />
    This is text:     {{ text }}
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
        src: "",
      },
    };
  },
};
</script>