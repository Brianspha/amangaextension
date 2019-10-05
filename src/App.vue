<template>
  <v-app id="inspire">
    <v-app-bar app color="indigo" dark>
      <v-app-bar-nav-icon  ><img src="https://image.flaticon.com/icons/svg/1862/1862164.svg" width="30" height="30"/></v-app-bar-nav-icon>
      <v-toolbar-title>Amanga</v-toolbar-title>
      
    </v-app-bar>

    <v-content>
      <v-container  fluid>
        <v-row align="center" justify="center">
          <v-col class="text-center">
            <Repo></Repo>
          </v-col>
        </v-row>
      </v-container>
    </v-content>

    <v-footer color="indigo" app>
      <span class="white--text">&copy; 2019</span>
    </v-footer>
  </v-app>
</template>

<script>
import Repo from "./components/Repo.vue";
export default {
  name: "app",
  components: {
    Repo
      },
  data() {
    return {
      drawer: null
    };
  }
};
</script>
<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>