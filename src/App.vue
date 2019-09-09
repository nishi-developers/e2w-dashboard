<template>
  <div id="app">
    <v-app>
      <v-form v-model="valid">
        <v-container>
          <v-row>
            <v-col cols="12" md="5" sm="5">
              <v-text-field
                v-model="collectionName"
                :rules="[rules.required]"
                label="CollectionName"
                required
              ></v-text-field>
            </v-col>
            <v-col cols="12" md="5" sm="5">
              <v-text-field
                v-model="token"
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
      <v-card width="90%" class="mx-auto">
        <v-tabs vertical>
          <v-tab
            class="text-left"
            v-for="item in Object.keys(this.performanceList)"
            :key="item"
            link
          >
            <!--<v-icon left>mdi-account</v-icon>!-->
            {{item}}
          </v-tab>
          <v-tab-item v-for="item in Object.keys(this.performanceList)" :key="item">
            <v-card flat>
              <seat-table class="mx-auto mt-10" :formation="formations[item]" :seats="seats[item]" />
              <v-container width="90%" class="mx-auto">
                <v-row class="mt-5">
                  <v-col cols="12" sm="5" md="5">
                    <v-text-field
                      v-model.number="seatNumber"
                      label="SeatNumber"
                      type="number"
                      min="0"
                    ></v-text-field>
                  </v-col>
                  <v-col cols="12" sm="5" md="5">
                    <v-text-field v-model="userId" label="UserID"></v-text-field>
                  </v-col>
                  <v-col cols="12" sm="2" md="2">
                    <v-btn block @click="() => {setReservation(item);update();}" height="50px">予約</v-btn>
                  </v-col>
                </v-row>
              </v-container>
            </v-card>
          </v-tab-item>
        </v-tabs>
      </v-card>

      <!--いろいろ!-->
      <v-form v-model="valid">
        <v-container>
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
          </v-row>
          <v-textarea v-model="formation_str" label="Formation"></v-textarea>
          <v-row>
            <v-col cols="12" md="3">
              <v-btn @click="makePerformance()">makePerformance</v-btn>
            </v-col>
          </v-row>
        </v-container>
      </v-form>
    </v-app>
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
      formations: [],
      seats: [],
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
      menu: false,
      right: null,
      performanceList: []
    };
  },
  methods: {
    update() {
      this.getPerformanceList().then(() => {
        for (let performanceName in this.performanceList) {
          this.getPerformance(performanceName);
        }
      });
    },
    getPerformance(performanceName) {
      const xhr = new XMLHttpRequest();
      xhr.open("POST", host + "getPerformance");
      xhr.onload = () => {
        console.log(xhr.response);
        const data = JSON.parse(xhr.response);
        this.formations[performanceName] =
          typeof data.formation === "undefined" || data.formation === ""
            ? []
            : JSON.parse(data.formation);
        this.seats[performanceName] = data.seats;
        this.seats.splice(); //This code is necessary to reflect changes
      };
      xhr.send(
        JSON.stringify({
          collectionName: this.collectionName,
          token: this.token,
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
            collectionName: this.collectionName,
            token: this.token,
            performanceName: this.performanceName,
            formation: this.formation_str,
            date: this.date
          })
        );
      }
    },
    setReservation(performanceName) {
      let flag = false;
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
            performanceName: performanceName,
            seatNumber: this.seatNumber,
            seat: { type: "line", id: this.userId }
          })
        );
      }
    },
    getPerformanceList() {
      return new Promise(resolve => {
        const xhr = new XMLHttpRequest();
        xhr.open("POST", host + "getPerformanceList");
        xhr.onload = () => {
          console.log(xhr.response);
          this.performanceList = JSON.parse(xhr.response);
          resolve();
        };
        xhr.send(
          JSON.stringify({
            collectionName: this.collectionName,
            token: this.token
          })
        );
      });
    }
  }
};
</script>

<style>
</style>