<template>
  <div class="container">
    <ImageReader v-model="text"/>
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
    };
  },
};
</script>