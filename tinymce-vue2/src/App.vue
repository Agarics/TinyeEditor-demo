<template>
  <div id="app">
    <h1>this is TinyeEitor sub app</h1>
    <TinyeEitor v-model="content" width="920"></TinyeEitor>
  </div>
</template>

<script>
import TinyeEitor from './components/Tinymce/index.vue';

export default {
  name: 'App',
  data() {
    return {
      content: '',
    };
  },
  components: {
    TinyeEitor,
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
