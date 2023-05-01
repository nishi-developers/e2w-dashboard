<template>
  <v-card class="mx-auto" max-width="700">
    <v-toolbar flat color="primary" dark>
      <v-toolbar-title>団体名</v-toolbar-title>
    </v-toolbar>

    <v-window v-model="step" touchless>
      <v-window-item :value="1">
        <v-list>
          <v-list-item-group v-model="selectedPerformanceNum" :mandatory="mandatory">
            <v-list-item
              v-for="(item, i) in Object.keys(performanceList)"
              :key="i"
              @click="mandatory = true; step = 2;"
            >
              <v-list-item-content>
                <v-list-item-title v-text="item"></v-list-item-title>
                <v-list-item-subtitle v-text="performanceList[item].date"></v-list-item-subtitle>
              </v-list-item-content>
            </v-list-item>
          </v-list-item-group>

          <v-list-item>
            <v-dialog v-model="addingDialog" width="500">
              <template v-slot:activator="{ on }">
                <v-btn v-on="on" outlined block color="secondary" @click="initEditForm()">
                  <v-icon>mdi-plus</v-icon>公演を追加
                </v-btn>
              </template>

              <v-card>
                <v-card-title class="pl-7 headline grey lighten-2" primary-title>公演を追加</v-card-title>
                <v-container>
                  <v-form ref="addPerformanceForm" v-model="addPerformanceValid" lazy-validation>
                    <v-row class="mx-1">
                      <v-col cols="12" md="3" class="py-0">
                        <v-text-field
                          v-model="editing.performanceName"
                          label="公演名"
                          :rules="[rules.required]"
                          required
                        ></v-text-field>
                      </v-col>
                      <v-col cols="12" md="3" class="py-0">
                        <v-menu
                          ref="menu"
                          v-model="menu"
                          :close-on-content-click="false"
                          :return-value.sync="editing.date"
                          transition="scale-transition"
                          offset-y
                          min-width="290px"
                        >
                          <template v-slot:activator="{ on }">
                            <v-text-field
                              v-model="editing.date"
                              label="日付"
                              readonly
                              v-on="on"
                              :rules="[rules.required]"
                              required
                            ></v-text-field>
                          </template>
                          <v-date-picker v-model="editing.date" no-title scrollable>
                            <div class="flex-grow-1"></div>
                            <v-btn text color="primary" @click="menu=false">キャンセル</v-btn>
                            <v-btn text color="primary" @click="$refs.menu.save(editing.date)">OK</v-btn>
                          </v-date-picker>
                        </v-menu>
                      </v-col>
                      <v-col cols="12" sm="6" md="6" class="py-0">
                        <v-row>
                          <v-col cols="12" sm="6" md="6" class="py-0">
                            <v-text-field
                              v-model.number="editing.width"
                              label="横"
                              type="number"
                              min="1"
                              :rules="[rules.required]"
                              required
                            ></v-text-field>
                          </v-col>
                          <v-col cols="12" sm="6" md="6" class="py-0">
                            <v-text-field
                              v-model.number="editing.height"
                              label="縦"
                              type="number"
                              min="1"
                              :rules="[rules.required]"
                              required
                            ></v-text-field>
                          </v-col>
                        </v-row>
                      </v-col>
                    </v-row>
                  </v-form>

                  <seat-table
                    class="mx-auto my-4"
                    @cellclicked="toggle"
                    :formation="editing.formation"
                    :seats="new Object()"
                  />

                  <v-card-actions class="d-flex justify-end pr-4">
                    <v-btn text @click="addingDialog = false" :disabled="loading">キャンセル</v-btn>
                    <v-btn
                      @click="
                      if($refs.addPerformanceForm.validate()){$emit('addPerformance', editing.performanceName, editing.date, editing.formation);}
                      "
                      color="primary"
                      :loading="loading"
                    >完了</v-btn>
                  </v-card-actions>
                </v-container>
              </v-card>
            </v-dialog>
          </v-list-item>
        </v-list>
      </v-window-item>

      <v-window-item :value="2">
        <v-card flat class="pa-3" v-if="selectedPerformanceNum >= 0">
          <v-btn text class="px-0" color="secondary" @click="step = 1" small>
            <v-icon small>mdi-less-than</v-icon>公演一覧
          </v-btn>
          <v-card-title>{{ selectedPerformance }}</v-card-title>
          <v-card-subtitle>{{ performanceList[selectedPerformance].date }}</v-card-subtitle>
          <seat-table
            class="mx-auto mb-10 mt-5"
            :formation="JSON.parse(performanceList[selectedPerformance].formation)"
            :seats="performanceList[selectedPerformance].seats"
            @cellclicked="(i, j) => {userId = ''; reservationValid = true; selectedSeat =  i * JSON.parse(performanceList[selectedPerformance].formation)[0].length + j + 1; reservationDialog = true;}"
          />
          <v-dialog v-model="reservationDialog" max-width="500">
            <v-card>
              <v-card-title class="headline grey lighten-2" primary-title>座席を予約</v-card-title>
              <v-card-text class="title black--text pt-8">
                座席番号&nbsp;
                <span class="display-1">{{selectedSeat}}</span>
              </v-card-text>
              <v-form class="px-6" ref="reservationForm" v-model="reservationValid" lazy-validation>
                <v-text-field v-model="userId" label="ユーザーID" :rules="[rules.required]" required></v-text-field>
              </v-form>
              <v-card-actions class="d-flex justify-end">
                <v-btn text @click="reservationDialog = false" :disabled="loading">キャンセル</v-btn>
                <v-btn
                  @click="
                  if($refs.reservationForm.validate()){$emit('setReservation', selectedPerformance, userId, selectedSeat);}
                  "
                  color="primary"
                  :loading="loading"
                >完了</v-btn>
              </v-card-actions>
            </v-card>
          </v-dialog>
        </v-card>
      </v-window-item>
    </v-window>

    <v-dialog v-model="failed" max-width="500">
      <v-card>
        <v-card-title class="headline grey lighten-2" primary-title>エラー</v-card-title>
        <v-card-text class="pt-8">{{ failedMessage }}</v-card-text>
        <v-card-actions class="d-flex justify-end">
          <v-btn @click="$emit('failedDone')">閉じる</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
    <!--<v-tabs vertical>
      <v-tab v-for="item in Object.keys(performanceList)" :key="item" link>
        {{item}}
      </v-tab>
      <v-tab v-if="Object.keys(performanceList).length > 0">
        <i>
          <b>＋ 追加...</b>
        </i>
      </v-tab>
      <v-tab-item v-for="item in Object.keys(performanceList)" :key="item">
        <v-card flat class="pa-3">
          <v-row>
            <v-col cols="12" sm="5" md="5" class="py-0">
              <v-text-field
                v-model.number="seatNumbers[item]"
                label="座席番号"
                type="number"
                min="0"
                hide-details="auto"
              ></v-text-field>
            </v-col>
            <v-col cols="12" sm="5" md="5" class="py-0" hide-details="auto">
              <v-text-field v-model="userIds[item]" label="ユーザーID"></v-text-field>
            </v-col>
            <v-col cols="12" sm="2" md="2" class="align-end" hide-details="auto">
              <v-btn
                block
                @click="$emit('setReservation', item, seatNumbers[item], userIds[item])"
              >予約</v-btn>
            </v-col>
          </v-row>
          <seat-table
            class="mx-auto mt-10"
            :style="{cursor:'default'}"
            :formation="formations[item]"
            :seats="seats[item]"
          />
        </v-card>
      </v-tab-item>
      <v-tab-item v-if="Object.keys(performanceList).length > 0">
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
                v-model="editing.performanceName"
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
                :return-value.sync="editing.date"
                transition="scale-transition"
                offset-y
                min-width="290px"
              >
                <template v-slot:activator="{ on }">
                  <v-text-field v-model="editing.date" label="日付" readonly v-on="on"></v-text-field>
                </template>
                <v-date-picker v-model="editing.date" no-title scrollable>
                  <div class="flex-grow-1"></div>
                  <v-btn text color="primary" @click="menu=false">キャンセル</v-btn>
                  <v-btn text color="primary" @click="$refs.menu.save(editing.date)">OK</v-btn>
                </v-date-picker>
              </v-menu>
            </v-col>
            <v-col cols="12" sm="3" md="3">
              <v-text-field v-model.number="editing.width" label="横" type="number" min="1"></v-text-field>
            </v-col>
            <v-col cols="12" sm="3" md="3">
              <v-text-field v-model.number="editing.height" label="縦" type="number" min="1"></v-text-field>
            </v-col>
          </v-row>
          <v-row>
            <v-col cols="12" md="3">
              <v-btn
                @click="() => {$emit('addPerformance', editing.performanceName, editing.date, editing.formation);initEditForm();}"
              >公演を追加</v-btn>
            </v-col>
          </v-row>
        </v-container>
      </v-tab-item>
    </v-tabs>-->
  </v-card>
