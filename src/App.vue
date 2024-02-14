<template>
  <nav>
    <router-link to="/">Home</router-link> |
    <router-link to="/about">About</router-link>
  </nav>
  <router-view />
</template>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  --subject-1: mediumpurple;
  --subject-2: plum;
  --subject-3: lightpink;
  --object-1: black;
  --object-2: midnightblue;
  --object-3: indigo;
  --background-1: white;
  color: var(--object-1);
  background: var(--background-1);
}

nav {
  padding: 30px;
}

nav a {
  font-weight: bold;
  color: var(--object-2);
}

nav a.router-link-exact-active {
  color: var(--subject-1);
}
</style>
