<template>
  <v-card width="90%" class="mx-auto">
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
        <v-card flat>
          <seat-table
            class="mx-auto mt-10"
            :style="{cursor:'default'}"
            :formation="info.formations[item]"
            :seats="info.seats[item]"
          />
          <v-container width="90%" class="mx-auto">
            <v-row class="mt-5">
              <v-col cols="12" sm="5" md="5">
                <v-text-field v-model.number="seatNumbers[item]" label="座席番号" type="number" min="0"></v-text-field>
              </v-col>
              <v-col cols="12" sm="5" md="5">
                <v-text-field v-model="userIds[item]" label="ユーザーID"></v-text-field>
              </v-col>
              <v-col cols="12" sm="2" md="2">
                <v-btn
                  block
                  @click="$emit('setReservation', item, seatNumbers[item], userIds[item])"
                  height="50px"
                >予約</v-btn>
              </v-col>
            </v-row>
          </v-container>
        </v-card>
      </v-tab-item>
      <v-tab-item v-if="Object.keys(info.performanceList).length > 0">
        <v-container>
          <seat-table
            class="mx-auto mt-10"
            @cellclicked="toggle"
            :formation="info.editing.formation"
            :seats="new Object()"
            :style="{cursor:'pointer'}"
          />
          <v-row>
            <v-col cols="12" md="3">
              <v-text-field
                v-model="info.performanceName"
                :rules="[rules.required]"
                label="公演名"
                required
              ></v-text-field>
            </v-col>
            <v-col cols="12" md="3">
              <v-menu
                ref="info.menu"
                v-model="info.menu"
                :close-on-content-click="false"
                :return-value.sync="info.date"
                transition="scale-transition"
                offset-y
                min-width="290px"
              >
                <template v-slot:activator="{ on }">
                  <v-text-field v-model="info.date" label="日付" readonly v-on="on"></v-text-field>
                </template>
                <v-date-picker v-model="info.date" no-title scrollable>
                  <div class="flex-grow-1"></div>
                  <v-btn text color="primary" @click="menu=false">キャンセル</v-btn>
                  <v-btn text color="primary" @click="$refs.menu.save(info.date)">OK</v-btn>
                </v-date-picker>
              </v-menu>
            </v-col>
            <v-col cols="12" sm="3" md="3">
              <v-text-field
                @change="updateWidth()"
                v-model.number="info.editing.width"
                label="横"
                type="number"
                min="1"
              ></v-text-field>
            </v-col>
            <v-col cols="12" sm="3" md="3">
              <v-text-field
                @change="updateHeight()"
                v-model.number="info.editing.height"
                label="縦"
                type="number"
                min="1"
              ></v-text-field>
            </v-col>
          </v-row>
          <v-row>
            <v-col cols="12" md="3">
              <v-btn @click="$emit('addPerformance')">公演を追加</v-btn>
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
      userIds: []
    };
  },
  methods: {
    toggle(i, j) {
      this.info.editing.formation[i][j] =
        (this.info.editing.formation[i][j] + 1) % 3;
      this.info.editing.formation.splice();
    },
    updateWidth() {
      if (this.info.editing.width > this.info.editing.formation[0].length) {
        for (let i = 0; i < this.info.editing.formation.length; i++) {
          this.info.editing.formation[i] = this.info.editing.formation[
            i
          ].concat(
            new Array(
              this.info.editing.width - this.info.editing.formation[i].length
            ).fill(1)
          );
        }
        this.info.editing.formation.splice(); //This code is necessary to reflect changes
      } else {
        for (let i = 0; i < this.info.editing.formation.length; i++) {
          this.info.editing.formation[i] = this.info.editing.formation[i].slice(
            0,
            this.info.editing.width
          );
        }
        this.info.editing.formation.splice(); //This code is necessary to reflect changes
      }
    },
    updateHeight() {
      if (this.info.editing.height > this.info.editing.formation.length) {
        while (
          this.info.editing.height !== this.info.editing.formation.length
        ) {
          this.info.editing.formation.push(
            new Array(this.info.editing.formation[0].length).fill(1)
          );
        }
      } else {
        this.info.editing.formation = this.info.editing.formation.slice(
          0,
          this.info.editing.height
        );
      }
    }
  }
};
</script>