</template>

<script>
import SeatTable from "./SeatTable.vue";
export default {
  components: {
    SeatTable
  },
  props: {
    performanceList: Object,
    loading: Boolean,
    failed: Boolean,
    failedMessage: String
  },
  data() {
    return {
      rules: {
        required: value => !!value || "入力してください"
      },
      menu: false,
      editing: {
        performanceName: "",
        formation: [[]],
        width: 0,
        height: 0,
        date: null
      },
      step: 1,
      selectedPerformanceNum: -1,
      addingDialog: false,
      reservationDialog: false,
      selectedSeat: -1,
      userId: "",
      addPerformanceValid: true,
      reservationValid: true,
      mandatory: false
    };
  },
  methods: {
    initEditForm() {
      this.editing.formation = [
        [1, 1, 1, 1, 1],
        [1, 1, 1, 1, 1],
        [1, 1, 1, 1, 1]
      ];
      this.editing.width = 5;
      this.editing.height = 3;
      this.editing.performanceName = "";
      this.editing.date = new Date().toISOString().substr(0, 10);
    },
    toggle(i, j) {
      this.editing.formation[i][j] = (this.editing.formation[i][j] + 1) % 3;
      this.editing.formation.splice();
    }
  },
  mounted: function() {
    this.initEditForm();
  },
  watch: {
    "editing.width": function(newVal, oldVal) {
      if (newVal > oldVal) {
        for (let i = 0; i < this.editing.formation.length; i++) {
          this.editing.formation[i] = this.editing.formation[i].concat(
            new Array(newVal - this.editing.formation[i].length).fill(1)
          );
        }
        this.editing.formation.splice(); //This code is necessary to reflect changes
      } else {
        for (let i = 0; i < this.editing.formation.length; i++) {
          this.editing.formation[i] = this.editing.formation[i].slice(
            0,
            newVal
          );
        }
        this.editing.formation.splice(); //This code is necessary to reflect changes
      }
    },
    "editing.height": function(newVal, oldVal) {
      if (newVal > oldVal) {
        while (this.editing.formation.length < newVal) {
          this.editing.formation.push(
            new Array(this.editing.formation[0].length).fill(1)
          );
        }
      } else {
        this.editing.formation = this.editing.formation.slice(0, newVal);
      }
    },
    loading: function() {
      this.addingDialog = this.reservationDialog = false;
    }
  },
  computed: {
    //アロー関数はthisが異なるため使えない
    selectedPerformance: function() {
      return Object.keys(this.performanceList)[this.selectedPerformanceNum];
    }
  }
};
</script>