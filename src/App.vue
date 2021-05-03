<template>
  <div class="container-fluid">
    <div class="header">
      <nav class="navbar navbar-light bg-light">
        <a
          class="navbar-brand"
          href="https://github.com/vue-libs/image-reader"
          target="_blank"
        >
          <img
            src="https://avatars.githubusercontent.com/u/37013844?v=4"
            width="30"
            height="30"
            class="d-inline-block align-top"
            alt=""
          />
          Image Reader DEMO : Made using tesseract JS in VUE
        </a>
      </nav>
    </div>
    <b-card>
      <div>
        <ImageReader v-model="text" :config="config" />
        <b-card
          bg-variant="light"
          header="This is the converted text"
          class="text-center"
        >
          <b-card-text>{{ text }}</b-card-text>
        </b-card>
      </div>
    </b-card>
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
        multiLanguage: true,
      },
    };
  },
};
</script>