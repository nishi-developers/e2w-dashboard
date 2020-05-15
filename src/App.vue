<template>
  <div id="app">
    <v-app>
      <div v-if="logined">
        <v-app-bar app color="secondary" dark>
          <v-toolbar-title class="ml-2">サイコの予約</v-toolbar-title>

          <v-spacer></v-spacer>

          <v-btn light @click="logout()">ログアウト</v-btn>
        </v-app-bar>
        <v-content>
          <v-container fluid>
            <main-card
              :performanceList="performanceList"
              @addPerformance="
            (performanceName, date, formation) => {
              this.makePerformance(performanceName, date, formation);
              this.update();
            }
          "
              @setReservation="
            (item, seatNumber, userId) => {
              if (!setReservation(item, seatNumber, userId)) update();
            }
          "
            ></main-card>
          </v-container>
        </v-content>
      </div>
      <login v-else v-model="loginData" @done="update()" :failed="failed" :loading="loading"></login>
    </v-app>
  </div>
</template>

<script>
const host = "https://asia-northeast1-easy-to-wait.cloudfunctions.net/";
import Vue from "vue";
import Vuetify from "vuetify";
import "vuetify/dist/vuetify.min.css";
Vue.use(Vuetify);
import MainCard from "./components/MainCard.vue";
import Login from "./components/Login.vue";
export default {
  name: "app",
  components: {
    MainCard,
    Login
  },
  data() {
    return {
      loginData: {
        collectionName: "",
        token: ""
      },
      performanceList: [],
      valid: false,
      collectionNameRules: [v => !!v || "CollectionName is required"],
      tokenRules: [v => !!v || "Token is required"],
      password: "Password",
      rules: {
        required: value => !!value || "Required."
      },
      right: null,
      logined: false,
      failed: false,
      loading: false
    };
  },
  methods: {
    update() {
      this.loading = true;
      this.getPerformanceList()
        .then(() => {
          this.logined = true;
          this.loading = false;
        })
        .catch(() => {
          this.failed = true;
          this.loading = false;
        });
    },
    logout() {
      this.loginData.collectionName = "";
      this.loginData.token = "";
      this.failed = false;
      this.logined = false;
    },
    makePerformance(performanceName, date, formation) {
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
            collectionName: this.loginData.collectionName,
            token: this.loginData.token,
            performanceName: performanceName,
            formation: JSON.stringify(formation),
            date: date
          })
        );
      }
    },
    setReservation(performanceName, id, seatNumber) {
      let flag = false;
      if (id === "") {
        alert("UserIdを入力してください");
        flag = true;
      }
      if (!flag) {
        const xhr = new XMLHttpRequest();
        xhr.open("POST", host + "setReservation2");
        xhr.onload = () => {
          console.log(xhr.response);
        };
        xhr.send(
          JSON.stringify({
            collectionName: this.loginData.collectionName,
            token: this.loginData.token,
            performanceName: performanceName,
            seatNumber,
            seat: { type: "line", id }
          })
        );
      }
      return flag;
    },
    getPerformanceList() {
      return new Promise((resolve, reject) => {
        const xhr = new XMLHttpRequest();
        xhr.open("POST", host + "getPerformanceList");
        xhr.onload = () => {
          console.log(xhr.response);
          try {
            this.performanceList = JSON.parse(xhr.response);
          } catch (e) {
            reject();
          }
          resolve();
        };
        xhr.send(
          JSON.stringify({
            collectionName: this.loginData.collectionName,
            token: this.loginData.token
          })
        );
      });
    }
  }
};
</script>

<style></style>
