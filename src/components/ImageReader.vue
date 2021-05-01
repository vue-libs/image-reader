<template>
  <div>
    <div class="row">
      <div class="col-10">
        <b-form-file
          v-model="imageFile"
          placeholder="Choose a file or drop it here..."
          drop-placeholder="Drop file here..."
          accept=".jpg, .png, .gif"
        ></b-form-file>

        <b-form-input
          type="url"
          v-model="imageURL"
          placeholder="Enter URL here"
        ></b-form-input>
      </div>
      <div class="col-1">
        <b-button
          :class="isReadable ? 'btn-success' : 'btn-danger'"
          :disabled="!isReadable"
          @click="clearSelections"
        >
          <b-icon icon="x-square-fill"> </b-icon
        ></b-button>
      </div>
      <div class="col-1">
        <b-button
          :class="isReadable ? 'btn-success' : 'btn-danger'"
          @click="recognize"
          :disabled="!isReadable"
        >
          Read
        </b-button>
      </div>
    </div>

    <div>
      <b-img
        v-if="isReadable && config.showImage"
        v-bind="mainProps"
        center
        id="text-img"
        :src="getDynamicSource"
        alt="Image rendering ..."
        crossorigin="anonymous"
      ></b-img>
    </div>
  </div>
</template>

<script>
import { createWorker, PSM, OEM } from "tesseract.js";

const worker = createWorker({
  logger: (m) => console.log(m),
});

export default {
  name: "ImageReader",
  data() {
    return {
      imageFile: null,
      selectedImageBase64: "",
      imageURL: "",
      mainProps: {
        blank: false,
        blankColor: "#777",
        width: "600",
        height: "300",
        class: "m1",
      },
    };
  },
  watch: {
    imageFile(file) {
      if (file) this.getImageData(file);
    },
  },
  props: {
    value: {
      type: String,
      required: true,
    },
    config: {
      type: Object,
      default: function () {
        return {
          browse: true,
          src: "https://tesseract.projectnaptha.com/img/eng_bw.png",
          showImage: true,
        };
      },
    },
  },
  mounted() {
    this.imageURL = this.config.src;
  },
  computed: {
    isReadable() {
      return this.selectedImageBase64.length > 0 || this.imageURL.length > 0;
    },
    getDynamicSource() {
      if (this.selectedImageBase64 && this.config.browse) {
        return this.selectedImageBase64;
      } else if (this.imageURL && this.config.src) {
        return this.config.src;
      }
      return "https://tesseract.projectnaptha.com/img/eng_bw.png";
    },
  },
  methods: {
    getBase64: function (file) {
      return new Promise((resolve, reject) => {
        const reader = new FileReader();
        reader.readAsDataURL(file);
        reader.onload = () => resolve(reader.result);
        reader.onerror = (error) => reject(error);
      });
    },

    getImageData: async function (file) {
      this.selectedImageBase64 = await this.getBase64(file);
    },

    clearSelections: function () {
      this.imageFile = null;
      this.selectedImageBase64 = "";
      this.imageURL = "";
    },

    recognize: async function () {
      await worker.load();
      await worker.loadLanguage("eng");
      await worker.initialize("eng", OEM.LSTM_ONLY);
      await worker.setParameters({
        tessedit_pageseg_mode: PSM.SINGLE_BLOCK,
      });

      let img = "https://tesseract.projectnaptha.com/img/eng_bw.png"; // Default Image

      if (this.selectedImageBase64 && this.config.browse) {
        img = document.getElementById("text-img");
      } else if (this.imageURL && this.config.src) {
        img = this.config.src;
      }

      const {
        data: { text },
      } = await worker.recognize(img);

      this.$emit("input", text);
    },
  },
};
</script>
