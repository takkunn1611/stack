<template>
  <div id="app">
    <div id="nav">
      <router-link to="/">Timer</router-link>
    </div>
    <router-view class="content" />
  </div>
</template>

<style>
*, ::before, ::after {
  margin: 0;
  box-sizing: border-box;
}

#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  height: 100vh;
  width: 100vw;
  display: flex;
  flex-direction: column;
}

#nav {
  padding: 30px;
}

#nav a {
  font-weight: bold;
  color: #2c3e50;
}

#nav a.router-link-exact-active {
  color: #42b983;
}

#app .content {
  flex: 1;
  padding: 2rem;
}

</style>
