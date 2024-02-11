<template>
  <CrosswordTile
    :x="getX(i)"
    :y="getY(i)"
    :letter="letter"
    :registerTile="registerTile"
    :focused="focused"
    :stealFocus="stealFocus"
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
    "registerWord",
    "getTile",
    "focused",
    "stealFocus",
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
    getX(i) {
      return this.x + this.across * i;
    },
    getY(i) {
      return this.y + !this.across * i;
    },
    registerTile(tile, i) {
      if (i === undefined) i = this.tiles.indexOf(null);
      this.tiles[i] = tile;
    },
  },
  created() {
    this.registerWord(this);
  },
};
</script>
