<template>
  <v-app>
    <v-main>
      <mint />
    </v-main>
  </v-app>
</template>

<script>
import Mint from "./components/Mint.vue";

export default {
  name: "App",
  components: {
    mint: Mint,
  },
};
</script>

