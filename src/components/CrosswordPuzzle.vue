<template>
  <h1>Crossword Puzzle: {{ puzzle.title }}</h1>
  <div id="crossword-container">
    <div
      id="puzzle"
      class="crossword-item"
      :style="`--puzzle-width: ${size[0]}; --puzzle-height: ${size[1]}`"
    >
      <div id="tiles">
        <CrosswordWord
          :num="word.num"
          :x="word.x"
          :y="word.y"
          :across="word.across"
          :word="word.word"
          :clue="word.clue"
          :registerWord="registerWord"
          :getTile="getTile"
          :stealFocus="stealFocus"
          :stealWordFocus="stealWordFocus"
          :focusNext="focusNext"
          :focusTo="focusTo"
          :backspace="backspace"
          :key="i"
          v-for="(word, i) in words"
        />
      </div>
      <ol id="focused-word-info" v-if="focusedWord !== null">
        <span id="focused-word-direction">
          {{ focusedWord.across ? "Across" : "Down" }}
        </span>
        <div id="focused-word-clue">
          <CrosswordClue :word="focusedWord" />
        </div>
      </ol>
    </div>
    <div id="clue-list" class="crossword-item">
      <div id="clues">
        <CrosswordClueList :words="downWords" :across="false" />
        <CrosswordClueList :words="acrossWords" :across="true" />
      </div>
    </div>
  </div>
</template>

<script>
import CrosswordWord from "@/components/CrosswordWord.vue";
import CrosswordClue from "@/components/CrosswordClue.vue";
import CrosswordClueList from "@/components/CrosswordClueList.vue";

export default {
  name: "CrosswordPuzzle",
  props: ["puzzle"],
  components: {
    CrosswordWord,
    CrosswordClue,
    CrosswordClueList,
  },
  data() {
    let words = this.puzzle.words.map(([x, y, dir, word, clue]) => {
      return { num: 0, x, y, across: dir === "across", word, clue };
    });
    // sort words in order of down before across -> ascending y -> ascending x
    words.sort((a, b) => {
      return (
        (a.across - b.across) * 4 +
        Math.sign(a.y - b.y) * 2 +
        Math.sign(a.x - b.x)
      );
    });
    // derive the down-across border
    words.firstAcross = words.findIndex((word) => word.across);
    // assign numbers to words
    let nextNum = 1;
    words.forEach((word, i) => {
      // if starting tile is already labeled, use that same number
      for (let j = 0; j < i; j++)
        if (words[j].x === word.x && words[j].y === word.y)
          return (word.num = words[j].num);
      // just give it the next number and then increment for the next word
      word.num = nextNum++;
    });
    // calculate puzzle dimensions
    let size = [0, 0];
    for (let word of words) {
      let i = 1 - word.across;
      let extent = (word.across ? word.x : word.y) + word.word.length;
      size[i] = Math.max(extent, size[i]);
    }
    return {
      focused: null,
      focusedWord: null,
      words,
      size,
    };
  },
  computed: {
    downWords() {
      return this.words.slice(0, this.words.firstAcross);
    },
    acrossWords() {
      return this.words.slice(this.words.firstAcross);
    },
  },
  methods: {
    registerWord(word) {
      this.words[word.$.vnode.key] = word;
    },
    getTile(x, y) {
      // get the tile found at (x, y)
      for (let word of this.words)
        if (word.hasTile?.(x, y)) return word.tiles[word.getTileI(x, y)];
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
      if (tile) tile.focus();
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
#crossword-container {
  display: flex;
  flex-flow: row wrap;
  justify-content: center;
  align-items: stretch;
  gap: 10px;
}

#clue-list {
  flex: 1 1 15rem;
}

#puzzle {
  flex: 1 0 auto;
}

#tiles {
  --puzzle-center-x: calc((var(--puzzle-width) - 1) / 2);
  --puzzle-center-y: calc((var(--puzzle-height) - 1) / 2);
  --tile-unit: min(90vw / var(--puzzle-width), 70vh / var(--puzzle-height));
  position: relative;
  width: calc(var(--puzzle-width) * var(--tile-unit) - 1px);
  height: calc(var(--puzzle-height) * var(--tile-unit) - 1px);
  background: gray;
  margin: auto;
  outline: 1px solid black;
  overflow: hidden;
}

#focused-word-info {
  width: fit-content;
  margin: auto;
  margin-top: 0.25em;
  padding: 0px;
}

#focused-word-direction {
  font-weight: bold;
}

#focused-word-clue {
  display: inline-block;
  padding-left: 4ex;
}

#focused-word-clue li {
  background: none;
}

#clues {
  width: fit-content;
  margin: auto;
}
</style>
