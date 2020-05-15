<template>
  <v-card class="mx-auto" max-width="900">
    <v-toolbar flat color="primary" dark>
      <v-toolbar-title>団体名</v-toolbar-title>
    </v-toolbar>
    <v-tabs vertical>
      <v-tab v-for="item in Object.keys(info.performanceList)" :key="item" link>
        <!--<v-icon left>mdi-account</v-icon>!-->
        {{item}}
      </v-tab>
      <v-tab v-if="Object.keys(info.performanceList).length > 0">
        <i>
          <b>＋ 追加...</b>
        </i>
      </v-tab>
      <v-tab-item v-for="item in Object.keys(info.performanceList)" :key="item">
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
            :formation="info.formations[item]"
            :seats="info.seats[item]"
          />
        </v-card>
      </v-tab-item>
      <v-tab-item v-if="Object.keys(info.performanceList).length > 0">
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
    </v-tabs>
  </v-card>
</template>

<script>
import SeatTable from "./SeatTable.vue";
export default {
  components: {
    SeatTable
  },
  props: {
    info: Object
  },
  data() {
    return {
      rules: {
        required: value => !!value || "Required."
      },
      seatNumbers: [],
      userIds: [],
      menu: false,
      editing: {
        performanceName: "",
        formation: [[]],
        width: 0,
        height: 0,
        date: null
      }
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
    }
  }
};
</script>