<template>
  <div id="app">
    <v-app>
      <v-form v-model="valid" ref="form">
        <v-container>
          <v-row>
            <v-col cols="12" md="6">
              <v-text-field
                v-model="collectionName"
                :rules="[rules.required]"
                label="CollectionName"
                required
              ></v-text-field>
            </v-col>
            <v-col cols="12" md="6">
              <v-text-field
                v-model="token"
                :rules="[rules.required]"
                :type="password"
                label="Token"
              ></v-text-field>
            </v-col>
          </v-row>
          <v-row>
            <v-col cols="12" md="3">
              <v-text-field
                v-model="performanceName"
                :rules="[rules.required]"
                label="PerformanceName"
                required
              ></v-text-field>
            </v-col>
            <v-col cols="12" md="3">
              <v-menu
                ref="menu"
                v-model="menu"
                :close-on-content-click="false"
                :return-value.sync="date"
                transition="scale-transition"
                offset-y
                full-width
                min-width="290px"
              >
                <template v-slot:activator="{ on }">
                  <v-text-field v-model="date" label="Date" readonly v-on="on"></v-text-field>
                </template>
                <v-date-picker v-model="date" no-title scrollable>
                  <div class="flex-grow-1"></div>
                  <v-btn text color="primary" @click="menu=false">Cancel</v-btn>
                  <v-btn text color="primary" @click="$refs.menu.save(date)">OK</v-btn>
                </v-date-picker>
              </v-menu>
            </v-col>
            <v-col cols="12" md="3">
              <v-text-field v-model.number="seatNumber" label="SeatNumber" type="number" min="0"></v-text-field>
            </v-col>
            <v-col cols="12" md="3">
              <v-text-field v-model="userId" label="UserID"></v-text-field>
            </v-col>
          </v-row>
          <v-textarea v-model="formation_str" label="Formation"></v-textarea>
          <v-row>
            <v-col cols="12" md="3">
              <v-btn @click="makePerformance()">makePerformance</v-btn>
            </v-col>
            <v-col cols="12" md="3">
              <v-btn @click="setReservation()">setReservation</v-btn>
            </v-col>
            <v-col cols="12" md="3">
              <v-btn @click="getPerformance()">getPerformance</v-btn>
            </v-col>
            <v-col cols="12" md="3">
              <v-btn @click="getPerformanceList()">getPerformanceList</v-btn>
            </v-col>
          </v-row>
          <seat-table :formation="formation" :seats="seats" />
        </v-container>
      </v-form>
    </v-app>
    <seat-table :formation="formation" :seats="seats" />
  </div>
</template>

<script>
const host = "https://asia-northeast1-easy-to-wait.cloudfunctions.net/";
import Vue from "vue";
import Vuetify from "vuetify";
import "vuetify/dist/vuetify.min.css";
Vue.use(Vuetify);
import SeatTable from "./components/SeatTable.vue";
export default {
  name: "app",
  components: {
    SeatTable
  },
  data() {
    return {
      formation: [],
      seats: {},
      valid: false,
      collectionName: "",
      collectionNameRules: [v => !!v || "CollectionName is required"],
      token: "",
      tokenRules: [v => !!v || "Token is required"],
      password: "Password",
      token: "",
      rules: {
        required: value => !!value || "Required."
      },
      performanceName: "",
      seatNumber: 0,
      userId: "",
      formation_str: "",
      date: new Date().toISOString().substr(0, 10),
      menu: false
    };
  },
  methods: {
    getPerformance() {
      if (this.$refs.form.validate()) {
      }
      if (this.performanceName === "") {
        alert("PerformanceNameを入力してください");
      } else {
        const xhr = new XMLHttpRequest();
        xhr.open("POST", host + "getPerformance");
        xhr.onload = () => {
          console.log(xhr.response);
          const data = JSON.parse(xhr.response);
          this.formation = JSON.parse(data.formation);
          this.seats = data.seats;
        };
        xhr.send(
          JSON.stringify({
            collectionName: this.collectionName,
            token: this.token,
            performanceName: this.performanceName
          })
        );
      }
    },
    makePerformance() {
      if (this.performanceName === "") {
        alert("PerformanceNameを入力してください");
      } else {
        const xhr = new XMLHttpRequest();
        xhr.open("POST", host + "makePerformance");
        xhr.onload = () => {
          console.log(xhr.response);
        };
        xhr.send(
          JSON.stringify({
            collectionName: this.collectionName,
            token: this.token,
            performanceName: this.performanceName,
            formation: this.formation_str,
            date: this.date
          })
        );
      }
    },
    setReservation() {
      let flag = false;
      if (this.performanceName === "") {
        alert("PerformanceNameを入力してください");
        flag = true;
      }
      if (this.userId === "") {
        alert("UserIdを入力してください");
        flag = true;
      }
      if (!flag) {
        const xhr = new XMLHttpRequest();
        xhr.open("POST", host + "setReservation");
        xhr.onload = () => {
          console.log(xhr.response);
        };
        xhr.send(
          JSON.stringify({
            collectionName: this.collectionName,
            token: this.token,
            performanceName: this.performanceName,
            seatNumber: this.seatNumber,
            seat: { type: "line", id: this.userId }
          })
        );
      }
    },
    getPerformanceList() {
      const xhr = new XMLHttpRequest();
      xhr.open("POST", host + "getPerformanceList");
      xhr.onload = () => {
        console.log(xhr.response);
      };
      xhr.send(
        JSON.stringify({
          collectionName: this.collectionName,
          token: this.token
        })
      );
    }
  }
};
</script>

<style>
</style>