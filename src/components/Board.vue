<template>
<div class="container">
    <!-- <span  style="cursor: pointer; font-size: 35px; position: absolute; left: 20px;" class="float-start" @click="isMuted = !isMuted">
    <i class="fa fa-volume-up" v-if="!isMuted"></i>
    <i class="fa fa-volume-off" v-else></i>
    </span> -->
  <header>
      <h1 class="my-2">
        <img src="../assets/logo.png" lazy alt="Logo" width="40" /> Tic Tac Toe
    <div class="form-check form-switch d-inline-block" id="toggle" style="font-size: 20px;" v-show="!gameStart">
      <input
        class="form-check-input"
        type="checkbox"
        role="switch"
        v-model="isRow"
      />
    </div>

      </h1>
  </header>

  <div :class="[isRow?'row':'text-center']">

  <!-- !Controls and Message Section -->
  <section :class="[isRow?'col-md-6 col-sm-12':'']">
      <!-- Computer Switch -->
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
        >Computer</label>
    </div>


    <transition-group
    name="custom-classes-transition"
    enter-active-class="animated bounceIn"
    leave-active-class="animated bounceOut"
    tag="ul"
  >
    <h1 class="text-black" v-if="gameStart">Round : {{round}}</h1>


      <h4 v-if="gameStart&&isComputer" style="
        border: 2px solid skyblue;
      " class="alert font-weight-bold text-black d-inline-block">
        <span>
          <i class="fa fa-desktop text-primary"></i> Computer in
          <i class="fa fa-dashboard text-black mx-1"></i>
             <strong class="text-decoration-underline">   {{ level }} </strong> mode
        </span>
      </h4>
    </transition-group>
    
    <div
      v-if="!gameStart && isComputer"
      class="d-flex justify-content-center my-2"
    >
      <select
        name="level"
        v-model="level"
        id="level"
        class="form-select"
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
      v-if="!isComputer"
    >
      Player Move : "{{ player }}"
    </h4>
 <!-- </transition-group> -->

    <h1 v-if="winner == 'draw'" :style="!isComputer?{fontSize: '50px'}:{}" class="alert alert-warning">Its A Draw!</h1>
    <h1
      class="text-center mx-2 alert alert-success"
      :class="[isComputer && winner === 'O' ? 'alert-danger' : 'alert success']"
      v-if="winner != 'draw' && winner"
    >
      <span v-if="!isComputer">
        <strong v-if="winner != 'draw'">🎉 Player {{ winner }} wins! </strong>
      </span>
      <span v-else
        >🎉
        <strong v-if="winner == 'X'">You</strong>
        <strong v-else>Computer</strong>
        Wins!
      </span>
    </h1>

    </section>

    <!-- !Game Board Section -->
  <section :class="[isRow?'col-md-6 col-sm-12':'']">

  <!-- !Main Game Board -->
    <div class="board d-block margin-auto">
      <div v-for="(row, x) in squares" :key="x" class="row">

        <div
          v-for="(square, y) in row"
          :key="y"
          @click="move(x, y)"
          class="col square"
          :class="[winBoxes[x][y]==='win'?'winBox':'', squareClasses[x][y]]"
        >
    <transition
    name="custom-classes-transition-{{x}}-{{y}}"
    enter-active-class="animated bounceIn"
    leave-active-class="animated bounceOut">
          {{ square }}
    </transition>
        </div>
      </div>
    </div>
    </section>
 
  </div>
    <!-- </transition-group> -->


      <section class="d-flex justify-content-around">
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
      </section>
    <!-- !Game Status - Points, Round -->
    <footer class="mt-3 gameStatus" >
      <div class="d-flex p-1 flex-wrap justify-content-around">
        <h5 class="rounded-pill text-primary font-weight-bolder">
          <span v-if="isComputer"><i class="fa fa-user"></i> You</span>
          <span v-else><i class="fa fa-user-circle"></i> Player X</span>:
          <span class="font-weight-bold">{{ playerXPoints }}</span>
        </h5>
        <!-- <h5 class="p-2 rounded-pill d-block font-weight-bold text-black">
          <span><i class="fa fa-gamepad"></i> Round : </span>
          <span class="font-weight-bold" style="font-weight: bold">{{
            round
          }}</span>
        </h5> -->
        <h5 class="rounded-pill text-danger">
          <span v-if="isComputer"><i class="fa fa-desktop"></i> Computer</span>
          <span v-else><i class="fa fa-user-circle"></i> Player Y</span> :
          <span class="font-weight-bold">{{ playerOPoints }}</span>
        </h5>
      </div>
    </footer>

  </div>
</template>

