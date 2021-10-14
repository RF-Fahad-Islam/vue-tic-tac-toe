<template>
  <div class="home">
    <Board />
  </div>
</template>

<script>
// @ is an alias to /src
import Board from "@/components/Board.vue";

export default {
  name: "Home",
  components: {
    Board,
  },
};
</script>
