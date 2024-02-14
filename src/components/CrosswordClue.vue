<template>
  <li :value="num" class="crossword-clue">
    {{ clue }}
  </li>
</template>

<script>
export default {
  name: "CrosswordClue",
  props: ["num", "clue"],
};
</script>
