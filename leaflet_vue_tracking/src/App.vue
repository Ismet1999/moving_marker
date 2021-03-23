<template>
  <div id="app">
    <Map :duration="durationInteger" :keepAtCenter="keepAtCenter" />
  </div>
</template>

<script>
import Map from "./components/index.vue";

export default {
  name: "App",
  components: {
    Map,
  },
  data() {
    return {
      duration: 2000,
      keepAtCenter: false,
    };
  },
  computed: {
    durationInteger() {
      return parseInt(this.duration);
    },
  },
};
</script>

<style>
html,
body,
#app {
  height: 100%;
  width: 100%;
  margin: 0;
}

#app {
  display: flex;
}
</style>
