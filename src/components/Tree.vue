<script setup>
import { ref, watch, onMounted, defineExpose } from "vue";
import nlp from "compromise";

let an = ref([]);
let words = ref([]);
let sentence = ref("The quick brown fox jumps over the lazy dog");

function generateSyntaxTree(sentence) {
  // console.log(sentence(sentence).json());
  const parsed = nlp(sentence);
  // console.log(parsed);
  const root = {
    label: "Sentence",
    children: [],
  };
  console.log(parsed.document);
  an = parsed.document[0];

  // Generate a tree of constituents based on sentence structure
}

generateSyntaxTree(sentence.value);

watch(
  sentence,
  (val) => {
    words.value = val.split(" ");
    for (let i = 0; i < words.value.length; i++) {
      words.value[i] = words.value[i].length;
    }
    generateSyntaxTree(val);
  },
  { immediate: true }
);

const input = ref(null);
defineExpose({ input });
onMounted(() => {
  input.value.size = sentence.value.length + 1;
});
</script>

<template>
  <div
    class="m-[0.35rem] grid place-items-center rounded-xl py-2 font-mono font-bold text-white"
  >
    S
  </div>

  <div class="flex">
    <div
      v-for="(item, i) in an"
      class="m-[0.35rem] grid place-items-center rounded-xl py-2 font-mono font-bold"
      :style="{
        width: words[i] * 10.8 + 'px !important',
      }"
      :class="
        Array.from(item.tags)[0] == 'Verb'
          ? 'bg-red-900 text-red-400'
          : Array.from(item.tags)[0] == 'Noun'
          ? 'bg-emerald-900 text-emerald-400'
          : Array.from(item.tags)[0] == 'Adjective'
          ? 'bg-amber-900 text-amber-400'
          : Array.from(item.tags)[0] == 'Preposition'
          ? 'bg-blue-900 text-blue-400'
          : Array.from(item.tags)[0] == 'Article'
          ? 'bg-purple-900 text-purple-400'
          : 'bg-gray-600 text-gray-400'
      "
    >
      {{ Array.from(item.tags)[0].substring(0, 1) }}
    </div>
  </div>

  <input
    ref="input"
    class="inputglow m-2 min-w-[31.5rem] rounded-xl border-2 border-stone-500 bg-stone-800 py-2 text-center font-mono text-lg text-stone-300 outline-none duration-300 focus:border-stone-400"
    oninput="this.size = this.value.length + 1"
    v-model="sentence"
  />
</template>

<style>
.inputglow:focus {
  box-shadow: inset 0 0 0.5rem 0 #78716c, 0 0 0.5rem 0 #78716c;
  text-shadow: 0 0 0.75rem #78716c;
}
</style>
