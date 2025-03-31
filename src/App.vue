<template>
  <v-app>
    <v-app-bar app color="cyan">
      <v-toolbar-title class="cyan--text text--darken-4">{{ app_bar_title }}</v-toolbar-title>
      <v-spacer></v-spacer>
      <v-btn @click="on_theme_sel" icon><v-icon>{{ lghtdrk }}</v-icon></v-btn>
      <v-btn @click="on_btn_sel_dev" icon><v-icon>{{ blCnn }}</v-icon></v-btn>
      <!-- <v-btn @click="blue_disconn" icon><v-icon>mdi-bluetooth-off</v-icon></v-btn> -->
      <v-dialog v-model="sel_devs_dialog" scrollable max-width="300px">
        <v-list>
          <v-subheader>Сопряженные</v-subheader>
          <v-list-item-group v-model="selected_device" color="cyan">
            <v-list-item v-for="(item, i) in devices" :key="i" @click="on_devs_dialog(item)">
              <v-list-item-content>
                <v-list-item-title v-text="item.name"></v-list-item-title>
              </v-list-item-content>
            </v-list-item>
            <v-divider></v-divider>
            <v-progress-linear :active="true" :indeterminate="true" absolute bottom color="deep-purple accent-4">
            </v-progress-linear>
            <v-subheader>
              Не сопряженные
            </v-subheader>
            <v-list-item v-for="(item, i) in unpair_devices" :key="i" @click="on_devs_dialog(item)">
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
        <v-row style="height: 300px" align="center" justify="center">
          <v-col cols="auto">
            <v-avatar color="cyan lighten-3" size="200">
              <span class="cyan--text text--darken-3 text-h1">{{ amp }}</span>
            </v-avatar>
            <!-- <v-text-field v-model="snd_msg" append-icon="mdi-send" clearable label="Введите текст" type="text"
              @click:append="send_text">
            </v-text-field> -->
          </v-col>
        </v-row>

        <!-- <v-row>
          <v-col cols="12">
            <v-textarea v-model="inc_msg"></v-textarea>
          </v-col>
        </v-row> -->
        <v-row>
          <v-col cols="12">
            <v-slider v-model="ust" color="cyan darken-1" @end="() => { send_cmd(`<ust${ust}>`); }" thumb-label="always"
              min="0" max="70" :append-icon="plus" :prepend-icon="minus"
              @click:append="() => { if (ust < 70) { ust++; send_cmd(`<ust${ust}>`); } }"
              @click:prepend="() => { if (ust > 0) { ust--; send_cmd(`<ust${ust}>`); } }">
            </v-slider>
          </v-col>
        </v-row>
        <v-row justify="space-around">
          <v-col cols="auto">
            <v-btn-toggle v-model="gradvalan" mandatory @change="() => { if (strt == '<start>') strt = '' }"
              color="cyan darken-1" group rounded>
              <v-btn value="<left>" rounded @click="() => { if (gradvalan != '<left>') send_cmd('<left>') }" icon>
                <v-icon x-large>{{ rotleft }}</v-icon>
              </v-btn>
              <v-btn value="<right>" rounded @click="() => { if (gradvalan != '<right>') send_cmd('<right>') }" icon>
                <v-icon x-large>{{ rotright }}</v-icon>
              </v-btn>
            </v-btn-toggle>
          </v-col>
          <v-col cols="auto">
            <v-btn-toggle v-model="strt" :mandatory="strt == '<start>'" rounded color="success" group>
              <v-btn value="<start>" rounded :disabled="(alstp == '<alstp>')"
                @click="() => { if (strt != '<start>') send_cmd('<start>') }" icon>
                <v-icon x-large>{{ gestap }}</v-icon>
              </v-btn>
            </v-btn-toggle>
          </v-col>
        </v-row>
        <v-row justify="space-around">
          <v-col cols="auto">
            <v-btn-toggle v-model="autmain" mandatory rounded color="cyan darken-1" group>
              <v-btn value="<gidr>" v-show="gidr_shw" rounded
                @click="() => { if (autmain != '<gidr>') send_cmd('<gidr>') }" text>
                Гидр.
              </v-btn>
              <v-btn value="<aut>" rounded @click="() => { if (autmain != '<aut>') send_cmd('<aut>') }" icon>
                <v-icon x-large>{{ splchk }}</v-icon>
              </v-btn>
              <v-btn value="<main>" rounded @click="() => { if (autmain != '<main>') send_cmd('<main>') }" icon>
                <v-icon x-large>{{ hndfrntlftotln }}</v-icon>
              </v-btn>
            </v-btn-toggle>
          </v-col>
          <v-col cols="auto">
            <v-btn-toggle v-model="alstp" @change="() => { if (strt == '<start>') strt = '' }" rounded
              color="red accent-2" group>
              <v-btn value="<alstp>" rounded @click="() => { send_cmd((alstp != '<alstp>') ? '<alstp0>' : '<alstp1>') }"
                icon>
                <v-icon x-large>{{ pwr }}</v-icon>
              </v-btn>
            </v-btn-toggle>
          </v-col>
        </v-row>
        <v-row justify="center">
          <v-col cols="auto">
            <v-btn-toggle v-model="hod" mandatory color="cyan darken-1" group rounded>
              <v-btn value="<back>" rounded :disabled="(hod == '<front>') || (alstp == '<alstp>')"
                @click="() => { if (hod != '<back>') send_cmd('<back>') }" icon>
                <v-icon x-large>{{ arleft }}</v-icon>
              </v-btn>
              <v-btn value="<not>" rounded :disabled="(alstp == '<alstp>')"
                @click="() => { if (hod != '<not>') send_cmd('<not>') }" icon>
                <v-icon x-large>{{ cls }}</v-icon>
              </v-btn>
              <v-btn value="<front>" rounded :disabled="(hod == '<back>') || (alstp == '<alstp>')"
                @click="() => { if (hod != '<front>') send_cmd('<front>') }" icon>
                <v-icon x-large>{{ arright }}</v-icon>
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
import {
  mdiBluetoothConnect, mdiPlus, mdiMinus, mdiRotateLeft, mdiRotateRight,
  mdiGestureTap, mdiSpellcheck, mdiHandFrontLeftOutline, mdiPower, mdiArrowLeft, mdiClose, mdiArrowRight, mdiThemeLightDark
} from '@mdi/js';
import permissions from '../src-cordova/plugins/cordova-plugin-android-permissions/www/permissions.js';
import { enable, discoverUnpaired, list } from '../src-cordova/plugins/cordova-plugin-bluetooth-classic-serial-port/www/bluetoothClassicSerial.js';
import { connect, isConnected, disconnect, write, subscribe } from '../src-cordova/plugins/cordova-plugin-bluetooth-serial/www/bluetoothSerial.js';
export default {
  name: 'App',

  data: () => ({
    blCnn: mdiBluetoothConnect,
    plus: mdiPlus,
    minus: mdiMinus,
    rotleft: mdiRotateLeft,
    rotright: mdiRotateRight,
    gestap: mdiGestureTap,
    splchk: mdiSpellcheck,
    hndfrntlftotln: mdiHandFrontLeftOutline,
    pwr: mdiPower,
    arleft: mdiArrowLeft,
    cls: mdiClose,
    arright: mdiArrowRight,
    lghtdrk: mdiThemeLightDark,
    app_bar_title: '',
    sel_devs_dialog: false,
    selected_device: null,
    devices: null,
    unpair_devices: null,
    scs_snk: false,
    scs_snk_ms1: '',
    scs_snk_ms2: '',
    fail_snk: false,
    fail_snk_ms1: '',
    fail_snk_ms2: '',
    snd_msg: '',
    inc_msg: '',
    gradvalan: '<left>',
    strt: undefined,
    autmain: '<main>',
    alstp: undefined,
    hod: '<not>',
    amp: 0,
    ust: 0,
    ustok: false,
    val: '',
    mydb: null,
    last_device: null,
    cmd: "",
    func: null,
    gidr_shw: false,
    j: 0,
    amper: 0,
    st: null
  }),
  mounted() {
    document.addEventListener('deviceready', this.onDeviceReady, false);
  },
  methods: {
    onDeviceReady() {
      window.screen.orientation.lock("portrait");
      // cordova.plugins.diagnostic.switchToLocationSettings();
      // eslint-disable-next-line
      cordova.plugins.diagnostic.registerLocationStateChangeHandler((state) => {
        // this.inc_msg += state + '\n';
        if ((state == "high_accuracy") || (state == "device_only"))
          if (this.func != null) this.func();
      });
      // this.inc_msg += "device ready\n";
      window.sqlitePlugin.openDatabase({ name: 'mydb.db', location: 'default' }, (db) => {
        this.mydb = db;
        // this.inc_msg += "mydb is opened!\n";
        this.mydb.executeSql(["CREATE TABLE IF NOT EXISTS last_device (name TEXT, id TEXT)"], [],
          () => {
            // this.inc_msg += "table last_device is created!\n";
            this.mydb.executeSql("SELECT * FROM last_device", [], (res) => {
              // this.inc_msg += "last_device is select, res.rows.length = " + JSON.stringify(res.rows.item(0)) + '\n';
              if (res.rows.length == 0) {
                this.mydb.executeSql("INSERT INTO last_device VALUES (?1,?2)", ["some_name", "some_id"], () => {
                  // this.inc_msg += "insert is success";
                  this.sel_device();
                }, (/* error */) => {
                  // this.inc_msg += "insert is error: " + error + '\n';
                })
              } else {
                this.last_device = res.rows.item(0);
                // this.inc_msg += "name: " + this.last_device.name;
                // this.inc_msg += "; id: " + this.last_device.id + '\n';
                this.sel_device();
              }
            }, (/* error */) => {
              // this.inc_msg += "select is error: " + error + '\n';
            })
          }, (/* error */) => {
            // this.inc_msg += "table is error: " + error + '\n';
          });
      }, (/* err */) => {
        // this.inc_msg += ('Open database ERROR: ' + JSON.stringify(err) + '\n');
      });
    },
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
    send_cmd(cmd) {
      isConnected(() => {
        // this.inc_msg += "is connected!\n"; 
        this.cmd = cmd;
        if (this.app_bar_title != this.last_device.name) {
          this.working(this.last_device.name);
        } else {
          write(this.cmd, () => { }, () => { });
        }
      }, () => {
        // this.inc_msg += "is not connected!\n";
        connect(this.last_device.id, () => { this.working(this.last_device.name) }, () => {
          this.fail_snk_shw("Не подключено к", this.last_device.name);
        })
      })
    },
    send_text() {
      // write(this.snd_msg, this.scs_snk_shw("Успешно", "отправлено"), this.fail_snk_shw("Не", "отправлено"))
      // const data = '[false,true,false,true,true,false]'
      this.recData(this.snd_msg);
    },
    recData(data) {
      var inc_msg = (data.match(/\[((true|false),){2}(1|2|3|true|false),((true|false),){2}(true|false)\]/gi));
      // this.inc_msg += inc_msg + '\n';
      if (inc_msg != null) {
        // this.inc_msg += data;
        this.st = JSON.parse(inc_msg);
        this.gradvalan = (!this.st[0]) ? "<left>" : "<right>";
        this.strt = (this.st[1]) ? "<start>" : undefined;
        this.gidr_shw = (typeof this.st[2] === "number");
        // this.inc_msg += this.gidr_shw + '\n';
        if ((this.st[2] == true) || (this.st[2] == 1)) this.autmain = "<main>";
        else if ((this.st[2] == false) || (this.st[2] == 2)) this.autmain = "<aut>";
        else if ((this.st[2] == 3)) this.autmain = "<gidr>";
        this.alstp = (this.st[3]) ? undefined : "<alstp>";
        this.hod = (this.st[4] == this.st[5]) ? "<not>" : (this.st[4]) ? "<back>" : "<front>";
        inc_msg = null;
        write("<?ust>", () => { }, () => { });
      } else {
        if (!this.ustok) {
          inc_msg = data.match(/<\d+>/gi);
          if (inc_msg != null) {

            this.ustok = true;
            this.ust = (/\d+/gi).exec(inc_msg)[0];
          }
          write("<amp>", () => { }, () => { });
        } else {
          inc_msg = data.match(/<\d+>/gi);
          if (inc_msg != null) {
            if (this.j < 10) {
              this.j += 1;
              this.val = (/\d+/gi).exec(inc_msg)[0];
              this.amper += Number.parseInt(this.val);
            }
            if (this.j == 10) {
              this.amp = this.amper / 10;
              this.j = 0;
              this.amper = 0;
            }
          }
        }
        write("<amp>", () => { }, () => { });
      }
    },
    working(name) {
      this.app_bar_title = name;
      this.scs_snk_shw("Подключено к", name);
      write("<inic>", () => { }, () => { });
      subscribe('\n', this.recData)
    },
    conn(name, id) {
      isConnected(() => {
        // this.inc_msg += "is connected!\n";
        // this.mydb.executeSql("UPDATE last_device SET name = (?1), id = (?2)", [name, id], () => { }, () => { });
        // this.last_device.name = name;
        // this.last_device.id = id;
        this.working(name);
      }, () => {
        // this.inc_msg += "is not connected!\n";
        connect(id, () => {
          this.mydb.executeSql("UPDATE last_device SET name = (?1), id = (?2)", [name, id], () => { }, () => { });
          this.last_device = { name: name, id: id };
          this.working(name)
        }, () => {
          this.fail_snk_shw("Не подключено к", name);
        })
      })
    },
    on_devs_dialog(dev) {
      isConnected(() => {
        disconnect(() => {
          // this.inc_msg += "disconnect";
          this.app_bar_title = '';
          // this.scs_snk_shw("Успешно", "Отключено!");
          this.sel_devs_dialog = false;
          this.conn(dev.name, dev.id);
        }, () => {/* this.fail_snk_shw("Неудалось", "Отключиться!") */ })
      }, () => {
        // this.inc_msg += "disconnect";
        this.app_bar_title = '';
        // this.scs_snk_shw("Успешно", "Отключено!");
        this.sel_devs_dialog = false;
        this.conn(dev.name, dev.id);
      })
    },
    wrap_per_loc() {
      permissions.requestPermission(permissions.ACCESS_FINE_LOCATION, (status) => {
        if (status.hasPermission) {
          // eslint-disable-next-line
          cordova.plugins.diagnostic.getLocationMode((locationMode) => {
            switch (locationMode) {
              // eslint-disable-next-line
              case cordova.plugins.diagnostic.locationMode.HIGH_ACCURACY: {
                // this.inc_msg += ("High accuracy\n");
                this.func();
                break;
              }
              // eslint-disable-next-line
              case cordova.plugins.diagnostic.locationMode.BATTERY_SAVING:
                // this.inc_msg += ("Battery saving\n");
                break;
              // eslint-disable-next-line
              case cordova.plugins.diagnostic.locationMode.DEVICE_ONLY: {
                // this.inc_msg += ("Device only\n");
                this.func();
                break;
              }
              // eslint-disable-next-line
              case cordova.plugins.diagnostic.locationMode.LOCATION_OFF: {
                // this.inc_msg += ("Location off\n");
                // eslint-disable-next-line
                cordova.plugins.diagnostic.switchToLocationSettings();
                break;
              }
            }
          }, (/* error */) => {
            // this.inc_msg += ("The following error occurred: " + error) + '\n';
          });
        }
      }, () => { });
    },
    onlist() {
      this.func = () => {
        discoverUnpaired((unpair_devices) => {
          let unp_st = new Set(unpair_devices);
          this.unpair_devices = [...unp_st];
        }, () => { });
        list((devices) => {
          this.sel_devs_dialog = true;
          this.devices = devices;
        })
      }
      this.wrap_per_loc();
    },
    on_btn_sel_dev() {
      this.func = () => {
        list(() => {
          enable(() => {
            this.onlist();
          }, () => {
            this.onlist();
          });
        })
      }
      this.wrap_per_loc();
    },
    sel_device() {
      this.func = () => {
        list(() => {
          enable(() => {
            if ((this.last_device != null) && ((this.last_device.name != "some_name") && (this.last_device.id != "some_id")))
              this.conn(this.last_device.name, this.last_device.id);
            else this.onlist();
          }, () => {
            // this.inc_msg += "blu disable"
            if ((this.last_device != null) && ((this.last_device.name != "some_name") && (this.last_device.id != "some_id")))
              this.conn(this.last_device.name, this.last_device.id);
            else this.onlist();
          });
        })
      }
      this.wrap_per_loc();
    },
    blue_disconn() {
      disconnect(() => {
        this.app_bar_title = '';
        this.scs_snk_shw("Успешно", "Отключено!");
      }, () => this.fail_snk_shw("Неудалось", "Отключиться!"))
    },
    on_theme_sel() {
      this.$vuetify.theme.dark = !this.$vuetify.theme.dark;
    }
  }
};
</script>
