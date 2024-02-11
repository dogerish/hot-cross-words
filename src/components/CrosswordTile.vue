<template>
  <div
    class="crossword-tile"
    :style="`--coord-x: ${x}px; --coord-y: ${y}px`"
    tabindex="0"
    @focus="stealFocus(this)"
    @blur="stealFocus(null)"
    @keypress="value = $event.key"
  >
    {{ value.toLowerCase() }}
  </div>
</template>

<script>
export default {
  name: "CrosswordTile",
  props: ["x", "y", "letter", "registerTile", "focused", "stealFocus"],
  data() {
    return { value: " " };
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

.crossword-tile:focus {
  background: lightblue;
}
</style>
