<template>
  <div id="app">
    <div class="audio-controls">
      <audio
        ref="audioPlayer"
        @play="handlePlay"
        @timeupdate="onTimeUpdate"
        @ended="handleEnded"
        @error="handleError"
        :src="audioSource"
        controls
      ></audio>
    </div>

    <div class="dialog">
      <div
        v-for="(dialog, dialogIndex) in jsonData.dialog"
        :key="dialogIndex"
        :class="['dialog-line', { 'second-speaker': dialog.role === 'Lan' }]"
      >
        <span
          v-for="(word, wordIndex) in getDialogWords(dialog.text)"
          :key="wordIndex"
          :class="[
            'word',
            { highlight: isWordHighlighted(dialogIndex, wordIndex) },
          ]"
        >
          {{ word }}
        </span>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from "vue";
import jsonData from "./assets/jameslan.json";
import audioFile from "@/assets/jameslan.ogg";

const audioSource = audioFile;
const audioPlayer = ref(null);
const currentWordIndex = ref(-1);

const getDialogWords = (text) => {
  const words = text.split(" ");
  return words;
};

const isWordHighlighted = (dialogIndex, wordIndex) => {
  const previousWords = jsonData.dialog
    .slice(0, dialogIndex)
    .reduce((sum, dialog) => sum + dialog.text.split(" ").length, 0);

  const actualWordIndex = previousWords + wordIndex;

  return actualWordIndex === currentWordIndex.value;
};

const onTimeUpdate = () => {
  const currentTime = audioPlayer.value.currentTime * 1000;

  for (let i = 0; i < jsonData.timestamp.length; i++) {
    const [start, duration] = jsonData.timestamp[i];
    if (currentTime >= start && currentTime <= start + duration) {
      currentWordIndex.value = i;
      break;
    }
  }
};

const handlePlay = () => {
  console.log(audioSource, audioPlayer);
};

const handleEnded = () => {
  currentWordIndex.value = -1;
};

const handleError = (e) => {
  console.error("Lỗi audio:", e.target.error);
};
</script>

<style scoped>
.dialog {
  line-height: 1.8;
  font-size: 18px;
  padding: 20px;
  text-align: left;
}

.dialog-line {
  margin-bottom: 1rem;
  color: #333;
}

.second-speaker {
  color: #2196f3;
}

.word {
  margin-right: 0.3em;
}

.highlight {
  background-color: yellow;
}

.audio-controls {
  margin: 20px;
  text-align: center;
}

audio {
  margin: 20px;
  width: 300px;
}
</style>