<script>
import { reactive, toRefs, ref, computed} from "vue";
export default {
  name: "Board",
  setup() {
    //* Initialize all sounds for the game
    const audio = new Audio("musics/ting.mp3");
 
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
        return utils.gameEnd ? "rgb(233,233,233)" : "none";
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
      isRow: true,
      playFont: computed(()=> {
        if (utils.isRow) return "75px"
        else return "75px"
      }),
      squareSize: computed(()=> {
        if(utils.isRow) return "120px"
        else return "100px"
      }),
      winBoxes : [
        ["","",""],
        ["","",""],
        ["","",""]
      ],
      conditionThatMatched : "",  
      //Classes for squares
      squareClasses: [
        ["top left","top middle","top right"],
        ["left", "middle","right"],
        ["bottom left","bottom middle","bottom right"]
      ],
      soundsPath: "musics",
      //Marking color to mark the win conditions used in css
      markingColor: computed(()=> {
          return "#ff0111" //Red
      }),
      isMuted: false,
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

    //Play Sound Method
    const playSounds = async (action) => {
      if(action==="move"){
        audio.play();
      }
    }

     const winner = computed(() => {
      const win = findWinner(squares.value.flat());
      if (!win && utils.usedBoxes === 9) {
        updatePlayerPoints("draw");
        return "draw";
      }
      if(win==="X"||win==="O") updateWinBoxes(utils.winBoxes.flat())
      if(win) playSounds("move")
      updatePlayerPoints(win);
      return win;

    });

    //* For marking the boxes which matches the win conditions
    //* Take utils.winBoxes.flat() as argument
    const updateWinBoxes = (winBoxesFlat)=> {
      utils.conditionThatMatched.forEach((c)=> {
        winBoxesFlat[c] = "win"
      })
      utils.conditionThatMatched = "" //Empty the conditions
      utils.winBoxes = array1dTo2d(winBoxesFlat)
    }
    
    const move = (x, y) => {
      if (squares.value[x][y] === "" && !utils.gameEnd) {
        squares.value[x][y] = utils.player;
        playerSwap(); //!Swap the player to "O"
        if (utils.isComputer&&!winner.value) executeComputer(); //!Running Computer Move
      }
    };

    const executeComputer = () => {
      if (utils.player === "O" && utils.usedBoxes < 9) {
        const newSquares = computerChoice(squares.value.flat());
        squares.value = newSquares;
        // playSounds("move")
      }
    };

    //* Swap player between 'X' and 'O'
    const playerSwap = () => {
      utils.player = utils.player === "X" ? "O" : "X";
    };

     //* Generating Computer choice
     //! Dealing with flat array of 2D squares array
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

    //* Player is X or O and square is squares.value.flat()
    const findWinChances = (square, player)=> {
      for (let conditions of winConditions) {
          const [a, b, c] = conditions;
          if (square[a] === "" && square[b] === player && square[c] === player) return a;
          else if (square[b] === "" && square[a] === player && square[c] === player) return b;
          else if (square[c] === "" && square[a] === player && square[b] === player) return c;
      }
          return null
    }

    //? Making the computer understand the action based on the positions and user actions
    const smartChoice = (square, level) => {
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

    //* For converting squares flat array to 2D again
    const array1dTo2d = (array)=> {
      const newArray = [];
      while (array.length) newArray.push(array.splice(0, 3));
      return newArray;
    }

    //* Update the points
    const updatePlayerPoints = (winner) => {
      if (winner) {
        if (winner === "X") {
          utils.playerXPoints = utils.playerXPoints + 1;
        } else if (winner === "O") {
          utils.playerOPoints = utils.playerOPoints + 1;
        }
      }
    };

    //* find winner along with the winConditions
    //* Returns the winner 'X' or 'O'
    const findWinner = (square) => {
      for (const conditions of winConditions) {
        const [a, b, c] = conditions;
        if (square[a] && square[a] === square[b] && square[a] === square[c]) {
          utils.conditionThatMatched = [a,b,c] //For accessing the boxes
          return square[a]; // Returns player that matches all conditions
        }
      }
      return "";
    };

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
      utils.winBoxes = [
        ["", "", ""],
        ["", "", ""],
        ["", "", ""],
      ]
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
        utils.winBoxes = [
          ["", "", ""],
          ["", "", ""],
          ["", "", ""],
        ]
        utils.round += 1;
      } else {
        alert("Can't go to next round until finis current round.");
      }
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
  max-width: fit-content;
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
  border-radius: 30px!important;
}

.square {
  color: black;
  width: v-bind(squareSize);
  height: v-bind(squareSize);
  border: 2px solid black;
  border-radius: 2px;
  display: inline-block;
  margin: 0px;
  padding: 0px;
  font-size: v-bind(playFont);
  text-align: center;
  cursor: pointer;
  background: v-bind(color);
  font-family: "Varela Round", sans-serif;
}
.square:hover {
  background: rgb(233,233,233);
}

.winBox {
  color: v-bind(markingColor);
  background: rgb(255, 255, 255)!important;
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
footer {
  border: 2px dashed rgb(68, 68, 68);
  border-radius: 30px;
}

#level{
  width: 330px;
}

/* Squares Classes */
.top{
  border-top: none!important;
}

.bottom{
  border-bottom: none!important;
}

.left{
  border-left: none!important;
}

.right{
  border-right: none!important;;
}

@media screen and (max-width: 766px) {
  #toggle{
    display: none;
  }  
}
@media screen and (max-width: 370px) {
  .square {
    width: 90px!important;
    height: 90px!important;
    font-size: 60px!important;
  }
  #level {
    width: 250px;
  }
}
</style>