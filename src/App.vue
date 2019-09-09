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
          <v-tab v-for="item in Object.keys(this.performanceList)" :key="item" link>
            <!--<v-icon left>mdi-account</v-icon>!-->
            {{item}}
          </v-tab>
          <v-tab v-if="Object.keys(this.performanceList).length > 0">
            <i>
              <b>＋ 追加...</b>
            </i>
          </v-tab>
          <v-tab-item v-for="item in Object.keys(this.performanceList)" :key="item">
            <v-card flat>
              <seat-table
                class="mx-auto mt-10"
                :style="{cursor:'default'}"
                :formation="formations[item]"
                :seats="seats[item]"
              />
              <v-container width="90%" class="mx-auto">
                <v-row class="mt-5">
                  <v-col cols="12" sm="5" md="5">
                    <v-text-field v-model.number="seatNumber" label="座席番号" type="number" min="0"></v-text-field>
                  </v-col>
                  <v-col cols="12" sm="5" md="5">
                    <v-text-field v-model="userId" label="ユーザーID"></v-text-field>
                  </v-col>
                  <v-col cols="12" sm="2" md="2">
                    <v-btn block @click="() => {setReservation(item);update();}" height="50px">予約</v-btn>
                  </v-col>
                </v-row>
              </v-container>
            </v-card>
          </v-tab-item>
          <v-tab-item v-if="Object.keys(this.performanceList).length > 0">
            <v-container>
              <seat-table
                class="mx-auto mt-10"
                @cellclicked="toggle"
                :formation="editing.formation"
                :seats="new Object()"
                :style="{cursor:'pointer'}"
              />
              <v-row>
                <v-col cols="12" md="3">
                  <v-text-field
                    v-model="performanceName"
                    :rules="[rules.required]"
                    label="公演名"
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
                      <v-text-field v-model="date" label="日付" readonly v-on="on"></v-text-field>
                    </template>
                    <v-date-picker v-model="date" no-title scrollable>
                      <div class="flex-grow-1"></div>
                      <v-btn text color="primary" @click="menu=false">キャンセル</v-btn>
                      <v-btn text color="primary" @click="$refs.menu.save(date)">OK</v-btn>
                    </v-date-picker>
                  </v-menu>
                </v-col>
                <v-col cols="12" sm="3" md="3">
                  <v-text-field
                    @change="updateWidth()"
                    v-model.number="editing.width"
                    label="横"
                    type="number"
                    min="1"
                  ></v-text-field>
                </v-col>
                <v-col cols="12" sm="3" md="3">
                  <v-text-field
                    @change="updateHeight()"
                    v-model.number="editing.height"
                    label="縦"
                    type="number"
                    min="1"
                  ></v-text-field>
                </v-col>
              </v-row>
              <v-row>
                <v-col cols="12" md="3">
                  <v-btn @click="() => {makePerformance();initEditForm();update();}">公演を追加</v-btn>
                </v-col>
              </v-row>
            </v-container>
          </v-tab-item>
        </v-tabs>
      </v-card>
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
      editing: {
        formation: [[]],
        width: 0,
        height: 0
      },
      date: Object,
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
    initEditForm() {
      this.editing.formation = [
        [1, 1, 1, 1, 1],
        [1, 1, 1, 1, 1],
        [1, 1, 1, 1, 1]
      ];
      this.editing.width = 5;
      this.editing.height = 3;
      this.performanceName = "";
      this.date = new Date().toISOString().substr(0, 10);
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
            formation: JSON.stringify(this.editing.formation),
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
    },
    updateWidth() {
      if (this.editing.width > this.editing.formation[0].length) {
        for (let i = 0; i < this.editing.formation.length; i++) {
          this.editing.formation[i] = this.editing.formation[i].concat(
            new Array(
              this.editing.width - this.editing.formation[i].length
            ).fill(1)
          );
        }
        this.editing.formation.splice(); //This code is necessary to reflect changes
      } else {
        for (let i = 0; i < this.editing.formation.length; i++) {
          this.editing.formation[i] = this.editing.formation[i].slice(
            0,
            this.editing.width
          );
        }
        this.editing.formation.splice(); //This code is necessary to reflect changes
      }
    },
    updateHeight() {
      if (this.editing.height > this.editing.formation.length) {
        while (this.editing.height !== this.editing.formation.length) {
          this.editing.formation.push(
            new Array(this.editing.formation[0].length).fill(1)
          );
        }
      } else {
        this.editing.formation = this.editing.formation.slice(
          0,
          this.editing.height
        );
      }
    },
    toggle(i, j) {
      this.editing.formation[i][j] = (this.editing.formation[i][j] + 1) % 3;
      this.editing.formation.splice();
    }
  },
  mounted: function() {
    this.initEditForm();
  }
};
</script>

<style>
</style>