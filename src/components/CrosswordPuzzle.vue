<template>
  <div
    id="puzzle"
    :style="`--puzzle-width: ${size[0]}; --puzzle-height: ${size[1]}`"
    :class="{ correct }"
  >
    <div id="fireworks" v-if="correct">
      <FireworkRocket
        :size="`${Math.random() * 3 + 5}px`"
        :color="
          fireworkColors[Math.floor(Math.random() * fireworkColors.length)]
        "
        :delay="`${i * 0.25}s`"
        :launchX="`${Math.random() * 80 + 10}%`"
        :burstY="`${Math.random() * 60 + 25}%`"
        v-for="i in 20"
        :key="i"
      />
    </div>
    <CrosswordWord
      :num="word.num"
      :x="word.x"
      :y="word.y"
      :across="word.across"
      :word="word.word"
      :clue="word.clue"
      :gradeMode="gradeMode"
      :getTile="getTile"
      :stealFocus="stealFocus"
      :stealWordFocus="stealWordFocus"
      :focusNext="focusNext"
      :focusTo="focusTo"
      :backspace="backspace"
      :key="i"
      :ref="(w) => words.push(w)"
      v-for="(word, i) in rawWords"
    />
  </div>
</template>

<script>
import CrosswordWord from "@/components/CrosswordWord.vue";
import FireworkRocket from "@/components/FireworkRocket.vue";

export default {
  name: "CrosswordPuzzle",
  props: ["rawWords", "gradeMode"],
  components: {
    CrosswordWord,
    FireworkRocket,
  },
  data() {
    // calculate puzzle dimensions
    let size = [0, 0];
    for (let word of this.rawWords) {
      let i = 1 - word.across;
      let extent = (word.across ? word.x : word.y) + word.word.length;
      size[i] = Math.max(extent, size[i]);
    }
    return {
      fireworkColors: [
        "red",
        "orange",
        "yellow",
        "limegreen",
        "turquoise",
        "blue",
        "magenta",
        "pink",
        "white",
      ],
      words: [],
      focused: null,
      focusedWord: null,
      size,
    };
  },
  computed: {
    gradeworthy() {
      return (
        this.gradeMode === 0 &&
        this.words.every((word) => word.value.length === word.word.length)
      );
    },
    correct() {
      return this.words.every((word) => word.correct);
    },
  },
  watch: {
    gradeworthy: {
      handler(gradeworthy) {
        if (gradeworthy) this.words.forEach((word) => word.grade());
        else this.words.forEach((word) => word.ungrade());
      },
    },
  },
  methods: {
    getTile(x, y) {
      // get the tile found at (x, y)
      for (let word of this.words)
        if (word.hasTile?.(x, y)) return word.tiles[word.getTileI(x, y)];
    },
    getWord(num, across) {
      return across
        ? this.words.findLast((w) => w.num === num && w.across)
        : this.words.find((w) => w.num === num && !w.across);
    },
    stealFocus(tile) {
      if (tile === null) {
        this.focusedWord?.unfocusWord();
        this.focusedWord = null;
        this.focused = null;
        return;
      }
      // If already focused on this tile, give opposite direction precedence.
      // Otherwise give same direction or down precedence.
      let preferAcross =
        (this.focused === tile) !=
        (this.focusedWord !== null && this.focusedWord.across);
      // Give focus to tile and its word
      this.focused = tile;
      // Don't bother if the focused word has tile and is preferred
      if (
        this.focusedWord !== null &&
        this.focusedWord.hasTile(tile) &&
        preferAcross === this.focusedWord.across
      )
        return;
      // Search backwards if preferring across because the word list is sorted
      this.stealWordFocus(
        preferAcross
          ? this.words.findLast((word) => word.hasTile(tile))
          : this.words.find((word) => word.hasTile(tile))
      );
    },
    stealWordFocus(word) {
      if (this.focusedWord === word) return;
      this.focusedWord?.unfocusWord();
      this.focusedWord = word;
      this.focusedWord?.focusWord();
    },
    getNextTile(sign) {
      // return [word, tile] of the following (sign = +1) or previous (-1) tile
      let i = this.focusedWord.getTileI(this.focused.x, this.focused.y) + sign;
      // return next tile if it is in the same word
      if (i < this.focusedWord.word.length && i >= 0)
        return [this.focusedWord, this.focusedWord.tiles[i]];
      // if last letter of word, move to next word
      let idx = this.words.indexOf(this.focusedWord) + sign;
      // if last word of the wordlist, wrap around to the other side
      if (idx === this.words.length || idx === -1)
        idx -= sign * this.words.length;
      // focus first tile of next word
      i = sign < 0 ? this.words[idx].tiles.length - 1 : 0;
      return [this.words[idx], this.words[idx].tiles[i]];
    },
    focusNext(sign = +1) {
      let [word, tile] = this.getNextTile(sign);
      // manually update the word so that it chooses the right direction
      this.stealWordFocus(word);
      tile.focus();
    },
    focusTo(x, y) {
      // Try to give focus to tile at (x, y)
      let tile = this.getTile(x, y);
      tile?.focus();
    },
    focusToWord(num, across) {
      let word = this.getWord(num, across);
      if (!word) return;
      this.stealWordFocus(word);
      word.tiles[0].focus();
    },
    backspace() {
      let [word, tile] = this.getNextTile(-1);
      tile.value = "";
      this.stealWordFocus(word);
      tile.focus();
    },
  },
};
</script>

<style>
#puzzle {
  --puzzle-center-x: calc((var(--puzzle-width) - 1) / 2);
  --puzzle-center-y: calc((var(--puzzle-height) - 1) / 2);
  --tile-unit: min(90vw / var(--puzzle-width), 70vh / var(--puzzle-height));
  position: relative;
  width: calc(var(--puzzle-width) * var(--tile-unit) - 1px);
  height: calc(var(--puzzle-height) * var(--tile-unit) - 1px);
  background: var(--object-2);
  margin: auto;
  outline: 1px solid var(--object-1);
  overflow: hidden;
}

#puzzle.correct {
  background: var(--object-3);
}
</style>
