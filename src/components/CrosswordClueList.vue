<template>
  <div class="clues-section">
    <h2>{{ across ? "Across" : "Down" }}</h2>
    <ol class="clues-list">
      <CrosswordClue :word="word" :key="i" v-for="(word, i) in words" />
    </ol>
  </div>
</template>

<script>
import CrosswordClue from "@/components/CrosswordClue.vue";

export default {
  name: "CrosswordClueList",
  props: ["words", "across"],
  components: { CrosswordClue },
};
</script>

<style>
.clues-section {
  text-align: left;
}
</style>
