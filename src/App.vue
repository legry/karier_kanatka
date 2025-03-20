<template>
  <v-app>
    <v-app-bar app color="primary" dark>
      <v-toolbar-title>{{ app_bar_title }}</v-toolbar-title>
      <v-spacer></v-spacer>
      <v-btn @click="sel_device" icon><v-icon>mdi-bluetooth-connect</v-icon></v-btn>
      <v-btn @click="blue_disconn" icon><v-icon>mdi-bluetooth-off</v-icon></v-btn>
      <v-dialog v-model="sel_devs_dialog" scrollable max-width="300px">
        <v-list>
          <v-subheader>Блютуз устройства</v-subheader>
          <v-list-item-group v-model="selected_device" color="primary">
            <v-list-item v-for="(item, i) in devices" :key="i" @click="on_devs_dialog(item)">
              <v-list-item-content>
                <v-list-item-title v-text="item.name"></v-list-item-title>
              </v-list-item-content>
            </v-list-item>
          </v-list-item-group>
        </v-list>
      </v-dialog>
    </v-app-bar>

    <v-main>
      <v-container>
        <v-row>
          <v-col cols="12">
            <v-text-field v-model="snd_msg" append-icon="mdi-send" clearable label="Введите текст" type="text"
              @click:append="send_text"></v-text-field>
          </v-col>
        </v-row>
        <v-row>
          <v-col cols="12">
            <v-textarea v-model="inc_msg"></v-textarea>
          </v-col>
        </v-row>
        <v-row justify="space-around">
          <v-col cols="auto">
            <v-btn-toggle v-model="gradvalan" mandatory @change="() => { if (strt == '<start>') strt = '' }"
              color="primary" group rounded>
              <v-btn value="<left>" @click="() => { if (gradvalan != '<left>') inc_msg += '<left>\n' }" icon>
                <v-icon x-large>mdi-rotate-left</v-icon>
              </v-btn>
              <v-btn value="<right>" @click="() => { if (gradvalan != '<right>') inc_msg += '<right>\n' }" icon>
                <v-icon x-large>mdi-rotate-right</v-icon>
              </v-btn>
            </v-btn-toggle>
          </v-col>
          <v-col cols="auto">
            <v-btn-toggle v-model="strt" :mandatory="strt == '<start>'" tile color="primary" group>
              <v-btn value="<start>" :disabled="(alstp == '<alstp>')"
                @click="() => { if (strt != '<start>') inc_msg += '<start>\n' }" icon>
                <v-icon x-large>mdi-gesture-tap</v-icon>
              </v-btn>
            </v-btn-toggle>
          </v-col>
        </v-row>
        <v-row justify="space-around">
          <v-col cols="auto">
            <v-btn-toggle v-model="autmain" mandatory tile color="primary" group>
              <v-btn value="<aut>" @click="() => { if (autmain != '<aut>') inc_msg += '<aut>\n' }" icon>
                <v-icon x-large>mdi-spellcheck</v-icon>
              </v-btn>
              <v-btn value="<main>" @click="() => { if (autmain != '<main>') inc_msg += '<main>\n' }" icon>
                <v-icon x-large>mdi-hand-front-left-outline</v-icon>
              </v-btn>
            </v-btn-toggle>
          </v-col>
          <v-col cols="auto">
            <v-btn-toggle v-model="alstp" @change="() => { if (strt == '<start>') strt = '' }" tile color="primary" group>
              <v-btn value="<alstp>"
                @click="() => { inc_msg = (alstp != '<alstp>') ? inc_msg + '<alstp0>\n' : inc_msg + '<alstp1>\n' }" icon>
                <v-icon x-large>mdi-power</v-icon>
              </v-btn>
            </v-btn-toggle>
          </v-col>
        </v-row>
        <v-row justify="center">
          <v-col cols="auto">
            <v-btn-toggle v-model="hod" mandatory tile color="primary" group>
              <v-btn value="<back>" :disabled="(hod == '<front>') || (alstp == '<alstp>')"
                @click="() => { if (hod != '<back>') inc_msg += '<back>\n' }">
                back
              </v-btn>
              <v-btn value="<not>" :disabled="(alstp == '<alstp>')"
                @click="() => { if (hod != '<not>') inc_msg += '<not>\n' }">
                not
              </v-btn>
              <v-btn value="<front>" :disabled="(hod == '<back>') || (alstp == '<alstp>')"
                @click="() => { if (hod != '<front>') inc_msg += '<front>\n' }">
                front
              </v-btn>
            </v-btn-toggle>
          </v-col>
        </v-row>
      </v-container>
      <v-snackbar v-model="fail_snk" color="red accent-2">
        {{ fail_snk_ms1 }} <strong>{{ fail_snk_ms2 }}</strong>
      </v-snackbar>
      <v-snackbar v-model="scs_snk" color="success">
        {{ scs_snk_ms1 }} <strong>{{ scs_snk_ms2 }}</strong>
      </v-snackbar>
    </v-main>
  </v-app>
</template>

<script>
import { list, connect, disconnect, write, subscribeRawData } from 'C:/Users/User/Documents/vuecli_projs/karier_kanatka/src-cordova/plugins/cordova-plugin-bluetooth-serial/www/bluetoothSerial.js';

export default {
  name: 'App',

  data: () => ({
    app_bar_title: '',
    sel_devs_dialog: false,
    selected_device: null,
    devices: null,
    scs_snk: false,
    scs_snk_ms1: '',
    scs_snk_ms2: '',
    fail_snk: false,
    fail_snk_ms1: '',
    fail_snk_ms2: '',
    snd_msg: '',
    inc_msg: '',
    gradvalan: '<left>',
    strt: '',
    autmain: '<main>',
    alstp: '',
    hod: '<not>'
  }),
  methods: {
    scs_snk_shw(ms1, ms2) {
      this.scs_snk_ms1 = ms1;
      this.scs_snk_ms2 = ms2;
      this.scs_snk = true;
    },
    fail_snk_shw(ms1, ms2) {
      this.fail_snk_ms1 = ms1;
      this.fail_snk_ms2 = ms2;
      this.fail_snk = true;
    },
    send_text() {
      write(this.snd_msg + '\n', this.scs_snk_shw("Успешно", "отправлено"), this.fail_snk_shw("Не", "отправлено"))
    },
    on_devs_dialog(dev) {
      this.sel_devs_dialog = false;
      connect(dev.id, () => {
        this.app_bar_title = dev.name;
        this.scs_snk_shw("Подключено к", dev.name);
        const txtDec = new TextDecoder();
        subscribeRawData((data) => {
          this.inc_msg += txtDec.decode(data);
        })
      }, () => {
        this.fail_snk_shw("Не подключено к", dev.name);
      })
    },
    sel_device() {
      list((devices) => {
        this.devices = devices;
        this.sel_devs_dialog = true;
      })
    },
    blue_disconn() {
      disconnect(() => {
        this.app_bar_title = '';
        this.scs_snk_shw("Успешно", "Отключено!");
      }, () => this.fail_snk_shw("Неудалось", "Отключиться!"))
    }
  }
};
</script>
