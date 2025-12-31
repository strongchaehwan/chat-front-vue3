<template>
  <v-app>
    <HeaderComponent />
    <v-main>
      <RouterView />
    </v-main>
  </v-app>
</template>

<script>
import { RouterView } from "vue-router";
import HeaderComponent from "./components/HeaderComponent.vue";

export default {
  name: "App",
  components: { RouterView, HeaderComponent },
};
</script>

<style></style>
