<template>
  <h1>Crossword Puzzle: {{ puzzle.title }}</h1>
  <div id="puzzle">
    <CrosswordWord
      :x="word[0]"
      :y="word[1]"
      :across="word[2] == 'across'"
      :word="word[3]"
      :registerWord="registerWord"
      :getTile="getTile"
      :focused="focused"
      :stealFocus="stealFocus"
      :key="i"
      v-for="(word, i) in puzzle.words"
    />
  </div>
</template>

<script>
import CrosswordWord from "@/components/CrosswordWord.vue";

export default {
  name: "CrosswordPuzzle",
  props: ["puzzle"],
  components: {
    CrosswordWord,
  },
  data() {
    return { focused: null, across: [], down: [] };
  },
  methods: {
    registerWord(word) {
      if (word.across) this.across.push(word);
      else this.down.push(word);
    },
    getTile(x, y) {
      for (let word of this.across) {
        if (y != word.y || x >= word.x + word.word.length) continue;
        return word.tiles[x - word.x];
      }
      for (let word of this.down) {
        if (x != word.x || y >= word.y + word.word.length) continue;
        return word.tiles[y - word.y];
      }
    },
    stealFocus(tile) {
      this.focused = tile;
    },
  },
};
</script>

<style>
#puzzle {
  position: relative;
  width: 499px;
  height: 499px;
  background: gray;
  margin: auto;
  outline: 1px solid black;
}

.puzzle-row {
  line-height: 0;
}
</style>
