<template>
    <div class="container">
      <div class="form-check form-switch switch">
        <input
          class="form-check-input"
          type="checkbox"
          role="switch"
          id="flexSwitchCheckDefault"
          v-model="isComputer"
          :disabled="usedBoxes > 0"
        />
        <label class="form-check-label mx-2" for="flexSwitchCheckDefault"
          >Computer</label
        >
      </div>

  <!-- <transition-group
    name="custom-classes-transition"
    enter-active-class="animated bounceIn"
    leave-active-class="animated bounceOut"
  > -->
      <h1 class="my-2" v-if="!winner">
        <img src="../assets/logo.png" lazy alt="Logo" width="40" /> Tic Tac Toe
      </h1>

      <h4
        class="p-2 alert-warning d-inline-block rounded-2"
        v-if="!isComputer && !winner"
      >
        Player Move : "{{ player }}"
      </h4>
      <h1 v-if="winner == 'draw'" class="alert alert-warning">Its A Draw!</h1>
      <h1
        class="text-center mx-2 alert alert-success"
        :class="[
          isComputer && winner === 'O' ? 'alert-danger' : 'alert success',
        ]"
        v-if="winner != 'draw' && winner"
      >
        <span v-if="!isComputer">
          <span v-if="winner != 'draw'"> Player {{ winner }} wins! </span>
        </span>
        <span v-else>
          <strong v-if="winner == 'X'">You</strong>
          <strong v-else>Computer</strong>
          Wins!
        </span>
      </h1>
      <!-- <h1 v-if="!winner"></h1> -->

      <div class="board d-block margin-auto">
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
      </div>

      <div class="d-block mx-auto">
        <div class="btn btn-block btn-outline-danger my-2" @click="reset">
          Reset
        </div>
      </div>

      <footer class="mt-3">
        <div class="d-flex justify-content-around">
          <h5 class="p-2 rounded-pill text-primary font-weight-bolder">
            <span v-if="isComputer">You</span> <span v-else>Player X</span>:
            <span class="font-weight-bold">{{ playerXPoints }}</span>
          </h5>
          <h5
            class="p-2 rounded-pill d-block font-weight-bold text-warning"
          >
            <span>Round : </span>
            <span class="font-weight-bold">{{ round }}</span>
          </h5>
          <h5 class="p-2 rounded-pill text-danger">
            <span v-if="isComputer">Computer</span>
            <span v-else>Player Y</span> :
            <span class="font-weight-bold">{{ playerOPoints }}</span>
          </h5>
        </div>
      </footer>
  <!-- </transition-group> -->
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
      isComputer: true,
      usedBoxes: computed(() => {
        return squares.value.flat().filter((x) => x !== "").length;
      }),
      leftBoxes: computed(() => {
        return squares.value.flat().filter((x) => x === "").length;
      }),
      playerXPoints: 0,
      playerOPoints: 0,
      round: 1,
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
        if (utils.isComputer) handleNextPlayer(); //!Running Computer Move
      }
    };

    const handleNextPlayer = () => {
      if (utils.player === "O" && utils.usedBoxes < 9) {
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
        if (square[a] === "" && square[b] === "X" && square[c] === "X") {
          return a;
        } else if (square[b] === "" && square[a] === "X" && square[c] === "X") {
          return b;
        } else if (square[c] === "" && square[a] === "X" && square[b] === "X") {
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
        } while (square[choice] !== "");
      }
      square[choice] = utils.player;
      const newSquares = [];
      while (square.length) newSquares.push(square.splice(0, 3));
      console.log({ newSquares });
      playerSwap(); //!Swap the player to 'X' again
      return newSquares;
    };

    const updatePlayerPoints = (winner) => {
      if (winner) {
        if (winner === "X") {
          utils.playerXPoints = utils.playerXPoints + 1;
        } else if (winner === "O") {
          utils.playerOPoints = utils.playerOPoints + 1;
        }
        utils.round = utils.round + 1;
      }
    };

    const winner = computed(() => {
      const win = findWinner(squares.value.flat());
      if (!win && utils.usedBoxes === 9) {
        updatePlayerPoints("draw");
        return "draw";
      }
      updatePlayerPoints(win);
      return win;
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
@import url("https://fonts.googleapis.com/css2?family=Kanit:wght@500&family=Varela+Round&display=swap");
.board {
  max-width: 300px;
  display: block;
  margin: auto;
}
:root{
  animation-duration: 1s;
}
h1 {
  font-family: "Varela Round", sans-serif;
  font-size: 2rem;
}

.alert {
  padding: 6px !important;
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
  font-family: "Varela Round", sans-serif;
}
.square:hover {
  background: #eee;
}
.activeBox {
  background: skyblue;
}

.switch {
  display: flex;
  justify-content: center;
  font-size: 29px;
  cursor: pointer !important;
}
input {
  cursor: pointer;
}
h5,
h4,
h3 {
  font-family: "Kanit", sans-serif;
}
</style>