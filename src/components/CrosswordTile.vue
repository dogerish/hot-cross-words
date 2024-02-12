<template>
  <div
    :class="{ 'crossword-tile': true, 'word-focus': wordfocus }"
    :style="`--coord-x: ${x}; --coord-y: ${y}; --start-x: ${start[0]}; --start-y: ${start[1]};`"
    tabindex="0"
    ref="elem"
    @mousedown="
      $event.target == $event.view.document.activeElement && stealFocus(this)
    "
    @focus="stealFocus(this)"
    @blur="
      $event.relatedTarget?.classList?.contains('crossword-tile') ||
        stealFocus(null)
    "
    @keypress="alpha.test($event.key) && (value = $event.key) && focusNext()"
    @keydown.up="focusTo(x, y - 1)"
    @keydown.down="focusTo(x, y + 1)"
    @keydown.left="focusTo(x - 1, y)"
    @keydown.right="focusTo(x + 1, y)"
    @keydown.delete="value === '' ? backspace() : (value = '')"
    @keydown.enter.prevent="focusNext()"
    @keydown.tab.exact.prevent="focusNext()"
    @keydown.shift.tab.exact.prevent="focusNext(-1)"
  >
    {{ value.toLowerCase() }}
  </div>
</template>

<script>
export default {
  name: "CrosswordTile",
  props: [
    "x",
    "y",
    "letter",
    "puzzleSize",
    "registerTile",
    "stealFocus",
    "focusNext",
    "focusTo",
    "backspace",
  ],
  data() {
    // create ring starting locations
    let puzzleCenter = this.puzzleSize.map((s) => (s - 1) / 2);
    let ang = Math.atan2(this.y - puzzleCenter[0], this.x - puzzleCenter[1]);
    let start = [Math.cos, Math.sin].map(
      (f, i) => puzzleCenter[i] + f(ang) * (this.puzzleSize[i] / 3)
    );
    return { alpha: /^[A-Za-z]$/, value: "", wordfocus: false, start };
  },
  methods: {
    focus() {
      this.$refs.elem.focus();
    },
  },
  created() {
    this.registerTile(this);
  },
};
</script>

<style>
@keyframes explode-tiles {
  0% {
    left: calc(50% - var(--tile-unit) / 2);
    top: calc(50% - var(--tile-unit) / 2);
    transform: rotate(-240deg);
  }
  60% {
    left: calc(var(--start-x) * var(--tile-unit) - 1px);
    top: calc(var(--start-y) * var(--tile-unit) - 1px);
  }
  100% {
    left: calc(var(--coord-x) * var(--tile-unit) - 1px);
    top: calc(var(--coord-y) * var(--tile-unit) - 1px);
  }
}

.crossword-tile {
  position: absolute;
  background: white;
  border: 1px solid black;
  width: calc(var(--tile-unit) - 1px);
  height: calc(var(--tile-unit) - 1px);
  font-size: calc(var(--tile-unit) * 0.75);
  overflow: hidden;
  display: inline-block;
  animation: explode-tiles 0.5s ease-out;
  animation-fill-mode: both;
}

.crossword-tile.word-focus {
  background: lightblue;
}

.crossword-tile:focus,
.crossword-tile:focus-visible {
  background: salmon;
  outline: none;
}
</style>
