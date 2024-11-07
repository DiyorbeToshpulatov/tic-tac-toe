<script setup>
import { ref, computed, watch } from 'vue';

// Game state
const player = ref('X');
const isAiPlaying = ref(false);
const aiDifficulty = ref('easy'); // Track AI difficulty level
const board = ref([
  ['', '', ''],
  ['', '', ''],
  ['', '', ''],
]);

// Function to calculate the winner
const CalculateWinner = squares => {
  const lines = [
    [0, 1, 2],
    [3, 4, 5],
    [6, 7, 8],
    [0, 3, 6],
    [1, 4, 7],
    [2, 5, 8],
    [0, 4, 8],
    [2, 4, 6],
  ];
  for (let i = 0; i < lines.length; i++) {
    const [a, b, c] = lines[i];
    if (squares[a] && squares[a] === squares[b] && squares[a] === squares[c]) {
      return squares[a];
    }
  }
  return null;
};

// Winner computation
const winner = computed(() => CalculateWinner(board.value.flat()));

// Make move for human player
const MakeMove = (x, y) => {
  if (winner.value || board.value[x][y] !== '') return;

  board.value[x][y] = player.value;
  player.value = player.value === 'X' ? 'O' : 'X';

  if (isAiPlaying.value && player.value === 'O' && !winner.value) {
    AiMakeMove(); // Let AI make its move
  }
};

// AI moves based on difficulty level
const AiMakeMove = () => {
  if (aiDifficulty.value === 'easy') {
    AiRandomMove();
  } else if (aiDifficulty.value === 'medium') {
    AiMediumMove();
  } else if (aiDifficulty.value === 'hard') {
    AiMinimaxMove();
  }
};

// Easy mode: AI makes a random move
const AiRandomMove = () => {
  const emptyCells = [];
  for (let x = 0; x < 3; x++) {
    for (let y = 0; y < 3; y++) {
      if (board.value[x][y] === '') {
        emptyCells.push([x, y]);
      }
    }
  }
  if (emptyCells.length > 0) {
    const randomMove =
      emptyCells[Math.floor(Math.random() * emptyCells.length)];
    board.value[randomMove[0]][randomMove[1]] = 'O';
    player.value = 'X';
  }
};

// Medium mode: AI blocks if the player is about to win, otherwise random
const AiMediumMove = () => {
  const bestMove = FindBlockingMove('X'); // Try to block human ('X')
  if (bestMove) {
    board.value[bestMove[0]][bestMove[1]] = 'O';
  } else {
    AiRandomMove(); // Otherwise, play a random move
  }
  player.value = 'X';
};

// Hard mode: AI uses the Minimax algorithm
const AiMinimaxMove = () => {
  const bestMove = Minimax(board.value, 'O').position;
  if (bestMove) {
    board.value[bestMove[0]][bestMove[1]] = 'O';
    player.value = 'X';
  }
};

// Function to find a blocking move for the given player ('X' or 'O')
const FindBlockingMove = playerSymbol => {
  const emptyCells = [];
  for (let x = 0; x < 3; x++) {
    for (let y = 0; y < 3; y++) {
      if (board.value[x][y] === '') {
        const tempBoard = [...board.value.map(row => [...row])];
        tempBoard[x][y] = playerSymbol;
        if (CalculateWinner(tempBoard.flat()) === playerSymbol) {
          return [x, y];
        }
        emptyCells.push([x, y]);
      }
    }
  }
  return null;
};

// Minimax algorithm for unbeatable AI (player 'O')
const Minimax = (boardState, currentPlayer) => {
  const availableMoves = [];
  for (let x = 0; x < 3; x++) {
    for (let y = 0; y < 3; y++) {
      if (boardState[x][y] === '') {
        availableMoves.push([x, y]);
      }
    }
  }

  const winner = CalculateWinner(boardState.flat());
  if (winner === 'O') return { score: 1 };
  if (winner === 'X') return { score: -1 };
  if (availableMoves.length === 0) return { score: 0 }; // Draw

  const moves = [];
  availableMoves.forEach(move => {
    const newBoard = boardState.map(row => [...row]);
    newBoard[move[0]][move[1]] = currentPlayer;

    const nextPlayer = currentPlayer === 'O' ? 'X' : 'O';
    const result = Minimax(newBoard, nextPlayer);

    moves.push({
      position: move,
      score: currentPlayer === 'O' ? result.score : -result.score,
    });
  });

  const bestMove = moves.reduce(
    (best, move) => (move.score > best.score ? move : best),
    { score: -Infinity }
  );
  return bestMove;
};

// Reset the game
const ResetGame = () => {
  board.value = [
    ['', '', ''],
    ['', '', ''],
    ['', '', ''],
  ];
  player.value = 'X';
};

// Watch winner to prevent AI from playing after game ends
watch(winner, newWinner => {
  if (newWinner) {
    console.log(`Game Over! Player ${newWinner} won.`);
  }
});
</script>

<template>
  <main class="pt-8 text-center dark:bg-gray-800 min-h-screen dark:text-white">
    <h1 class="mb-8 text-3xl font-bold uppercase">Tic Tac Toe</h1>

    <h3 class="text-xl mb-4">Player {{ player }}'s turn</h3>

    <div class="flex flex-col items-center mb-8">
      <div v-for="(row, x) in board" :key="x" class="flex">
        <div
          v-for="(cell, y) in row"
          :key="y"
          @click="MakeMove(x, y)"
          :class="`border border-white w-20 h-20 hover:bg-gray-700 flex items-center justify-center material-icons-outlined text-4xl cursor-pointer ${
            cell === 'X' ? 'text-pink-500' : 'text-blue-400'
          }`"
        >
          {{ cell === 'X' ? 'close' : cell === 'O' ? 'circle' : '' }}
        </div>
      </div>
    </div>

    <h2 v-if="winner" class="text-6xl font-bold mb-8">
      Player '{{ winner }}' wins
    </h2>

    <button
      @click="ResetGame"
      class="px-4 py-2 bg-green-500 rounded uppercase font-bold hover:bg-green-700 duration-300 mb-4"
    >
      Reset Game
    </button>

    <div class="mb-4">
      <label for="difficulty" class="mr-2">Choose AI Difficulty:</label>
      <select
        id="difficulty"
        v-model="aiDifficulty"
        class="bg-gray-700 text-white rounded p-2"
      >
        <option value="easy">Easy</option>
        <option value="medium">Medium</option>
        <option value="hard">Hard</option>
      </select>
    </div>

    <button
      @click="isAiPlaying = !isAiPlaying"
      class="px-4 py-2 bg-blue-500 rounded uppercase font-bold hover:bg-blue-700 duration-300"
    >
      Play {{ isAiPlaying ? 'vs Human' : 'vs AI' }}
    </button>
  </main>
</template>

<style scoped></style>
