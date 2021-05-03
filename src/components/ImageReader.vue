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
      <div class="col-2">
        <div class="row">
          <div class="col-6">
            <b-button
              :class="isReadable ? 'btn-success' : 'btn-danger'"
              :disabled="!isReadable"
              @click="clearSelections"
            >
              <b-icon icon="x-square-fill"> </b-icon
            ></b-button>
          </div>
          <div class="col-6">
            <b-button
              :class="isReadable ? 'btn-success' : 'btn-danger'"
              @click="recognize"
              :disabled="!isReadable"
            >
              Read
            </b-button>
          </div>
        </div>
        <div class="row">
          <b-form-select
            v-model="selectedLanguage"
            :options="languageOptions"
          ></b-form-select>
        </div>
      </div>
    </div>

    <div v-if="showStats">
      <div class="progress-text">
         <p>{{workerStatus}} || {{workerStatusPercentage.toFixed(2)}} %</p>
      </div>
      <b-progress :value="workerStatusPercentage" height="5px" :max="100" animated>
      </b-progress>
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
import languages from "./languages.json";
let THAT = null;

const worker = createWorker({
  logger: (m) => {
    THAT.getWorkerLogs(m);
  },
});

export default {
  name: "ImageReader",
  data() {
    return {
      workerStatusPercentage: 0,
      workerStatus:"",
      showStats:false,
      languages: languages,
      languageOptions: [{ value: null, text: "Select language" }],
      selectedLanguage: null,
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
          multiLanguage: true,
        };
      },
    },
  },
  mounted() {
    this.imageURL = this.config.src;
    for (const lang in languages) {
      this.languageOptions.push({ value: lang, text: languages[lang] });
    }

    // Initialize global THAT with current instance of this
    THAT = this;
  },
  computed: {
    isReadable() {
      return this.selectedImageBase64.length > 0 || this.imageURL.length > 0;
    },
    getDynamicSource() {
      if (this.selectedImageBase64.length > 0 && this.config.browse) {
        return this.selectedImageBase64;
      } else if (this.imageURL.length > 0 && this.config.src) {
        return this.imageURL;
      }
      return "https://tesseract.projectnaptha.com/img/eng_bw.png";
    },
  },
  methods: {
    getWorkerLogs(e) {
      this.workerStatusPercentage = e.progress * 100;
      this.workerStatus = e.status;
    },
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
      this.showStats = false;
      this.$emit("input", "");
      this.workerStatusPercentage = 0;
      this.workerStatus = "";
    },

    recognize: async function () {
      this.showStats = true;
      await worker.load();

      let language = "eng";
      if (this.config.multiLanguage) {
        language = this.selectedLanguage;
      }

      await worker.loadLanguage(language);
      await worker.initialize(language, OEM.LSTM_ONLY);

      await worker.setParameters({
        tessedit_pageseg_mode: PSM.SINGLE_BLOCK,
      });

      let img = "https://tesseract.projectnaptha.com/img/eng_bw.png"; // Default Image

      if (this.selectedImageBase64.length > 0 && this.config.browse) {
        img = document.getElementById("text-img");
      } else if (this.imageURL.length > 0 && this.config.src) {
        img = this.imageURL;
      }

      const {
        data: { text },
      } = await worker.recognize(img);

      this.$emit("input", text);
    },
  },
};
</script>
<style scoped>
.progress-text{
  text-align: center;
}
</style>
