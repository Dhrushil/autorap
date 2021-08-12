<template>
  <v-app>
    <v-main class="app">
      <v-container>
        <h1>AutoRAP</h1>

        <v-btn @click="rhyme">rhyme</v-btn>
      </v-container>

      <speechtotext />
    </v-main>
  </v-app>
</template>


<style>
.app {
  background-color: orange;
  background-image: url("./assets/background.jpg");
}
.glassmorphic {
  background: rgba(255, 0, 171, 0.25);
  box-shadow: 0 8px 32px 0 rgba(31, 38, 135, 0.37);
  backdrop-filter: blur(4px);
  -webkit-backdrop-filter: blur(4px);
  border-radius: 10px;
  border: 1px solid rgba(255, 255, 255, 0.18);
}
.center {
  margin: auto;
  width: 50%;
}
</style>

<script>
import Speechtotext from "./components/speechtotext.vue";
import axios from "axios";

export default {
  name: "speech_to_text",
  components: {
    Speechtotext,
  },

  methods: {
    rhyme() {
         axios
            .get("https://api.datamuse.com/words?rel_rhy=yes")
            .then((res) => {
              console.log(res)
            });
    },
  },
};
</script>