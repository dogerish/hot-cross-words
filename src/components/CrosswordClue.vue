<template>
  <li
    :value="word.num"
    class="crossword-clue"
    :class="{ 'clue-focus': word.isfocused }"
    :key="i"
    @click="word.focus()"
  >
    {{ word.clue }}
  </li>
</template>

<script>
export default {
  name: "CrosswordClue",
  props: ["word"],
};
</script>

<style>
.crossword-clue.clue-focus {
  background: lightblue;
}
</style>
