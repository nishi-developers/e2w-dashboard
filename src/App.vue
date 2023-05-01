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
              :loading="loading"
              :failed="failed"
              :failedMessage="failedMessage"
              @failedDone="failed = false"
              @addPerformance="
            (performanceName, date, formation) => {
              makePerformance(performanceName, date, formation);
            }
          "
              @setReservation="
            (item, seatNumber, userId) => {
              setReservation(item, seatNumber, userId);
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
        host: "",
        token: ""
      },
      collectionName: "psychopath",
      performanceList: [],
      logined: false,
      failed: false,
      failedMessage: "",
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
      this.loginData.host = "";
      this.loginData.token = "";
      this.failed = false;
      this.logined = false;
    },
    makePerformance(performanceName, date, formation) {
      this.loading = true;
      const xhr = new XMLHttpRequest();
      xhr.open(
        "POST",
        "https://asia-northeast1-" +
          this.loginData.host +
          ".cloudfunctions.net/makePerformance"
      );
      xhr.onload = () => {
        console.log(xhr.response);
        if (xhr.response === "success") {
          this.update();
        } else {
          this.failedMessage = xhr.response;
          this.failed = true;
        }
      };
      xhr.send(
        JSON.stringify({
          collectionName: this.collectionName,
          token: this.loginData.token,
          performanceName: performanceName,
          formation: JSON.stringify(formation),
          date: date
        })
      );
    },
    setReservation(performanceName, id, seatNumber) {
      this.loading = true;
      const xhr = new XMLHttpRequest();
      xhr.open(
        "POST",
        "https://asia-northeast1-" +
          this.loginData.host +
          ".cloudfunctions.net/setReservation2"
      );
      xhr.onload = () => {
        console.log(xhr.response);
        if (xhr.response === "success") {
          this.update();
        } else {
          this.failedMessage = xhr.response;
          this.failed = true;
        }
      };
      xhr.send(
        JSON.stringify({
          collectionName: this.collectionName,
          token: this.loginData.token,
          performanceName: performanceName,
          seatNumber,
          seat: { type: "line", id }
        })
      );
    },
    getPerformanceList() {
      return new Promise((resolve, reject) => {
        const xhr = new XMLHttpRequest();
        xhr.open(
          "POST",
          "https://asia-northeast1-" +
            this.loginData.host +
            ".cloudfunctions.net/getPerformanceList"
        );
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
            collectionName: this.collectionName,
            token: this.loginData.token
          })
        );
      });
    }
  }
};
</script>

<style></style>
