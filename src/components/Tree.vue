<script setup>
import { ref, watch, onMounted } from "vue";
import nlp from "compromise";
// import ParseTree from "nlptoolkit-parsetree";
// import { Tag } from "en-pos";
// let tags = ref([]);

let pos = ref([]);
let words = ref([]);
let chunks = ref([]);
let sentence = ref("The quick brown fox jumps over the lazy dog");
let parsedsen = ref();

function generateSyntaxTree(sentence) {
  // tags.value = new Tag(sentence.split(" "))
  //   .initial() // initial dictionary and pattern based tagging
  //   .smooth().tags; // further context based smoothing
  // console.log(tags.value);
  // console.log(sentence(sentence).json());
  const parsed = nlp(sentence);
  // console.log(parsed.html({}));
  // console.log(parsed.out("tags"));
  console.log(parsed.debug());
  console.log(parsed.json());
  parsedsen = parsed;
  pos = parsed.document[0];
  chunks = parsed.chunks().out("array");
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
  <div class="pointer-events-none fixed right-4 top-4 w-96 text-stone-500">
    {{ parsedsen.json() }}
  </div>
  <div
    class="m-[0.35rem] grid place-items-center rounded-xl py-2 text-center font-mono font-bold text-white"
  >
    S <br />
    <span>
      <span v-for="(item, i) in chunks">
        {{ i > (chunks.length - 1) / 2 ? "\\" : "/" }}
      </span>
    </span>
  </div>

  <div class="flex">
    <div
      v-for="(item, i) in chunks"
      class="m-[0.35rem] grid place-items-center rounded-xl bg-stone-600 py-2 font-mono font-bold text-stone-400"
    >
      <!-- :style="{
        width: words[i] * 10.8 + 'px !important',
      }" -->
      {{ item }}
    </div>
  </div>

  <!-- <div class="relative flex">
    <div class="absolute mt-3 -translate-x-[110%] text-stone-500">
      lib en-pos
    </div>
    <div
      v-for="(item, i) in tags"
      class="m-[0.35rem] grid place-items-center rounded-xl py-2 font-mono font-bold"
      :style="{
        width: words[i] * 10.8 + 'px !important',
      }"
      :class="
        item.substring(0, 1) == 'V'
          ? 'bg-red-900 text-red-400'
          : item.substring(0, 1) == 'N'
          ? 'bg-emerald-900 text-emerald-400'
          : item.substring(0, 1) == 'J'
          ? 'bg-amber-900 text-amber-400'
          : item.substring(0, 1) == 'P'
          ? 'bg-blue-900 text-blue-400'
          : item.substring(0, 1) == 'A'
          ? 'bg-purple-900 text-purple-400'
          : 'bg-gray-600 text-gray-400'
      "
    >
      {{ item.substring(0, 1) }}
    </div>
  </div> -->
  <div class="relative flex whitespace-nowrap">
    <div class="absolute mt-3 -translate-x-[110%] text-stone-500">
      lib compromise
    </div>
    <div
      v-for="(item, i) in pos"
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

  <div class="flex flex-col">
    <input
      ref="input"
      class="inputglow m-2 mb-4 min-w-[31.5rem] rounded-xl border-2 border-stone-500 bg-stone-800 py-2 text-center font-mono text-lg text-stone-300 outline-none duration-300 focus:border-stone-400"
      oninput="this.size = this.value.length + 1"
      v-model="sentence"
    />

    <div
      v-if="
        nlp(sentence).match('#Preposition? #Verb *').out('array').length > 0
      "
      class="mx-2 my-4"
    >
      <span class="font-bold text-red-400">VPs: &nbsp;</span>
      <span
        class="m-1 rounded-lg bg-red-950 p-2 text-red-300"
        v-for="constituwu in nlp(sentence)
          .match('#Preposition? #Verb *')
          .out('array')"
        >{{ constituwu }}</span
      >
    </div>

    <div
      v-if="
        nlp(sentence).match('#Determiner? #Adjective? #Noun').out('array')
          .length > 0
      "
      class="mx-2 my-4"
    >
      <span class="font-bold text-emerald-400">NPs:&nbsp;</span>
      <span
        class="m-1 rounded-lg bg-emerald-950 p-2 text-emerald-300"
        v-for="constituwu in nlp(sentence)
          .match('#Determiner? #Adjective? #Noun')
          .out('array')"
        >{{ constituwu }}</span
      >
    </div>
    <div
      v-if="nlp(sentence).match('#Preposition *').out('array').length > 0"
      class="mx-2 my-4"
    >
      <span class="font-bold text-blue-400">PPs:&nbsp;</span>
      <span
        class="m-1 rounded-lg bg-blue-950 p-2 text-blue-300"
        v-for="constituwu in nlp(sentence).match('#Preposition *').out('array')"
        >{{ constituwu }}</span
      >
    </div>
  </div>
</template>

<style>
.inputglow:focus {
  box-shadow: inset 0 0 0.5rem 0 #78716c, 0 0 0.5rem 0 #78716c;
  text-shadow: 0 0 0.75rem #78716c;
}
</style>
