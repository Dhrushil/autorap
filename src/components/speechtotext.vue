<template>
  <div class="speechtotext" style="height: 80%; width: 100%">
    <v-container style="height: 100%; width: 100%">
      <v-sheet
        height="40%"
        width="100%"
        style="
          background: rgba(255, 0, 171, 0.25);
          box-shadow: 0 8px 32px 0 rgba(31, 38, 135, 0.37);
          backdrop-filter: blur(4px);
          -webkit-backdrop-filter: blur(4px);
          border-radius: 1em;
          border: 1px solid rgba(255, 255, 255, 0.18);
        "
        dark
        class="ma-1"
      >
        <v-container>
          <h4 style="text-align: center">Your lyrics</h4>
          <v-divider color="white" />
          <p class="mb-0">
            <span v-for="sentence in sentences" :key="sentence.id"
              >{{ sentence }}.
            </span>
            <span>{{ runtimeTranscription }}</span>
          </p>
        </v-container>
      </v-sheet>
      <v-sheet
        class="ma-1"
        height="40%"
        width="100%"
        style="
          background: rgba(255, 0, 171, 0.25);
          box-shadow: 0 8px 32px 0 rgba(31, 38, 135, 0.37);
          backdrop-filter: blur(4px);
          -webkit-backdrop-filter: blur(4px);
          border-radius: 1em;
          border: 1px solid rgba(255, 255, 255, 0.18);
          
        "
        dark
      >
        <v-container style="overflow: scroll; height: 80%">
          <h4 style="text-align: center">Prediction</h4>
          <v-divider color="white" />
          <p v-for="word in words" :key="word.word" style="display:inline-flex;" class="mr-3">{{word.word}}</p>
        </v-container>
      </v-sheet>
    </v-container>
    <v-btn
      style="
        background: rgba(255, 0, 171, 0.25);
        box-shadow: 0 8px 32px 0 rgba(31, 38, 135, 0.37);
        backdrop-filter: blur(4px);
        -webkit-backdrop-filter: blur(4px);
        border-radius: 1em;
        border: 1px solid rgba(255, 255, 255, 0.18);
      "
      dark
      @click.stop="toggle ? endSpeechRecognition() : startSpeechRecognition()"
      class="center"
      fab
      x-large
      icon
    >
      <font-awesome-icon :icon="micStatus" />
    </v-btn>
  </div>
</template>

<script>
let SpeechRecognition =
  window.SpeechRecognition || window.webkitSpeechRecognition;
let recognition = SpeechRecognition ? new SpeechRecognition() : false;
import axios from "axios";

export default {
  props: {
    lang: {
      type: String,
      default: "en-US",
    },
    text: {
      type: [String, null],
      required: true,
    },
  },
  data() {
    return {
      error: false,
      speaking: false,
      micStatus: "microphone",
      toggle: false,
      runtimeTranscription: "",
      sentences: [],
      lastWord: "",
      words: [],
    };
  },
  methods: {
    checkCompatibility() {
      if (!recognition) {
        this.error =
          "Speech Recognition is not available on this browser. Please use Chrome or Firefox";
      }
    },
    endSpeechRecognition() {
      recognition.stop();
      this.micStatus = "microphone";
      this.toggle = false;
      this.$emit("speechend", {
        sentences: this.sentences,
        text: this.sentences.join(". "),
      });
    },
    startSpeechRecognition() {
      if (!recognition) {
        this.error =
          "Speech Recognition is not available on this browser. Please use Chrome or Firefox";
        return false;
      }
      this.toggle = true;
      recognition.lang = this.lang;
      recognition.interimResults = true;
      this.micStatus = "microphone-slash";

      recognition.addEventListener("speechstart", (event) => {
        this.speaking = true;
        console.log(event);
      });

      recognition.addEventListener("speechend", (event) => {
        this.speaking = false;
        console.log(event);
      });

      recognition.addEventListener("result", (event) => {
        const text = Array.from(event.results)
          .map((result) => result[0])
          .map((result) => result.transcript)
          .join("");
        this.runtimeTranscription = text;
      });

      recognition.addEventListener("end", () => {
        if (this.runtimeTranscription !== "") {
          this.sentences.push(
            this.capitalizeFirstLetter(this.runtimeTranscription)
          );
          this.$emit(
            "update:text",
            `${this.text}${this.sentences.slice(-1)[0]}. `
          );
          this.lastWord = this.sentences[this.sentences.length - 1].split(" ");
          this.lastWord = this.lastWord[this.lastWord.length - 1];
          axios
            .get("https://api.datamuse.com/words?rel_rhy=" + this.lastWord)
            .then((res) => {
              this.words = res.data;
            });
        }
        this.runtimeTranscription = "";
        recognition.stop();
        if (this.toggle) {
          // keep it going.
          recognition.start();
        }
      });
      recognition.start();
    },
    capitalizeFirstLetter(string) {
      return string.charAt(0).toUpperCase() + string.slice(1);
    },
  },
  mounted() {
    this.checkCompatibility();
  },
};
</script>

<style>
.center {
  position: fixed;
  left: 50%;
  transform: translate(-50%, -50%);
  bottom: 0;
}
.center2 {
  position: absolute;
  left: 50%;
  transform: translate(-50%, -50%);
}
</style>