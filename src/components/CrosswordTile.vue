<template>
  <div class="crossword-tile" :style="`--coord-x: ${x}; --coord-y: ${y}`">
    <span class="tile-num" v-if="numb != null">{{ numb }}</span>
    <div
      class="focusable-tile"
      :class="{ 'word-focus': wordfocus, gradeworthy, correct }"
      tabindex="0"
      ref="elem"
      @mousedown="
        $event.target === $event.view.document.activeElement && stealFocus(this)
      "
      @focus="stealFocus(this)"
      @blur="
        $event.relatedTarget?.classList.contains('focusable-tile') ||
          stealFocus(null)
      "
      @keypress="
        alpha.test($event.key) &&
          (value = $event.key.toLowerCase()) &&
          focusNext()
      "
      @keydown.up.prevent="focusTo(x, y - 1)"
      @keydown.down.prevent="focusTo(x, y + 1)"
      @keydown.left="focusTo(x - 1, y)"
      @keydown.right="focusTo(x + 1, y)"
      @keydown.delete="value === '' ? backspace() : (value = '')"
      @keydown.enter.prevent="focusNext()"
      @keydown.tab.exact.prevent="focusNext()"
      @keydown.shift.tab.exact.prevent="focusNext(-1)"
    >
      {{ value }}
    </div>
  </div>
</template>

<script>
export default {
  name: "CrosswordTile",
  props: [
    "num",
    "x",
    "y",
    "letter",
    "gradeMode",
    "stealFocus",
    "focusNext",
    "focusTo",
    "backspace",
  ],
  data() {
    let numb = this.num;
    return {
      alpha: /^[A-Za-z]$/,
      value: "",
      wordfocus: false,
      numb,
      showGrade: 0,
    };
  },
  computed: {
    gradeworthy() {
      return (
        this.showGrade > 0 || (this.gradeMode === 2 && this.value.length !== 0)
      );
    },
    correct() {
      return this.value === this.letter;
    },
  },
  methods: {
    focus() {
      this.$refs.elem.focus();
    },
    grade() {
      this.showGrade++;
    },
    ungrade() {
      if (this.showGrade === 0) return;
      this.showGrade--;
    },
    getNum() {
      return this.numb;
    },
    setNum(num) {
      this.numb = num;
    },
  },
};
</script>

<style>
@property --anim-a {
  syntax: "<angle>";
  inherits: false;
  initial-value: 0deg;
}

@property --anim-h {
  syntax: "<number>";
  inherits: false;
  initial-value: 0;
}

@keyframes explode-tiles {
  0% {
    left: calc(var(--anim-x) * var(--tile-unit) - 1px);
    top: calc(var(--anim-y) * var(--tile-unit) - 1px);
    --anim-h: 0;
    --anim-a: 270deg;
    border-color: var(--background-1);
  }
  66% {
    --anim-h: calc((var(--puzzle-width) - 1) / 3);
    --anim-a: calc(var(--coord-a) + 90deg);
    border-color: var(--background-1);
  }
  100% {
    left: calc(var(--anim-x) * var(--tile-unit) - 1px);
    top: calc(var(--anim-y) * var(--tile-unit) - 1px);
    --anim-h: var(--coord-h);
    --anim-a: var(--coord-a);
  }
}

.crossword-tile {
  --coord-a: atan2(
    var(--coord-y) - var(--puzzle-center-y),
    var(--coord-x) - var(--puzzle-center-x)
  );
  --coord-h: sqrt(
    pow(var(--coord-x) - var(--puzzle-center-x), 2) +
      pow(var(--coord-y) - var(--puzzle-center-y), 2)
  );
  --anim-x: calc(var(--puzzle-center-x) + cos(var(--anim-a)) * var(--anim-h));
  --anim-y: calc(var(--puzzle-center-y) + sin(var(--anim-a)) * var(--anim-h));
  position: absolute;
  left: calc(var(--coord-x) * var(--tile-unit) - 1px);
  top: calc(var(--coord-y) * var(--tile-unit) - 1px);
  background: var(--background-1);
  border: 1px solid var(--object-1);
  width: calc(var(--tile-unit) - 1px);
  height: calc(var(--tile-unit) - 1px);
  font-size: calc(var(--tile-unit) * 0.75);
  overflow: hidden;
  display: inline-block;
  animation: explode-tiles 0.5s linear;
  animation-fill-mode: backwards;
}

.focusable-tile {
  width: 100%;
  height: 100%;
}

.gradeworthy:not(.correct) {
  text-decoration: line-through;
}

.gradeworthy.correct {
  background: var(--subject-3);
}

.focusable-tile.word-focus {
  background: var(--subject-2);
}

.focusable-tile:focus,
.focusable-tile:focus-visible {
  background: var(--subject-1);
  outline: none;
}

.tile-num {
  position: absolute;
  left: 5%;
  top: 5%;
  font-size: 35%;
}
</style>
