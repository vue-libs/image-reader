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
      </div>
      <div class="col-2">
        <b-button
          :class="isNotReadable ? 'btn-danger' : 'btn-success'"
          @click="recognize"
          :disabled="isNotReadable"
        >
          Read
        </b-button>
      </div>
    </div>

    <img
      id="text-img"
      height="500px"
      alt="No image selected"
      crossorigin="anonymous"
      :src="selectedImageBase64"
    />
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
    };
  },
  watch: {
    imageFile(file) {
      this.getImageData(file);
    },
  },
  props: {
    value: {
      type: String,
      required: true,
    },
  },
  computed: {
    isNotReadable() {
      return this.selectedImageBase64.length <= 0;
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

    recognize: async function () {
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

      this.$emit("input", text);
    },
  },
};
</script>
