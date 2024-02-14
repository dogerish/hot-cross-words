<template>
  <div class="clues-section">
    <h2>{{ across ? "Across" : "Down" }}</h2>
    <ol class="clues-list">
      <CrosswordClue
        :num="word.num"
        :clue="word.clue"
        :class="{ 'clue-focus': wordIsFocused(word.num) }"
        @click="focusToWord(word.num, across)"
        :key="i"
        v-for="(word, i) in words"
      />
    </ol>
  </div>
</template>

<script>
import CrosswordClue from "@/components/CrosswordClue.vue";

export default {
  name: "CrosswordClueList",
  props: ["words", "focusedWord", "focusToWord", "across"],
  components: { CrosswordClue },
  methods: {
    wordIsFocused(num) {
      return (
        this.focusedWord?.num === num &&
        this.focusedWord?.across === this.across
      );
    },
  },
};
</script>

<style>
.clues-section {
  text-align: left;
}

.clue-focus {
  background: var(--subject-2);
}
</style>
