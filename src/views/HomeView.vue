<template>
  <div class="home">
    <CrosswordPuzzle />
  </div>
</template>

<script>
// @ is an alias to /src
import CrosswordPuzzle from "@/components/CrosswordPuzzle.vue";

export default {
  name: "HomeView",
  components: {
    CrosswordPuzzle,
  },
};
</script>
