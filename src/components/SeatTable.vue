<template>
  <div>
    <div class="wrapper">
      <v-card
        class="pl-3 mx-auto"
        :width="formation[0].length*40"
        flat
        max-width="none"
        max-height="50vh"
      >
        <v-row v-for="(row, i) in formation" :key="i">
          <v-col v-for="(cell, j) in row" :key="j" class="seat-col">
            <v-card
              tile
              flat
              class="text-center"
              :color="backgroundColor(cell, i * row.length + j)"
              dark
              @click="$emit('cellclicked', i, j)"
            >{{ i * row.length + j + 1}}</v-card>
          </v-col>
        </v-row>
      </v-card>
    </div>
  </div>
</template>

<script>
const fillstyles = ["#b2bec3", "#e84393", "#00cec9"];
const textfillstyles = ["#808080", "#ffffff"];
export default {
  props: {
    formation: Array,
    seats: Object
  },
  methods: {
    backgroundColor(cell, index) {
      const colorIndex = this.seats[index] ? 0 : cell;
      return fillstyles[colorIndex];
    },
    textColor(cell) {
      return textfillstyles[cell === 0 ? 0 : 1];
    }
  }
};
</script>

<style scoped>
td {
  width: 20px;
  height: 20px;
  position: relative;
  text-align: center;
}
.fukidashi {
  white-space: nowrap;
  bottom: 30px;
  left: 0px;
  position: absolute;
  padding: 5px;
  border-radius: 2px;
  background: #f563c4;
  color: #fff;
}
.fukidashi:after {
  position: absolute;
  width: 0;
  height: 0;
  left: 0;
  bottom: -19px;
  margin-left: 1px;
  border: solid transparent;
  border-color: rgba(51, 204, 153, 0);
  border-top-color: #f563c4;
  border-width: 10px;
  pointer-events: none;
  content: " ";
}
.seat-col {
  padding: 2px;
}
.wrapper {
  overflow: auto;
}
</style>
