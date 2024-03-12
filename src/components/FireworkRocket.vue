<template>
  <div
    class="firework-rocket"
    :style="`
      --rocket-size: ${size};
      --rocket-color: ${color};
      --rocket-duration: ${Number.parseFloat(burstY) / 100};
      --rocket-delay: ${delay};
      --launch-x: ${launchX};
      --burst-y: ${burstY};
    `"
  >
    <div
      class="firework-particle"
      :style="`--trajectory: ${(i / 15) * 360}deg;`"
      v-for="i in 15"
      :key="i"
    ></div>
  </div>
</template>

<script>
export default {
  name: "FireworkRocket",
  props: ["size", "color", "delay", "launchX", "burstY"],
};
</script>

<style>
@property --rocket-size {
  syntax: "<length>";
  inherits: true;
  initial-value: 5px;
}

@property --rocket-color {
  syntax: "<color>";
  inherits: true;
  initial-value: white;
}

@property --rocket-duration {
  syntax: "<time>";
  inherits: true;
  initial-value: 1s;
}

@property --rocket-delay {
  syntax: "<time>";
  inherits: true;
  initial-value: 0s;
}

@property --launch-x {
  syntax: "<length-percentage>";
  inherits: false;
  initial-value: 0%;
}

@property --burst-y {
  syntax: "<length-percentage>";
  inherits: false;
  initial-value: 100%;
}

@property --trajectory {
  syntax: "<angle>";
  inherits: false;
  initial-value: 0deg;
}

@keyframes firework-launch {
  0% {
    bottom: 0%;
  }
  100% {
    bottom: var(--burst-y);
  }
}

.firework-rocket {
  position: absolute;
  left: var(--launch-x);
  animation: var(--rocket-duration) linear var(--rocket-delay) both
    firework-launch;
}

@keyframes firework-burst {
  0% {
    left: 0px;
    bottom: 0px;
    background: var(--rocket-color);
    border-radius: 50%;
    opacity: 100%;
  }
  85% {
    opacity: 100%;
  }
  100% {
    left: calc(var(--rocket-size) * 10 * cos(var(--trajectory)));
    bottom: calc(var(--rocket-size) * 10 * sin(var(--trajectory)));
    background: var(--rocket-color);
    border-radius: 50%;
    opacity: 0%;
  }
}

.firework-particle {
  width: var(--rocket-size);
  height: var(--rocket-size);
  border-radius: 50% 50% 0% 0%;
  background: var(--background-1);
  position: absolute;
  animation: 0.4s ease-out calc(var(--rocket-delay) + var(--rocket-duration))
    forwards firework-burst;
}
</style>
