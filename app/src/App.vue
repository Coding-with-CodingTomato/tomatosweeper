<script setup>
import fieldRow from "./components/fieldRow.vue";
import { ref, onBeforeMount } from "vue";

const gameState = ref({
  gameOver: false,
  gameWon: false,
});

const started = ref(false);
const winner = ref("");
const fieldBoxes = ref([]);
const fieldSize = ref(7);
const frequency = ref(0.15);

// [
//   [b, b, b, b, b, b],
//   [b, b, b, b, b, b],
//   [b, b, b, b, b, b],
//   [b, b, b, b, b, b],
//   [b, b, b, b, b, b],
//   [b, b, b, b, b, b],
// ];
const generateBoxes = (maxX, maxY) => {
  for (let i = 0; i < maxX; i++) {
    const row = [];
    for (let j = 0; j < maxY; j++) {
      const data = {
        row: i,
        col: j,
        revealed: false,
        flagged: false,
        tomato: false,
        neighbors: -1,
      };

      row.push(data);
    }

    fieldBoxes.value.push(row);
  }
};

const placeTomatos = (maxX, maxY, frequency) => {
  const amountTomatos = maxX * maxY * frequency;

  for (let i = 0; i < amountTomatos; i++) {
    let placed = false;

    while (!placed) {
      const row = Math.floor(Math.random() * maxY);
      const col = Math.floor(Math.random() * maxX);

      if (!fieldBoxes.value[row][col].tomato) {
        fieldBoxes.value[row][col].tomato = true;
        placed = true;
      }
    }
  }
};

const calcNeighbors = () => {
  for (let i = 0; i < fieldBoxes.value.length; i++) {
    for (let j = 0; j < fieldBoxes.value[i].length; j++) {
      let totalNeighbors = 0;
      if (fieldBoxes.value[i][j].tomato) continue;

      for (let offI = -1; offI <= 1; offI++) {
        for (let offJ = -1; offJ <= 1; offJ++) {
          const neighborI = i + offI;
          const neighborJ = j + offJ;

          if (
            neighborI < 0 ||
            neighborI > fieldBoxes.value[i].length - 1 ||
            neighborJ < 0 ||
            neighborJ > fieldBoxes.value[i].length - 1
          ) {
            continue;
          }

          if (fieldBoxes.value[i + offI][j + offJ].tomato) {
            totalNeighbors += 1;
          }
        }
      }

      fieldBoxes.value[i][j].neighbors = totalNeighbors;
    }
  }
};

const revealBox = (row, col, username) => {
  if (gameState.value.gameOver) return;

  winner.value = username;

  const box = fieldBoxes.value[row][col];

  if (!box) return;
  box.revealed = true;

  if (box.tomato) {
    gameState.value.gameOver = true;

    setTimeout(() => {
      initGame();
      started.value = false;
    }, 10 * 1000);
  }

  const allNonTomRevealed = fieldBoxes.value.some((row) => {
    const rowRevealed = row.some((box) => {
      if (!box.tomato && !box.revealed) return true;
      return false;
    });
    if (rowRevealed) return true;
    return false;
  });

  if (!allNonTomRevealed) {
    gameState.value.gameWon = true;

    fieldBoxes.value.forEach((row) => {
      row.forEach((box) => {
        box.revealed = true;
      });
    });

    setTimeout(() => {
      initGame();
      started.value = false;
    }, 10 * 1000);
  }

  if (box.neighbors === 0) {
    for (let offI = -1; offI <= 1; offI++) {
      for (let offJ = -1; offJ <= 1; offJ++) {
        const neighborI = row + offI;
        const neighborJ = col + offJ;

        if (
          neighborI < 0 ||
          neighborI > fieldBoxes.value[row].length - 1 ||
          neighborJ < 0 ||
          neighborJ > fieldBoxes.value[row].length - 1
        ) {
          continue;
        }

        const neighbor = fieldBoxes.value[neighborI][neighborJ];

        if (!neighbor.revealed) {
          revealBox(neighbor.row, neighbor.col, username);
        }
      }
    }
  }
};

const flagBox = (row, col) => {
  if (gameState.value.gameOver) return;

  const box = fieldBoxes.value[row][col];

  if (!box && !box.revealed) return;
  box.flagged = true;
};

const initGame = () => {
  fieldBoxes.value = [];
  gameState.value = {
    gameOver: false,
    gameWon: false,
  };

  generateBoxes(fieldSize.value, fieldSize.value);
  placeTomatos(fieldSize.value, fieldSize.value, frequency.value);
  calcNeighbors();
};

onBeforeMount(() => {
  initGame();
});

const client = new tmi.Client({
  channels: ["codingtomato"],
});

client.connect();

client.on("message", (channel, tags, message, self) => {
  if (self || !message.startsWith("!")) return;

  const args = message.slice(1).split(" ");
  const command = args.shift().toLowerCase();

  if (command === "sweeper" || command === "s") {
    const row = parseInt(args[0], 10);
    const col = parseInt(args[1], 10);

    started.value = true;

    if (!isNaN(row) && !isNaN(col)) {
      revealBox(row, col);
    }
  }

  if (command === "flag" || command === "f") {
    const row = parseInt(args[0], 10);
    const col = parseInt(args[1], 10);

    if (!isNaN(row) && !isNaN(col)) {
      flagBox(row, col);
    }
  }
});
</script>

<template>
  <div class="app" v-if="started">
    <h1 v-if="!gameState.gameOver && !gameState.gameWon">TomatoSweeper</h1>
    <h1 v-if="gameState.gameOver">Leider verloren!</h1>
    <h1 v-if="gameState.gameWon">Gewonnen, {{ winner }}!</h1>

    <div
      class="field"
      :style="{
        gridTemplateColumns: `repeat(${fieldSize}, 1fr)`,
        gridTemplateRows: `repeat(${fieldSize}, 1fr)`,
      }"
    >
      <fieldRow
        v-for="row in fieldBoxes"
        :key="row"
        :row="row"
        @reveal="revealBox"
      />
    </div>
    <h3>!sweeper #reihe #spalte</h3>
    <h3>!flag #reihe #spalte</h3>
  </div>
</template>

<style>
@import url("https://fonts.googleapis.com/css2?family=Space+Mono:wght@400;700&display=swap");

:root {
  --boxSize: 70px;
}

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  color: white;
  font-family: "Space Mono", monospace;
  font-size: small;
}

.app {
  width: 100vw;
  height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
}

h1 {
  font-size: 30pt;
}

h3 {
  font-size: 18pt;
  text-shadow: 1px solid black;
}

.field {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
  grid-template-rows: repeat(7, 1fr);
  grid-column-gap: 0px;
  grid-row-gap: 0px;
}

.field > div {
  background-color: grey;
  width: var(--boxSize);
  height: var(--boxSize);
  border: 1px solid white;
  display: flex;
  justify-content: center;
  align-items: center;
  cursor: pointer;
}
</style>
