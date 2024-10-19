<script setup lang="ts">
import kuromoji from "kuromoji";
import { ref, useTemplateRef } from "vue";
import packageJson from "../package.json";

const { promise, resolve, reject } =
  Promise.withResolvers<kuromoji.Tokenizer<kuromoji.IpadicFeatures>>();

const isLoading = ref(true);
kuromoji
  .builder({
    dicPath: `https://cdn.jsdelivr.net/npm/kuromoji@${packageJson.dependencies.kuromoji}/dict`,
  })
  .build((err, tokenizer) => {
    if (err) {
      reject(err);
    } else {
      isLoading.value = false;
      resolve(tokenizer);
    }
  });

const text = ref("");
const output = useTemplateRef<HTMLTextAreaElement>("output");
const tokenize = async () => {
  const tokenizer = await promise;
  const result = tokenizer.tokenize(text.value);

  output.value!.value = JSON.stringify(result, null, 2);
};
</script>

<template>
  <main>
    <textarea ref="input" v-model="text" />
    <textarea ref="output" class="output" readonly />
    <button @click="tokenize" :disabled="isLoading">Kuromoji</button>
  </main>
</template>

<style scoped>
main {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}
textarea {
  width: max(600px, 60vw);
  height: 10rem;
}
.output {
  font-family: monospace;
}
</style>
