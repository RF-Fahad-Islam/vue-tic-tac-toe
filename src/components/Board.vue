<template>
  <div class="container">
    <div class="form-check form-switch switch" v-show="!gameStart">
      <input
        class="form-check-input"
        type="checkbox"
        role="switch"
        id="flexSwitchCheckDefault"
        v-model="isComputer"
        :disabled="gameStart"
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

    <div
      v-if="gameStart&&isComputer"
      style="
        border: 2px solid skyblue;
        border-radius: 20px;
        padding: 10px 15px !important;
      "
      class="alert d-inline-block"
    >
      <h4  class="font-weight-bold text-black">
        <span>
          <i class="fa fa-desktop text-primary"></i> Computer in
          <i class="fa fa-dashboard text-black mx-1"></i>
             <strong class="text-decoration-underline">   {{ level }} </strong> mode
        </span>
      </h4>
    </div>

    <h1 class="my-2" v-if="!winner">
      <img src="../assets/logo.png" lazy alt="Logo" width="40" /> Tic Tac Toe
    </h1>

    <div
      v-if="!gameStart && isComputer"
      class="d-flex justify-content-around my-2"
    >
      <select
        name="level"
        v-model="level"
        id="level"
        class="form-select"
        style="width: 300px"
        :disabled="usedBoxes > 0"
      >
        <option value="hard" default>Choose Difficulty Level (Default Hard)</option>
        <option value="noob">Noob</option>
        <option value="easy">Easy</option>
        <option value="medium">Medium</option>
        <option value="hard">Hard</option>
      </select>
    </div>

    <h4
      class="p-2 alert-warning d-inline-block rounded-2"
      v-if="!isComputer && !winner"
    >
      Player Move : "{{ player }}"
    </h4>
    <h1 v-if="winner == 'draw'" :style="!isComputer?{fontSize: '50px'}:{}" class="alert alert-warning">Its A Draw!</h1>
    <h1
      class="text-center mx-2 alert alert-success"
      :class="[isComputer && winner === 'O' ? 'alert-danger' : 'alert success']"
      v-if="winner != 'draw' && winner"
    >
      <span v-if="!isComputer">
        <span v-if="winner != 'draw'" style="font-size: 50px;"> Player {{ winner }} wins! </span>
      </span>
      <span v-else
        >🎉
        <strong v-if="winner == 'X'">You</strong>
        <strong v-else>Computer</strong>
        Wins!
      </span>
    </h1>
    <!-- <h1 v-if="!winner"></h1> -->

    <div class="board d-block margin-auto">
      <div class="line"></div>
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
    <div>
      <div class="d-flex justify-content-around">
        <div
          style="border-radius: 16px"
          class="btn btn-block btn-outline-danger my-2 ml-2"
          @click="restart"
        >
          Restart <i class="fa fa-refresh"></i>
        </div>

        <div
          style="border-radius: 16px"
          class="btn btn-block btn-outline-success my-2"
          @click="next"
        >
          Next Round <i class="fa fa-arrow-right"></i>
        </div>
      </div>
    </div>
    <div class="mt-3 gameStatus">
      <div class="d-flex justify-content-around">
        <h5 class="p-2 rounded-pill text-primary font-weight-bolder">
          <span v-if="isComputer"><i class="fa fa-user"></i> You</span>
          <span v-else><i class="fa fa-user-circle"></i> Player X</span>:
          <span class="font-weight-bold">{{ playerXPoints }}</span>
        </h5>
        <h5 class="p-2 rounded-pill d-block font-weight-bold text-black">
          <span><i class="fa fa-gamepad"></i> Round : </span>
          <span class="font-weight-bold" style="font-weight: bold">{{
            round
          }}</span>
        </h5>
        <h5 class="p-2 rounded-pill text-danger">
          <span v-if="isComputer"><i class="fa fa-desktop"></i> Computer</span>
          <span v-else><i class="fa fa-user-circle"></i> Player Y</span> :
          <span class="font-weight-bold">{{ playerOPoints }}</span>
        </h5>
      </div>
    </div>
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
      level: "hard",
      isComputer: true,
      usedBoxes: computed(() => {
        return squares.value.flat().filter((x) => x !== "").length;
      }),
      leftBoxes: computed(() => {
        return squares.value.flat().filter((x) => x === "").length;
      }),
      gameStart: computed(()=> {
        return utils.usedBoxes>0||utils.round>1
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

    //Player is X or O and square is squares.value.flat()
    const findWinChances = (square, player)=> {
      for (let conditions of winConditions) {
          const [a, b, c] = conditions;
          if (square[a] === "" && square[b] === player && square[c] === player) return a;
          else if (square[b] === "" && square[a] === player && square[c] === player) return b;
          else if (square[c] === "" && square[a] === player && square[b] === player) return c;
        }
          return null
    }

    const smartChoice = (square, level) => {
      //? Making the computer understand the action based on the positions and user actions
        let returnPosition;
        //! Level Hard
        if(level==="hard"){
          //* Priotorize computer's win possibilities
          returnPosition = findWinChances(square, "O")
          //*If there is no chances for computer then find the user win possibilites and prevent from winning
          if(returnPosition == null) returnPosition = findWinChances(square, "X")
        }
        
        //! Level Medium
        else if (level==="medium") {
          //* Prevent the user from winning
          returnPosition = findWinChances(square, "X")
        }
        else if (level==="easy") {
          //* Only focuses on computer win. Don't prevent user from winning
          returnPosition = findWinChances(square, "O")
        }
        else if (level==="noob"){
          //* Randomly Make all Choices
          returnPosition = null
        }
        return returnPosition
    };

    const computerChoice = (square) => {
      let choice;
      choice = smartChoice(square, utils.level);
      if (choice === null) {
        do {
          choice = Math.floor(Math.random() * 9); //Random number between 0-8
        } while (square[choice] !== "");
      }
      square[choice] = utils.player;
      playerSwap(); //!Swap the player to 'X' again
      return array1dTo2d(square)
    };

    const array1dTo2d = (array)=> {
      const newArray = [];
      while (array.length) newArray.push(array.splice(0, 3));
      return newArray;
    }

    const updatePlayerPoints = (winner) => {
      if (winner) {
        if (winner === "X") {
          utils.playerXPoints = utils.playerXPoints + 1;
        } else if (winner === "O") {
          utils.playerOPoints = utils.playerOPoints + 1;
        }
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

    const restart = () => {
      if (confirm("Do you really want to restart the game?")) {
        utils.player = "X";
        utils.playerXPoints = 0;
        utils.playerOPoints = 0;
        utils.round = 1;
        squares.value = [
          ["", "", ""],
          ["", "", ""],
          ["", "", ""],
        ];
      }
    };

    const next = () => {
      if (winner.value) {
        utils.player = "X";
        squares.value = [
          ["", "", ""],
          ["", "", ""],
          ["", "", ""],
        ];
        utils.round += 1;
      } else {
        alert("Can't go to next round until finis current round.");
      }
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
      next,
      restart,
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
:root {
  animation-duration: 1s;
}
h1 {
  font-family: "Varela Round", sans-serif;
  font-size: 2rem;
}

.alert {
  padding: 8px !important;
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

.gameStatus {
  font-size: 30px !important;
}
</style>