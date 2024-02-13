<template>
  <CrosswordTile
    :num="i === 0 ? num : null"
    :x="getTileX(i)"
    :y="getTileY(i)"
    :letter="letter"
    :registerTile="registerTile"
    :stealFocus="stealFocus"
    :stealWordFocus="stealWordFocus"
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
    "num",
    "x",
    "y",
    "across",
    "word",
    "clue",
    "registerWord",
    "getTile",
    "stealFocus",
    "stealWordFocus",
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
      let tile = this.getTile(this.getTileX(i), this.getTileY(i));
      if (!tile) return true;
      tiles[i] = tile;
      if (i === 0) tile.setNum(this.num);
      return false;
    });
    return { tiles, letters, isfocused: false };
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
    hasTile(x, y) {
      if (x instanceof Object) {
        y = x.y;
        x = x.x;
      }
      // return true if tile at x, y is in this word
      return (
        x >= this.x &&
        x <= this.x + this.across * (this.word.length - 1) &&
        y >= this.y &&
        y <= this.y + !this.across * (this.word.length - 1)
      );
    },
    registerTile(tile, i) {
      if (i === undefined) i = this.tiles.indexOf(null);
      this.tiles[i] = tile;
    },
    focus() {
      this.stealWordFocus(this);
      this.tiles[0].focus();
    },
    focusWord() {
      for (let tile of this.tiles) tile.wordfocus = true;
      this.isfocused = true;
    },
    unfocusWord() {
      for (let tile of this.tiles) tile.wordfocus = false;
      this.isfocused = false;
    },
  },
  created() {
    this.registerWord(this);
  },
};
</script>
