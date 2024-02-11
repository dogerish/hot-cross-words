<template>
  <div class="home">
    <CrosswordPuzzle :puzzle="puzzle" />
  </div>
</template>

<script>
// @ is an alias to /src
import CrosswordPuzzle from "@/components/CrosswordPuzzle.vue";
import puzzle from "@/puzzles/hot-cross-words.json";

export default {
  name: "HomeView",
  data() {
    return { puzzle };
  },
  components: {
    CrosswordPuzzle,
  },
};
</script>
