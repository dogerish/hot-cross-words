<template>
  <div
    :class="{ 'crossword-tile': true, 'word-focus': wordfocus }"
    :style="`--coord-x: ${x}px; --coord-y: ${y}px`"
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
    "registerTile",
    "stealFocus",
    "focusNext",
    "focusTo",
    "backspace",
  ],
  data() {
    return { alpha: /^[A-Za-z]$/, value: "", wordfocus: false };
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
.crossword-tile {
  position: absolute;
  left: calc(var(--coord-x) * 50 - 1px);
  top: calc(var(--coord-y) * 50 - 1px);
  background: white;
  border: 1px solid black;
  width: 49px;
  height: 49px;
  line-height: 50px;
  font-size: 40px;
  overflow: hidden;
  display: inline-block;
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
