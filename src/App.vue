<template>
  <v-app>
    <v-app-bar app color="primary" dark flat>
      <div class="d-flex align-center">
        
      </div>
      <v-spacer></v-spacer>
    </v-app-bar>

    <v-main class="mb-12">
      <MainThread/>
    </v-main>
  </v-app>
</template>

<script>
import MainThread from './components/MainThread';

export default {
  name: 'App',

  components: {
    MainThread,
  },

  data: () => ({
    //
  }),
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}
</style>
