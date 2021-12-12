<template>
  <v-app>
    <v-app-bar app color="primary" dark>
      <v-app-bar-title>Scan</v-app-bar-title>
    </v-app-bar>

    <v-main align="center">
      <router-link to="/">Bluetooth</router-link> |
      <router-link to="/">MQTT</router-link> |
      <router-link to="/history">Historique</router-link>
      <router-view />
    </v-main>
  </v-app>
</template>

<script>
export default {
  name: "App",

  data: () => ({
    //
  }),
};
</script>
