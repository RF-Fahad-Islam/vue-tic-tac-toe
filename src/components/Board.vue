<template>
  <div class="container">
    <h1 class="text-center mx-2" v-if="winner">Player {{ winner }} wins!</h1>
    <!-- <h1 v-if="!winner"></h1> -->
    <h1>You VS Computer</h1>
    <div class="board d-block margin-auto">
      <transition-group
        name="custom-classes-transition"
        enter-active-class="animated tada"
        leave-active-class="animated bounceOutRight" tag="div"
      >
        <div v-for="(row, x) in squares" :key="x" class="row">
          <div
            v-for="(square, y) in row"
            :key="y"
            @click="move(x, y)"
            class="col square"
          >
            {{ square }}
          </div>
        </div>
      </transition-group>
    </div>

    <div class="d-block">
      <div class="btn btn-block btn-outline-danger my-2" @click="reset">
        Reset
      </div>
    </div>
  </div>
</template>

<script>
import { reactive, toRefs, ref, computed } from "vue";
export default {
  name: "Board",
  setup() {
    const squares = ref([
      ["", "", ""],
      ["", "", ""],
      ["", "", ""],
    ]);

    const utils = reactive({
      player: "X",
      gameEnd: computed(() => {
        return Boolean(winner.value);
      }),
      color: computed(() => {
        return utils.gameEnd ? "#eee" : "white";
      }),
    });

    const winConditions = [
      //Vertically- rows
      [0, 1, 2],
      [3, 4, 5],
      [6, 7, 8],
      //Horizontally- columns
      [0, 3, 6],
      [1, 4, 7],
      [2, 5, 8],
      //Corner
      [0, 4, 8],
      [2, 4, 6],
    ];

    const move = (x, y) => {
      if (squares.value[x][y] === "" && !utils.gameEnd) {
        squares.value[x][y] = utils.player;
        playerSwap(); //!Swap the player to "O"
        executeComputerChoice(); //!Running Computer Move
      }
    };

    const executeComputerChoice = () => {
      if (utils.player === "O" && squares.value.filter(x=> x!=="").length < 9) {
        const newSquares = computerChoice(squares.value.flat());
        squares.value = newSquares;
      }
    };

    //Swap player between 'X' and 'O'
    const playerSwap = () => {
      utils.player = utils.player === "X" ? "O" : "X";
    };

    const smartChoice = (square) => {
      //? Making the computer understand the action based on the positions and user actions
      for (let conditions of winConditions) {
        const [a, b, c] = conditions;
        if (square[a]==="" && square[b]==="X" && square[c]==="X") {
          return a;
        } else if (square[b]==="" && square[a]==="X" && square[c]==="X") {
          return b;
        } else if (square[c]==="" && square[a]==="X" && square[b] === "X") {
          return c;
        }
      }
      return null;
    };

    const computerChoice = (square) => {
      let choice;
      choice = smartChoice(square);
      if (choice === null) {
        do {
          choice = Math.floor(Math.random() * 9); //Random number between 0-8
        } while (!square[choice] && square.filter(x=> x!=="").length === 8);
      }
      square[choice] = utils.player;
      const newSquares = [];
      while (square.length) newSquares.push(square.splice(0, 3));
      console.log({ newSquares });
      playerSwap(); //!Swap the player to 'X' again
      return newSquares;
    };

    const winner = computed(() => {
      const win = findWinner(squares.value.flat());
      if (!win && squares.value.filter(x=> x!=="").length === 9) {
        return "draw" 
      }
      return win
    });

    const reset = () => {
      utils.player = "X";
      squares.value = [
        ["", "", ""],
        ["", "", ""],
        ["", "", ""],
      ];
    };

    //* find winner along with the winConditions
    //* Returns the winner 'X' or 'O'
    const findWinner = (square) => {
      for (const conditions of winConditions) {
        const [a, b, c] = conditions;
        if (square[a] && square[a] === square[b] && square[a] === square[c]) {
          return square[a]; // Returns player that matches all conditions
        }
      }
      return "";
    };

    return {
      ...toRefs(utils),
      move,
      squares,
      winner,
      reset,
    };
  },
};
</script>

<style>
.board {
  max-width: 300px;
  display: block;
  margin: auto;
}

.square {
  color: black;
  width: 100px;
  height: 100px;
  border: 2px dashed black;
  display: inline-block;
  margin: 0px;
  padding: 0px;
  font-size: 68px;
  cursor: pointer;
  background: v-bind(color);
}
.square:hover {
  background: #eee;
}
.activeBox {
  background: skyblue;
}
</style>