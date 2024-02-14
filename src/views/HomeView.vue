<template>
  <div class="home">
    <h1>Crossword Puzzle: {{ title }}</h1>
    <div id="crossword-container">
      <div id="puzzle-area" class="crossword-item">
        <CrosswordPuzzle
          :rawWords="rawWords"
          :gradeMode="gradeMode"
          :ref="(p) => (puzzle = p)"
        />
        <ol id="focused-word-info">
          <span id="focused-word-direction" v-if="focusedWord">
            {{ focusedWord.across ? "Across" : "Down" }}
          </span>
          <div id="focused-word-clue">
            <CrosswordClue
              :num="focusedWord.num"
              :clue="focusedWord.clue"
              v-if="focusedWord"
            />
          </div>
        </ol>
      </div>
      <div id="clue-list" class="crossword-item">
        <div id="grading-options">
          <label for="grading-mode">
            Grade per
            <select v-model="gradeMode" id="grading-mode">
              <option
                :value="i"
                :key="i"
                v-for="(mode, i) in ['Puzzle', 'Word', 'Letter']"
              >
                {{ mode }}
              </option>
            </select>
          </label>
        </div>
        <div id="clues">
          <CrosswordClueList
            :words="rawDownWords"
            :focusedWord="focusedWord"
            :focusToWord="focusToWord"
            :across="false"
          />
          <CrosswordClueList
            :words="rawAcrossWords"
            :focusedWord="focusedWord"
            :focusToWord="focusToWord"
            :across="true"
          />
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import CrosswordPuzzle from "@/components/CrosswordPuzzle.vue";
import CrosswordClue from "@/components/CrosswordClue.vue";
import CrosswordClueList from "@/components/CrosswordClueList.vue";
import puzzle from "@/puzzles/hot-cross-words.json";

export default {
  name: "HomeView",
  components: {
    CrosswordPuzzle,
    CrosswordClue,
    CrosswordClueList,
  },
  data() {
    let words = puzzle.words.map(([x, y, dir, word, clue]) => {
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
    // derive down-across boundary
    words.firstAcross = words.findIndex((w) => w.across);
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
    return { title: puzzle.title, rawWords: words, puzzle: null, gradeMode: 0 };
  },
  computed: {
    focusedWord() {
      return this.puzzle && this.puzzle.focusedWord;
    },
    rawDownWords() {
      return this.rawWords.slice(0, this.rawWords.firstAcross);
    },
    rawAcrossWords() {
      return this.rawWords.slice(this.rawWords.firstAcross);
    },
  },
  methods: {
    focusToWord(num, across) {
      this.puzzle?.focusToWord(num, across);
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

#puzzle-area {
  flex: 1 0 auto;
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
