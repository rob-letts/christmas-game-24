<script setup>
// do design
// animation
// deploy

import { ref, computed } from 'vue';

import bob from './assets/bob.png';
import bobby from './assets/bobby.png';
import charlotte from './assets/charlotte.png';
import ethan from './assets/ethan.png';
import kevin from './assets/kevin.png';
import kiki from './assets/kiki.png';
import lavender from './assets/lavender.png';
import river from './assets/river.png';
import stuart from './assets/stuart.png';
import roz from './assets/roz.png';

/* CONSTANTS */
const COLUMN_COUNT = 7;
const ROW_COUNT = 6;

/* ASSETS */
const images = [
  bob,
  bobby,
  charlotte,
  ethan,
  kevin,
  kiki,
  lavender,
  river,
  stuart,
  roz
];

/* STATE */
const currentPlayer = ref('one');
const hasWinner = ref(false);
const turnCount = ref(0);
const gameStarted = ref(false);
const isDraw = ref(false);
const cells = ref(getInitialCells());
const players = ref({
  one: {
    name: '',
    imgIndex: 0
  },
  two: {
    name: '',
    imgIndex: 1
  }
});

/* HELPER FUNCTIONS */
function getInitialCells() {
  return Array.from({ length: COLUMN_COUNT }, () =>
    Array.from({ length: ROW_COUNT }, () => ({ player: null }))
  )
}

function resetGame() {
  players.value.one.name = '';
  players.value.two.name = '';
  players.value.one.imgIndex = 0;
  players.value.two.imgIndex = 1;
  currentPlayer.value = 'one';
  hasWinner.value = false;
  turnCount.value = 0;
  gameStarted.value = false;
  isDraw.value = false;
  cells.value = getInitialCells();
}

function updateColumn(column, columnIndex) {
  if (hasWinner.value) return;

  if (turnCount.value === COLUMN_COUNT * ROW_COUNT - 1) {
    isDraw.value = true;
    return;
  }

  const rowIndex = column.findIndex(row => row.player === null);
  if (rowIndex === -1) return;
  column[rowIndex].player = currentPlayer.value;

  if (checkWin(column, columnIndex, rowIndex)) {
    hasWinner.value = true;
  } else {
    updateTurn();
  }
}

function updateTurn() {
  turnCount.value++;
  currentPlayer.value = currentPlayer.value === 'one' ? 'two' : 'one';
}

function checkWin(column, columnIndex, rowIndex) {
  return checkHorizontal(rowIndex) || checkVertical(column) || checkDiagonal(columnIndex, rowIndex)
}

function checkCells(cellsBeingChecked) {
  if (cellsBeingChecked.length < 4) return false;

  return cellsBeingChecked.some((cell, index) => {
    if (index > 3) return false;
    return [
      cell,
      cellsBeingChecked[index + 1],
      cellsBeingChecked[index + 2],
      cellsBeingChecked[index + 3]
    ].every(value => value === currentPlayer.value);
  });
}

function checkHorizontal(rowIndex) {
  const cellsBeingChecked = cells.value.map(column => column[rowIndex].player);
  return checkCells(cellsBeingChecked);
}

function checkVertical(column) {
  const cellsBeingChecked = column.map(cell => cell.player);
  return checkCells(cellsBeingChecked);
}

function checkDiagonal(columnIndex, rowIndex) {
  const offsets = [-3, -2, -1, 0, 1, 2, 3];
  const possibleDiagonals = [
    offsets.map(offset => cells.value[columnIndex + offset]?.[rowIndex + offset]?.player),
    offsets.map(offset => cells.value[columnIndex + offset]?.[rowIndex - offset]?.player)
  ];

  return possibleDiagonals.some(diagonal => checkCells(diagonal));
}

function changeImage(player, ascending = true) {
  const index = players.value[player].imgIndex;
  if (ascending && index === images.length - 1) {
    players.value[player].imgIndex = 0;
  } else if (!ascending && index === 0) {
    players.value[player].imgIndex = images.length - 1;
  } else {
    players.value[player].imgIndex = ascending ? index + 1 : index - 1;
  }
}

/* COMPUTED VALUES */
const message = computed(() => {
  if (isDraw.value) return `It's a draw!`;
  const name =  players.value[currentPlayer.value === 'one' ? 'one' : 'two'].name
  const announcement = hasWinner.value ? ' wins!' : `'s turn`;
  return `${name}${announcement}`;
})

const canStartGame = computed(() => (
  players.value.one.name
  && players.value.two.name
  && (players.value.one.name !== players.value.two.name)
  && (players.value.one.imgIndex !== players.value.two.imgIndex)
));

const gameComplete = computed(() => hasWinner.value || isDraw.value);
</script>

<template>
  <main>
    <h1>Merry Christmas River</h1>

    <template v-if="!gameStarted">
      <input v-model="players.one.name" placeholder="Player One Name" />
      <input v-model="players.two.name" placeholder="Player Two Name" />

      <img height="200" width="200" :src="images[players.one.imgIndex]" />
      <button @click="changeImage('one', false)">Previous</button>
      <button @click="changeImage('one')">Next</button>

      <img height="200" width="200" :src="images[players.two.imgIndex]" />
      <button @click="changeImage('two', false)">Previous</button>
      <button @click="changeImage('two')">Next</button>
      <button @click="gameStarted = true" :disabled="!canStartGame">Start Game</button>
    </template>

    <template v-else>
      <h2>{{ message }}</h2>
      <img :src="images[players[currentPlayer].imgIndex]" />
      <button v-if="gameComplete" @click="resetGame">Play Again</button>

      <div v-for="column, columnIndex in cells">
        <button :disabled="gameComplete" @click="updateColumn(column, columnIndex)" v-for="cell in column">
          <img v-if="cell.player" :src="images[players[cell.player].imgIndex]" height="25" width="25" />
          <p v-else>X</p>
        </button>
      </div>
      <button v-if="!gameComplete" @click="resetGame">Start Again</button>
    </template>
  </main>
</template>