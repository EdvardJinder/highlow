<script setup>
import axios from "axios";
import { ref, computed } from "vue";
const dark = ref(true);
const otherGames = ref(false);

const game = ref(null);
axios
  .get("https://localhost:7253/api/Game/Start")
  .then((response) => {
    game.value = response.data;
  })
  .catch((err) => {
    console.log(err);
  });

const guess = async (higher) => {
  if (higher == null) {
    const response = await axios.get(`https://localhost:7253/api/Game/Deal`);
    game.value = response.data;
  } else {
    const response = await axios.get(
      `https://localhost:7253/api/Game/Deal?higher=${higher}`
    );
    game.value = response.data;
  }
};

const restart = async () => {
  const response = await axios.get(`https://localhost:7253/api/Game/Start`);
  game.value = response.data;
};

const computedCardUrl = computed(() => {
  console.log(game.value.currentCard.color);
  let color;

  switch (game.value.currentCard.color) {
    case 0:
      color = "clubs";
      break;
    case 1:
      color = "diamonds";
      break;
    case 2:
      color = "hearts";
      break;
    case 3:
      color = "spades";
      break;
    default:
      "clubs";
  }

  console.log(`${color}_${game.value.currentCard.value}`);

  return `${color}_${game.value.currentCard.value}`;
});
</script>

<template>
  <v-app :theme="dark ? 'dark' : 'light'">
    <v-app-bar>
      <v-btn icon @click="dark = !dark">
        <v-icon>mdi-brightness-4</v-icon>
      </v-btn>
      <v-spacer></v-spacer>
      <v-btn @click="otherGames = !otherGames">Other games</v-btn>
    </v-app-bar>
    <v-main>
      <v-container
        v-if="!otherGames"
        class="d-flex w-100 h-100 align-center justify-center text-center"
      >
        <div v-if="game && game.deck.count > 0">
          <h3 class="mb-5">SCORE: {{ game.score }}</h3>

          <!-- <h2>{{ game.currentCard.value }}</h2> -->
          <v-img :src="`src/assets/cards/${computedCardUrl}.svg`" size="260" />
          <div v-if="game.isRunning">
            <div class="my-5">
              <v-btn @click="guess(false)">Lower</v-btn>
              <v-btn class="mx-1" @click="guess(null)">Same</v-btn>
              <v-btn @click="guess(true)">Higher</v-btn>
            </div>
          </div>
          <div v-else>
            <v-btn class="mr-3 my-3" @click="restart">Restart</v-btn>
            <v-btn @click="highscore">View Highscore</v-btn>
            <h2 class="mt-3">FUCK YOU LOSER!</h2>
          </div>

          <h4>Cards left: {{ game.deck.count }}</h4>
        </div>
        <div v-else>FUCK OFF</div>
      </v-container>
      <v-container v-else> Ricky </v-container>
    </v-main>
  </v-app>
</template>
