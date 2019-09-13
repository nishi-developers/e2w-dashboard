<template>
  <div id="app">
    <v-app>
      <v-form v-model="valid">
        <v-container>
          <v-row>
            <v-col cols="12" md="5" sm="5">
              <v-text-field
                v-model="info.collectionName"
                :rules="[rules.required]"
                label="CollectionName"
                required
              ></v-text-field>
            </v-col>
            <v-col cols="12" md="5" sm="5">
              <v-text-field
                v-model="info.token"
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
      </v-form>

      <!--カード!-->
      <main-card
        :info="info"
        @addPerformance="() => {this.makePerformance();this.initEditForm();this.update();}"
        @setReservation="(item, seatNumber, userId) => {this.info.seatNumber=seatNumber;this.info.userId=userId;if(!setReservation(item))update();}"
      ></main-card>
      <br />
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
export default {
  name: "app",
  components: {
    MainCard
  },
  data() {
    return {
      info: {
        formations: [],
        seats: [],
        collectionName: "",
        token: "",
        performanceName: "",
        seatNumber: Number,
        userId: "",
        formation_str: "",
        performanceList: [],
        editing: {
          formation: [[]],
          width: 0,
          height: 0
        },
        date: Object
      },
      valid: false,
      collectionNameRules: [v => !!v || "CollectionName is required"],
      tokenRules: [v => !!v || "Token is required"],
      password: "Password",
      rules: {
        required: value => !!value || "Required."
      },
      menu: false,
      right: null
    };
  },
  methods: {
    update() {
      this.getPerformanceList().then(() => {
        for (let performanceName in this.info.performanceList) {
          this.getPerformance(performanceName);
        }
      });
    },
    initEditForm() {
      this.info.editing.formation = [
        [1, 1, 1, 1, 1],
        [1, 1, 1, 1, 1],
        [1, 1, 1, 1, 1]
      ];
      this.info.editing.width = 5;
      this.info.editing.height = 3;
      this.info.performanceName = "";
      this.info.date = new Date().toISOString().substr(0, 10);
    },
    getPerformance(performanceName) {
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
      };
      xhr.send(
        JSON.stringify({
          collectionName: this.info.collectionName,
          token: this.info.token,
          performanceName: performanceName
        })
      );
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
            collectionName: this.info.collectionName,
            token: this.info.token,
            performanceName: this.info.performanceName,
            formation: JSON.stringify(this.info.editing.formation),
            date: this.info.date
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
            collectionName: this.info.collectionName,
            token: this.info.token,
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
            collectionName: this.info.collectionName,
            token: this.info.token
          })
        );
      });
    }
  },
  mounted: function() {
    this.initEditForm();
  }
};
</script>

<style>
</style>