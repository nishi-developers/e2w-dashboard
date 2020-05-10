<template>
  <div id="app">
    <v-app>
      <div v-if="logined">
        <!--<v-form v-model="valid">
          <v-container>
            <v-row>
              <v-col cols="12" md="5" sm="5">
                <v-text-field
                  v-model="loginData.collectionName"
                  :rules="[rules.required]"
                  label="CollectionName"
                  required
                ></v-text-field>
              </v-col>
              <v-col cols="12" md="5" sm="5">
                <v-text-field
                  v-model="loginData.token"
                  :rules="[rules.required]"
                  :type="password"
                  label="Token"
                ></v-text-field>
              </v-col>
              <v-col cols="12" md="2" sm="2">
                <v-btn block @click="update()" height="50px">OK</v-btn>
              </v-col>
            </v-row>
          </v-container>
        </v-form>-->

        <!--カード!-->
        <main-card
          :info="info"
          @addPerformance="
            (performanceName, date, formation) => {
              this.makePerformance(performanceName, date, formation);
              this.update();
            }
          "
          @setReservation="
            (item, seatNumber, userId) => {
              this.info.seatNumber = seatNumber;
              this.info.userId = userId;
              if (!setReservation(item)) update();
            }
          "
        ></main-card>
      </div>
      <login v-else v-model="loginData" @done="update()"></login>
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
      info: {
        formations: [],
        seats: [],
        seatNumber: 0,
        userId: "",
        formation_str: "",
        performanceList: []
      },
      valid: false,
      collectionNameRules: [v => !!v || "CollectionName is required"],
      tokenRules: [v => !!v || "Token is required"],
      password: "Password",
      rules: {
        required: value => !!value || "Required."
      },
      right: null,
      logined: false
    };
  },
  methods: {
    update() {
      this.getPerformanceList().then(() => {
        let promises = [];
        for (let performanceName in this.info.performanceList) {
          promises.push(this.getPerformance(performanceName));
        }
        Promise.all(promises).then(() => {
          this.logined = true;
        });
      });
    },
    getPerformance(performanceName) {
      return new Promise(resolve => {
        const xhr = new XMLHttpRequest();
        xhr.open("POST", host + "getPerformance");
        xhr.onload = () => {
          console.log(xhr.response);
          const data = JSON.parse(xhr.response);
          this.info.formations[performanceName] =
            typeof data.formation === "undefined" || data.formation === ""
              ? []
              : JSON.parse(data.formation);
          this.info.seats[performanceName] = data.seats;
          this.info.seats.splice(); //This code is necessary to reflect changes
          resolve();
        };
        xhr.send(
          JSON.stringify({
            collectionName: this.loginData.collectionName,
            token: this.loginData.token,
            performanceName: performanceName
          })
        );
      });
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
    setReservation(performanceName) {
      let flag = false;
      if (this.info.userId === "") {
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
            seatNumber: this.info.seatNumber,
            seat: { type: "line", id: this.info.userId }
          })
        );
      }
      return flag;
    },
    getPerformanceList() {
      return new Promise(resolve => {
        const xhr = new XMLHttpRequest();
        xhr.open("POST", host + "getPerformanceList");
        xhr.onload = () => {
          console.log(xhr.response);
          this.info.performanceList = JSON.parse(xhr.response);
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
