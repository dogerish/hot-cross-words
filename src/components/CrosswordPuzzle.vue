<template>
  <h1>Crossword Puzzle: {{ puzzle.title }}</h1>
  <div
    id="puzzle"
    :style="`--tile-unit: 3rem; --puzzle-width: ${puzzleSize[0]}; --puzzle-height: ${puzzleSize[1]}`"
  >
    <div id="tiles">
      <CrosswordWord
        :x="word[0]"
        :y="word[1]"
        :across="word[2] === 'across'"
        :word="word[3]"
        :puzzleSize="puzzleSize"
        :registerWord="registerWord"
        :getTile="getTile"
        :stealFocus="stealFocus"
        :focusNext="focusNext"
        :focusTo="focusTo"
        :backspace="backspace"
        :key="i"
        v-for="(word, i) in puzzle.words"
      />
    </div>
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
    // calculate puzzle dimensions
    let puzzleSize = [0, 0];
    for (let word of this.puzzle.words) {
      if (word[2] === "across") {
        if (word[0] + word[3].length > puzzleSize[0])
          puzzleSize[0] = word[0] + word[3].length;
      } else {
        if (word[1] + word[3].length > puzzleSize[1])
          puzzleSize[1] = word[1] + word[3].length;
      }
    }
    return {
      focused: null,
      focusedWord: null,
      across: [],
      down: [],
      puzzleSize,
    };
  },
  methods: {
    registerWord(word) {
      if (word.across) this.across.push(word);
      else this.down.push(word);
    },
    getTile(x, y) {
      // get the tile found at (x, y)
      for (let word of this.across) {
        if (y !== word.y || x < word.x || x >= word.x + word.word.length)
          continue;
        return word.tiles[x - word.x];
      }
      for (let word of this.down) {
        if (x !== word.x || y < word.y || y >= word.y + word.word.length)
          continue;
        return word.tiles[y - word.y];
      }
    },
    stealFocus(tile) {
      // If already focused, give opposite direction precedence. Otherwise give
      // same direction precedence.
      let direcPrece =
        !this.focusedWord || this.focusedWord.across
          ? [this.across, this.down]
          : [this.down, this.across];
      if (this.focused === tile) direcPrece.reverse();
      // Give focus to tile and its word or null.
      this.focused = tile;
      if (this.focusedWord) this.focusedWord.unfocusWord();
      this.focusedWord = null;
      if (this.focused === null) return;
      this.focusedWord =
        direcPrece[0].find((word) => word.tiles.includes(tile)) ||
        direcPrece[1].find((word) => word.tiles.includes(tile));
      this.focusedWord.focusWord();
    },
    getNextTile(sign) {
      // return following or previous tile (sign = +1 or -1, respectively)
      let i = this.focusedWord.getTileI(this.focused.x, this.focused.y) + sign;
      // return next tile if it is in the same word
      if (i < this.focusedWord.word.length && i >= 0)
        return this.focusedWord.tiles[i];
      // if last letter of word, move to next word
      let wordlist = this.focusedWord.across ? this.across : this.down;
      let idx = wordlist.indexOf(this.focusedWord) + sign;
      // if last word of this direction, move to other direction
      if (idx === wordlist.length || idx === -1) {
        wordlist = this.focusedWord.across ? this.down : this.across;
        idx = sign < 0 ? wordlist.length - 1 : 0;
      }
      // focus first tile of next word
      i = sign < 0 ? wordlist[idx].tiles.length - 1 : 0;
      return wordlist[idx].tiles[i];
    },
    focusNext(sign = +1) {
      this.getNextTile(sign).focus();
    },
    focusTo(x, y) {
      // Try to give focus to tile at (x, y)
      let tile = this.getTile(x, y);
      if (tile) tile.focus();
    },
    backspace() {
      let tile = this.getNextTile(-1);
      tile.value = "";
      tile.focus();
    },
  },
};
</script>

<style>
#puzzle {
  position: relative;
  width: calc(var(--puzzle-width) * var(--tile-unit) - 1px);
  height: calc(var(--puzzle-width) * var(--tile-unit) - 1px);
  background: gray;
  margin: auto;
  outline: 1px solid black;
  overflow: hidden;
}

@keyframes spin-puzzle {
  0% {
    transform: rotate(240deg);
  }
  100% {
    transform: rotate(0deg);
  }
}

#tiles {
  width: 100%;
  height: 100%;
  animation: spin-puzzle 0.5s ease-out;
  animation-fill-mode: both;
}
</style>
