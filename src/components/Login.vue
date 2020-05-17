<template>
  <v-container>
    <v-card class="mx-auto pa-6 mt-8" max-width="400">
      <div class="text-center my-8">
        <p class="display-2 mb-0">サイコの予約</p>
        <p class="subtitle">管理者用ダッシュボード</p>
      </div>
      <span v-if="failed" class="red--text">
        ログインに失敗しました。
        <br />入力内容に誤りがあります。
        <br />
        <br />
      </span>
      <v-form ref="form" v-model="valid" lazy-validation>
        <v-text-field v-model="value.host" :rules="[rules.required]" label="プロジェクト名" required></v-text-field>
        <v-text-field v-model="value.token" :rules="[rules.required]" type="Password" label="Token"></v-text-field>
        <br />
        <v-btn
          block
          @click="
            if($refs.form.validate())$emit('done');
          "
          height="50px"
          color="primary"
          :dark="valid"
          :loading="loading"
          :disabled="!valid"
        >ログイン</v-btn>
      </v-form>
    </v-card>
  </v-container>
</template>

<script>
export default {
  props: {
    value: Object,
    failed: Boolean,
    loading: Boolean
  },
  data() {
    return {
      rules: {
        required: value => !!value || "入力してください"
      },
      valid: true
    };
  }
};
</script>
