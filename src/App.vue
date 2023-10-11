<template>
  <main>
    <h1>
      QRCode Scanner
      <span>/ project-n1870</span>
    </h1>
    <QrStream class="scanner-zone" @decode="onDecode">
      <div class="validation-success" v-if="isLoading">
        <p class="scanner-text">
          <span class="scanner-text__loader"></span>
          Lecture en cours...
        </p>
      </div>

      <div class="validation-success" v-if="isValid && !isLoading && result !== null">
        <p class="scanner-text">
          <span class="scanner-text__success">✓</span>
          QRCode validé, redirection en cours...
        </p>
      </div>

      <div class="validation-failure" v-if="!isValid && !isLoading && result !== null">
        <p class="scanner-text">
          <span class="scanner-text__error">×</span>
          Le QRCode n'est pas au format VCARD
        </p>
      </div>
    </QrStream>
  </main>
</template>

<script setup lang="ts">
import { QrStream } from 'vue3-qr-reader';
import { ref } from "vue";

const isValid = ref<boolean>(false);
const isLoading = ref<boolean>(false);
const result = ref(null);


async function onDecode(content: any) {
  isLoading.value = true;
  result.value = content;

  // pretend it's taking really long
  isValid.value = content.toString().includes("VCARD");
  isLoading.value = false;

  // some more delay, so users have time to read the message
  await timeout(1000);

  if (isValid.value) {
    const nameExtractor = /\nN:(.*)/g;
    const emailExtractor = /EMAIL:(.*)/g;
    const internetExtractor = /INTERNET:(.*)/g;
    const companyExtractor = /ORG:(.*)/g;

    let name;
    if (nameExtractor.test(content)) {
      nameExtractor.lastIndex = 0;
      name = nameExtractor.exec(content);
      if (name !== null) {
        name = name[1].replace(";", "+");
      }
    }
    let email;
    if (emailExtractor.test(content)) {
      emailExtractor.lastIndex = 0;
      email = emailExtractor.exec(content)
    } else {
      email = internetExtractor.exec(content)
    }
    if (email !== null) {
      email = email[1];
    }

    let company = "";
    if (companyExtractor.test(content)) {
      companyExtractor.lastIndex = 0
      const companyValue = companyExtractor.exec(content);
      if (companyValue !== null)
        company = companyValue[1].replace(" ", "+");
    }
    const url: string = `https://docs.google.com/forms/d/e/1FAIpQLSd4VzAuuHynddXhR3-qKnyiak3ERkwhmJeOfN2K0VoMaVIxzQ/viewform?usp=pp_url&entry.100140466=${name}&entry.1386068958=${email}&entry.1985612851=${company}`;
    window.open(url, "_blank");

    result.value = null;
    isValid.value = false;
  } else {
    result.value = null;
  }
}

function timeout(ms: number) {
  return new Promise(resolve => {
    window.setTimeout(resolve, ms);
  });
}


</script>
