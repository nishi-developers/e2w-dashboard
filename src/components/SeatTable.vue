<template>
  <table @mouseleave="setActive(-1)">
    <tr v-for="(row, i) in formation">
      <td
        class="cell"
        v-for="(cell, j) in row"
        :style="{ backgroundColor: backgroundColor(cell, i * row.length + j) , color: textColor(cell)}"
        @mouseover="setActive(i * row.length + j)"
        @click="$emit('cellclicked', i, j)"
      >
        {{ i * row.length + j}}
        <div class="fukidashi" v-if="isActive(i * row.length + j)">{{ text(i * row.length + j) }}</div>
      </td>
    </tr>
  </table>
</template>

<script>
const fillstyles = ["#b2bec3", "#e84393", "#00cec9"];
const textfillstyles = ["#808080", "#ffffff"];
export default {
  props: {
    formation: Array,
    seats: Object
  },
  data: function() {
    return {
      activenum: -1
    };
  },
  methods: {
    text(index) {
      const seat = this.seats[index];
      if (seat) {
        return seat["id"];
      }
      return "";
    },
    backgroundColor(cell, index) {
      const colorIndex = this.seats[index] ? 0 : cell;
      return fillstyles[colorIndex];
    },
    textColor(cell) {
      return textfillstyles[cell === 0 ? 0 : 1];
    },
    isActive(index) {
      return this.text(index) !== "" && index === this.activenum;
    },
    setActive(index) {
      this.activenum = index;
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
</style>
