<template>
  <div id="app">
    <form name="form">
    CollectionName
    <input type="text" name="collectionName">
    <br>
    Token
    <input type="password" name="token">
    <br>
    <br>
    PerformanceName
    <input type="text" name="performanceName">
    <br>
    SeatNumber
    <input type="number" name="seatNumber" min="1">
    <br>
    UserId
    <input type="text" name="userId">
    <br>
    <br>
    Formation<br>
    <textarea name="formation"></textarea>
    </form>
    <br>
    <button id="makePerformance">makePerformance</button>
    <button id="setReservation">setReservation</button>
    <button id="getPerformance">getPerformance</button>
    <button id="getPerformanceList">getPerformanceList</button>
    <br>
    <br>
    <div id="table"></div>
    <seat-table :formation="formation" :seats="seats" />
  </div>
</template>

<script>
import SeatTable from './components/SeatTable.vue'

export default {
  name: 'app',
  components: {
    SeatTable
  },
  data() {
    return {
      formation: [],
      seats: {}
    }
  },
  mounted() {
    document.getElementById('getPerformance').addEventListener('click', () => {
      var collectionName = document.form.collectionName.value
      var token = document.form.token.value
      var performanceName = document.form.performanceName.value
      if(performanceName === ''){
        alert("PerformanceNameを入力してください")
      }else {
        const xhr = new XMLHttpRequest()
        xhr.open('POST', host + 'getPerformance')
        xhr.onload = () => {
          console.log(xhr.response)
          const data = JSON.parse(xhr.response)
          var formation = JSON.parse(data.formation)
          this.formation = formation
          this.seats = data.seats
        }
        xhr.send(JSON.stringify({
          collectionName,
          token,
          performanceName
        }))
      }
    })
  }
}

// non-vue code below

const host = 'https://asia-northeast1-easy-to-wait.cloudfunctions.net/'
document.addEventListener('DOMContentLoaded', () => {
  document.getElementById('makePerformance').addEventListener('click', () => {
    var collectionName = document.form.collectionName.value
    var token = document.form.token.value
    var performanceName = document.form.performanceName.value
    var formation = document.form.formation.value
    if(performanceName === ''){
      alert("PerformanceNameを入力してください")
    }else {
      const xhr = new XMLHttpRequest()
      xhr.open('POST', host + 'makePerformance')
      xhr.onload = () => {
        console.log(xhr.response)
      }
      xhr.send(JSON.stringify({
        collectionName,
        token,
        performanceName,
        formation
      }))
    }
  })
  document.getElementById('setReservation').addEventListener('click', () => {
    var collectionName = document.form.collectionName.value
    var token = document.form.token.value
    var performanceName = document.form.performanceName.value
    var seatNumber_str = document.form.seatNumber.value
    var userId = document.form.userId.value
    var flag = false
    if(performanceName === ''){
      alert("PerformanceNameを入力してください")
      flag = true
    }
    if(seatNumber_str === ''){
      alert("SeatNumberを入力してください")
      flag = true
    }else if(isNaN(seatNumber_str)){
      alert("無効なSeatNumberです")
      flag = true
    }
    if (userId === ''){
      alert("UserIdを入力してください")
      flag = true
    }else if(isNaN(userId)){
      alert("無効なUserIdです")
      flag = true
    }
    var seatNumber = parseInt(seatNumber_str)
    if(!flag){
      const xhr = new XMLHttpRequest()
      xhr.open('POST', host + 'setReservation')
      xhr.onload = () => {
        console.log(xhr.response)
      }
      xhr.send(JSON.stringify({
        collectionName,
        token,
        performanceName,
        seatNumber,
        userId
      }))
    }
  })
  document.getElementById('getPerformanceList').addEventListener('click', () => {
    var collectionName = document.form.collectionName.value
    var token = document.form.token.value
    const xhr = new XMLHttpRequest()
    xhr.open('POST', host + 'getPerformanceList')
    xhr.onload = () => {
      console.log(xhr.response)
    }
    xhr.send(JSON.stringify({
      collectionName,
      token,
    }))
  })
  const fillStyles = [
    '#b2bec3',
    '#e84393',
    '#00cec9'
  ]
  function makeTable(data, seats, tableId){
    var obj = document.getElementById(tableId)
    obj.innerHTML = ''
    var table = document.createElement("table")
    table.style.width = (30*data[0].length).toString() + "px"
    for(let i = 0; i < data.length; i++){
        let row = table.insertRow(-1);
        for(let j = 0; j < data[0].length; j++){
            let cell = row.insertCell(-1)
            const userId = seats[(i*data[0].length+j+1).toString()]
            let colorIndex
            if(typeof userId !== 'undefined'){
              colorIndex = 0
              cell.textContent = userId
            }else {
              colorIndex = data[i][j]
              cell.textContent = "\u00a0"
            }
            cell.style.backgroundColor = fillStyles[colorIndex]
        }
    }
    obj.appendChild(table)
  }
})
</script>

<style>
#app {
}
</style>
