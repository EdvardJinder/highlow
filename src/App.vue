<script setup>
import axios from "axios";
import { ref, computed, watch } from "vue";
const dark = ref(true);
const insult = ref(null);
const alert = ref(null);
const highscore = ref(null);
const name = ref(null);
const game = ref(null);

const start = async () => {
  if (name.value !== null) {
    axios
      .get(`https://localhost:7253/api/Game/Start?name=${name.value}`)
      .then((response) => {
        game.value = response.data;
      })
      .catch((err) => {
        console.log(err);
      });
  }
};

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
  const response = await axios.get(
    `https://localhost:7253/api/Game/Start?name=${name.value}`
  );
  game.value = response.data;
  insult.value = null;
};

const getHighscore = async () => {
  const response = await axios.get(`https://localhost:7253/api/Game/Highscore`);
  highscore.value = response.data;
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

watch(game, (value) => {
  if (!value.isRunning) {
    axios.get("https://insult.mattbas.org/api/insult").then((response) => {
      insult.value = response.data;
      console.log(insult.value);
    });
  } else {
    if (value.score % 5 == 0 && value.score > 4) {
      console.log(value.score);
      alert.value = {
        type: "success",
        message: "You are doing great!",
      };

      setTimeout(() => {
        alert.value = null;
      }, 3000);
    }
  }
});
</script>

<template>
  <v-app :theme="dark ? 'dark' : 'light'">
    <v-app-bar>
      <v-btn icon @click="dark = !dark">
        <v-icon>mdi-brightness-4</v-icon>
      </v-btn>
      <v-spacer></v-spacer>
    </v-app-bar>
    <v-main>
      <v-alert v-if="alert" :title="alert.message" :type="alert.type"></v-alert>
      <v-container
        v-if="!highscore"
        class="d-flex w-100 h-100 align-center justify-center text-center"
      >
        <div v-if="game && game.deck.count > 0" class="d-flex flex-column align-center">
          <!-- <h2>{{ game.currentCard.value }}</h2> -->
          <h3 class="mb-5">SCORE: {{ game.score }}</h3>
          <v-img :src="`src/assets/cards/${computedCardUrl}.svg`" width="250" />
          <div v-if="game.isRunning">
            <div class="my-5">
              <v-btn @click="guess(false)" color="red">Lower</v-btn>
              <v-btn class="mx-1" @click="guess(null)" color="yellow">Same</v-btn>
              <v-btn @click="guess(true)" color="blue">Higher</v-btn>
            </div>
            <h4>Cards left: {{ game.deck.count }}</h4>
          </div>
          <div v-else class="mt-3">
            <h2 class="mt-3" v-if="insult">{{ insult }}</h2>
            <v-btn class="mr-3 my-3" @click="restart" color="blue">Restart</v-btn>
            <v-btn @click="getHighscore" color="yellow">View Highscore</v-btn>
          </div>
        </div>
        <div v-else class="w-100 d-flex flex-column align-center">
          <v-card width="300" color="rgba(0,0,0,0)">
            <v-card-text>
              <v-text-field v-model="name" label="Namn"></v-text-field>
            </v-card-text>
            <v-btn @click="start">Starta</v-btn>
          </v-card>
        </div>
      </v-container>
      <v-container v-else>
        <v-btn @click="highscore = null" color="yellow" class="my-3">Tillbaka</v-btn>
        <v-list bg-color="rgba(0,0,0,0)">
          <v-list-item
            v-for="(item, index) in highscore"
            :key="item.id"
            :title="`${index + 1}: ${item.name}`"
            :subtitle="`Score: ${item.score}`"
          ></v-list-item> </v-list
      ></v-container>
    </v-main>
  </v-app>
</template>
