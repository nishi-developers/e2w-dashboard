<template>
  <v-dialog v-model="show" persistent max-width="700">
    <v-card>
      <v-card-title>サイコの予約 ダッシュボード</v-card-title>
      <v-card-text>
        <v-select
          :items="servers"
          label="ログイン先" />
        <v-radio-group v-model="loginType">
          <v-radio label="v2でログイン(推奨)" value="v2" />
          <v-radio label="v1でログイン" value="v1" />
        </v-radio-group>
        <v-btn block color="accent" v-if="loginType === 'v2'">
          <v-icon>mdi-google</v-icon>
          &nbsp;Google でログイン
        </v-btn>
        <v-text-field label="トークン" v-if="loginType === 'v1'" />
      </v-card-text>
    </v-card>
  </v-dialog>
</template>

<script>
export default {
  data() {
    return {
      show: true,
      servers: ['2019 2C', 'psycho-reservation-test'],
      loginType: "v2"
    }
  }
}
</script>
