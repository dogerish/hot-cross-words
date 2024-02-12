<template>
  <CrosswordTile
    :x="getTileX(i)"
    :y="getTileY(i)"
    :letter="letter"
    :puzzleSize="puzzleSize"
    :registerTile="registerTile"
    :stealFocus="stealFocus"
    :focusNext="focusNext"
    :focusTo="focusTo"
    :backspace="backspace"
    :key="i"
    v-for="[i, letter] in letters"
  />
</template>

<script>
import CrosswordTile from "@/components/CrosswordTile.vue";

export default {
  name: "CrosswordWord",
  props: [
    "x",
    "y",
    "across",
    "word",
    "puzzleSize",
    "registerWord",
    "getTile",
    "stealFocus",
    "focusNext",
    "focusTo",
    "backspace",
  ],
  components: {
    CrosswordTile,
  },
  data() {
    let tiles = Array(this.word.length).fill(null);
    // adopt preexisting tiles
    let letters = Array.from(Array.from(this.word).entries()).filter(([i]) => {
      let tile = this.getTile(this.x, this.y + i);
      if (!tile) return true;
      tiles[i] = tile;
      return false;
    });
    return { tiles, letters };
  },
  methods: {
    getTileX(i) {
      return this.x + this.across * i;
    },
    getTileY(i) {
      return this.y + !this.across * i;
    },
    getTileI(x, y) {
      return x - this.x + y - this.y;
    },
    registerTile(tile, i) {
      if (i === undefined) i = this.tiles.indexOf(null);
      this.tiles[i] = tile;
    },
    focusWord() {
      for (let tile of this.tiles) tile.wordfocus = true;
    },
    unfocusWord() {
      for (let tile of this.tiles) tile.wordfocus = false;
    },
  },
  created() {
    this.registerWord(this);
  },
};
</script>
