<template>
  <v-app>
    <v-app-bar app color="primary" dark>

      <v-spacer></v-spacer>

      <v-btn @click="clicklist" text>list</v-btn>
    </v-app-bar>

    <v-main>
      <v-list>
        <v-subheader>Блютуз устройства</v-subheader>
        <v-list-item-group v-model="selected_device" color="primary">
          <v-list-item v-for="(item, i) in devices" :key="i">
            <v-list-item-content>
              <v-list-item-title v-text="item.name"></v-list-item-title>
            </v-list-item-content>
          </v-list-item>
        </v-list-item-group>
      </v-list>
    </v-main>
  </v-app>
</template>

<script>
import { list } from 'C:/vue_cli_projs/karier_kanatka/src-cordova/plugins/cordova-plugin-bluetooth-serial/www/bluetoothSerial.js';

export default {
  name: 'App',

  data: () => ({
    selected_device: null,
    devices: null
  }),
  methods: {
    clicklist() {
      list((devices) => {
        this.devices = devices;
      })
    }
  }
};
</script>
