<script setup>
import { ref } from "vue";
import VueQrcode from "@chenfengyuan/vue-qrcode";
import { toPng } from "html-to-image";

const userInput = ref("");

const downloading = ref(false);

const templateReference = ref(null);

const downloadQRCode = () => {
  if (templateReference.value === null || !userInput.value) {
    return;
  }

  downloading.value = true;

  toPng(templateReference.value, { cacheBust: true })
    .then((dataUrl) => {
      const link = document.createElement("a");
      link.download = "my-qrcode.png";
      link.href = dataUrl;
      link.click();
      downloading.value = false;
    })
    .catch((err) => {
      console.log(err, templateReference.value);
      downloading.value = false;
    });
};

const useLogo = ref(false);
const uploadedImgs = ref([]);
const uploadedImgUrl = ref("");

const handleLogoUpload = ($event) => {
  const input = $event.target;
  uploadedImgs.value = input.files;

  if (uploadedImgs.value.length == 0) {
    return;
  }

  const reader = new FileReader();

  reader.addEventListener(
    "load",
    () => {
      uploadedImgUrl.value = reader.result;
    },
    false
  );

  reader.readAsDataURL(uploadedImgs.value[0]);
};
</script>

<template>
  <header>
    <span>QR Code Generator</span>
  </header>

  <div class="container">
    <div class="left">
      <textarea
        class="user-input-field"
        placeholder="Enter your text here"
        v-model="userInput"
      ></textarea>
    </div>

    <div class="right">
      <template v-if="userInput && userInput.length > 0">
        <div class="qrcode-box">
          <div
            v-if="userInput && userInput.length > 0"
            class="qrcode"
            ref="templateReference"
          >
            <vue-qrcode
              :value="userInput"
              :options="{
                width: 200,
              }"
            ></vue-qrcode>
            <img
              v-if="useLogo && uploadedImgUrl && uploadedImgUrl.length != 0"
              class="qrcode__image"
              :src="uploadedImgUrl"
              alt="business logo"
            />
          </div>
        </div>

        <div>
          <input type="checkbox" id="toggle-logo" v-model="useLogo" />
          <label for="toggle-logo"> Add logo to QR code </label>
        </div>

        <input
          v-if="useLogo"
          type="file"
          @change="handleLogoUpload"
          accept="image/*"
        />

        <button @click="downloadQRCode" class="download-button">
          <span v-if="!downloading">download code as png</span>
          <span v-else class="spinner"></span>
        </button>
      </template>

      <div v-else class="placeholder">
        <p>Enter some text on the left to generate a code</p>
      </div>
    </div>
  </div>
</template>
