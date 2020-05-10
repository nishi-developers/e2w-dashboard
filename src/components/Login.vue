<template>
  <v-container>
    <v-card class="mx-auto pa-6 mt-8" max-width="400">
      <div class="text-center my-8">
        <p class="display-2 mb-0">サイコ予約</p>
        <p class="subtitle">管理者用ダッシュボード</p>
      </div>
      <v-form>
        <v-text-field
          v-model="value.collectionName"
          :rules="[rules.required]"
          label="CollectionName"
          required
        ></v-text-field>
        <v-text-field
          v-model="value.token"
          :rules="[rules.required]"
          type="Password"
          label="Token"
        ></v-text-field>
        <br />
        <v-btn block @click="$emit('done')" height="50px">ログイン</v-btn>
      </v-form>
    </v-card>
  </v-container>
</template>

<script>
export default {
  props: {
    value: Object
  },
  data() {
    return {
      rules: {
        required: value => !!value || "Required."
      }
    };
  }
};
</script>
