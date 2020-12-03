<template lang="html">
  <input
    @change="onChangeInput"
    type="file"
    name="image"
    accept="image/*"
    capture="environment"
    multiple
  />
</template>

<script>
import { scan } from "../misc/scanner.js";
import { imageDataFromFile } from "../misc/image-data.js";
import CommonAPI from "../mixins/CommonAPI.vue";
import Worker from "../worker/jsqr.js";

export default {
  name: "qrcode-capture",

  mixins: [CommonAPI],

  props: {
    worker: {
      type: Function,
      default: Worker
    }
  },

  methods: {
    onChangeInput(event) {
      const files = [...event.target.files];
      const resultPromises = files.map(this.processFile);
      resultPromises.forEach(this.onDetect);
    },
    inputFile(data) {
      this.$emit("loading", true);
      fetch(data.webPath)
        .then(res => res.blob()) // Gets the response and returns it as a blob
        .then(blob => {
          let files = [blob];
          files = files.map(
            d =>
              new File([blob], "my_image." + data.format, {
                type: "image/" + data.format,
                lastModified: new Date()
              })
          );
          const resultPromises = files.map(this.processFile);
          resultPromises.forEach(this.onDetect);
          this.$emit("loading", false);
        });
    },

    async processFile(file) {
      const imageData = await imageDataFromFile(file);
      const scanResult = await scan(this.worker, imageData);

      return scanResult;
    }
  }
};
</script>
