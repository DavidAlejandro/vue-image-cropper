<template>
  <div id="app">
    <ImageCropper />
  </div>
</template>

<script>
import ImageCropper from "./components/ImageCropper.vue";

export default {
  name: "App",
  components: {
    ImageCropper,
  },
};
</script>

<style>
@font-face {
  font-family: "Roboto";
  font-style: normal;
  font-weight: 400;
  src: url("./assets/fonts/Roboto/Roboto-Regular.ttf");
}

@font-face {
  font-family: "Roboto";
  font-style: normal;
  font-weight: bold;
  src: url("./assets/fonts/Roboto/Roboto-Bold.ttf");
}

html {
  background: rgb(33, 44, 49);
  height: 100%;
}
body {
  height: 100%;
}
#app {
  font-family: Roboto, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  display: flex;
  height: 100%;
  align-items: center;
  justify-content: center;
}
</style>
