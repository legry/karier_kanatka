<template>
  <v-app>
    <v-app-bar app color="primary" dark>

      <v-spacer></v-spacer>

      <v-btn @click="clicklist" text>list</v-btn>
    </v-app-bar>

    <v-main>
      <v-textarea v-model="consl" label="Вывод"></v-textarea>
    </v-main>
  </v-app>
</template>

<script>
import { list } from 'C:/Users/User/Documents/vuecli_projs/karier_kanatka/src-cordova/plugins/cordova-plugin-bluetooth-serial/www/bluetoothSerial.js';

export default {
  name: 'App',

  data: () => ({
    consl: ''
  }),
  methods: {
    clicklist() {
      list((devices) => {
        devices.forEach((device) => {
          this.consl += device.id;
        })
      })
    }
  }
};
</script>
