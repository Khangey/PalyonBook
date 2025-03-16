<template>
    <div>
        <v-card class="my-2 elevation-4" v-for="card in cards" :key="card.title" >
            <v-card-title @click="card.show = !card.show">
                <v-btn @click.once="card.show = !card.show" icon>
                    <v-icon>{{ card.show ? 'keyboard_arrow_down' : 'keyboard_arrow_up' }}</v-icon>
                </v-btn>
                {{card.title}}
            </v-card-title>
          <v-expand-transition>
          
            <!-- v-card-text的padding导致动画收起时卡顿，置为 0，内部用一个div来控制-->

            <v-card-text v-show="card.show" style="padding: 0">
              <div style="padding: 0 16px 16px">
                <p v-if="card.subtitle" class="">{{card.subtitle}}</p>
                <template v-if="card.tips">
                  <p v-for="t in card.tips" :key="t.text">{{t.text}} <a v-if="t.link" target="_blank" :href="t.link">དྲ་ཐག</a></p>
                </template>

                <template v-for="f in card.fields" :key="f.key" >
                  <v-checkbox small hide-details v-if="f.type === 'checkbox' " :prepend-icon="f.icon" v-model="settings[f.key]" :label="f.label" ></v-checkbox>
                  <v-textarea outlined v-else-if="f.type === 'textarea' " :prepend-icon="f.icon" v-model="settings[f.key]" :label="f.label" ></v-textarea>
                  <v-select small hide-details v-else-if="f.type === 'select' " :prepend-icon="f.icon" v-model="settings[f.key]" :items="f.items" :label="f.label" > </v-select>
                  <v-text-field v-else :prepend-icon="f.icon" v-model="settings[f.key]" :label="f.label" type="text"></v-text-field>
                </template>
                <template v-for="b in card.buttons" :key="b.label" >
                  <v-btn @click="run(b.action)" color="primary"><v-icon>{{b.icon}}</v-icon>{{b.label}}</v-btn>
                </template>

                <template v-for="g in card.groups" :key="g.label" >
                  <v-checkbox small hide-details v-model="settings[g.key]" :label="g.label"></v-checkbox>
                  <template v-if="settings[g.key]">
                    <template v-for="f in g.fields" :key="f.key" >
                      <v-textarea outlined v-if="f.type === 'textarea' " :prepend-icon="f.icon" v-model="settings[f.key]" :label="f.label" ></v-textarea>
                      <v-text-field v-else :prepend-icon="f.icon" v-model="settings[f.key]" :label="f.label" type="text"></v-text-field>
                    </template>
                  </template>
                </template>

                <template v-if="card.show_friends">
                  <v-row v-for="(friend, idx) in settings.FRIENDS" :key="'friend-'+friend.href">
                    <v-col class='py-0' cols=3>
                      <v-text-field flat small hide-details single-line v-model="friend.text" label="མིང་།" type="text"></v-text-field>
                    </v-col>
                    <v-col class='pa-0' cols=9>
                      <v-text-field flat small hide-details single-line v-model="friend.href" label="དྲ་ཐག" type="text"
                                    append-outer-icon="delete" @click:append-outer="settings.FRIENDS.splice(idx, 1)" ></v-text-field>
                    </v-col>
                  </v-row>
                  <v-row>
                    <v-col align="center">
                      <v-btn color="primary" @click="settings.FRIENDS.push({text:'', href: ''})"><v-icon>add</v-icon>ཁ་སྣོན།</v-btn>
                    </v-col>
                  </v-row>
                </template>

                <template v-if="card.show_socials">
                  <p>ནུས་སློང་བྱས་ཟིན་པའི་སྤྱི་ཚོགས་འབྲེལ་ལམ་ཞབས་ཞུ་ནས་དྲ་ཚིགས་འདི་ལ་ཐོ་འཇུག་བྱ་ཆོག</p>
                  <v-combobox v-model="settings.SOCIALS" :items="sns_items" label="སྤྱི་ཚོགས་འབྲེལ་ལམ་ཞབས་ཞུ་གདམ་གསེས་བྱོས།" hide-selected multiple small-chips>
                    <template v-slot:selection="{ attrs, item, parent, selected }">
                      <v-chip v-bind="attrs" color="green lighten-3" :input-value="selected" label small >
                        <span class="pr-2"> {{ item.text }} </span>
                        <v-icon small @click="parent.selectItem(item)" >close</v-icon>
                      </v-chip>
                    </template>
                  </v-combobox>
                  <v-row v-for="s in settings.SOCIALS" :key="'social-'+s.value" >
                    <v-col class='py-0' cols=12 sm=2>
                      <v-subheader class="px-0 pt-4" :class="$vuetify.breakpoint.smAndUp?'float-right':''">
                        {{s.text}}  (<a @click="show_sns_config(s)">གསལ་བཤད།</a>)
                      </v-subheader>
                    </v-col>
                    <v-col class='py-0' cols=12 sm=3>
                      <v-text-field small hide-details single-line v-model="settings['SOCIAL_AUTH_'+s.value.toUpperCase()+'_KEY']" label="Key" type="text"></v-text-field>
                    </v-col>
                    <v-col class='py-0' cols=12 sm=7>
                      <v-text-field small hide-details single-line v-model="settings['SOCIAL_AUTH_'+s.value.toUpperCase()+'_SECRET']" label="Secret" type="text"></v-text-field>
                    </v-col>
                  </v-row>
                </template>

                <template v-if="card.show_ssl">
                  <ssl-manager />
                </template>
              </div>
            </v-card-text>
          </v-expand-transition>
        </v-card>

        <br/>
        <div class="text-center">
            <v-btn color="primary" @click="save_settings">གསོག་འཇོག</v-btn>
        </div>
    </div>
</template>

<script>
import SSLManager from "~/components/SSLManager.vue";
export default {
    components: {
        "ssl-manager": SSLManager,
    },
    created() {
        this.$backend("/admin/settings").then(rsp => {
            this.sns_items = rsp.sns;
            this.settings = rsp.settings;
            this.site_url = rsp.site_url;

            var m = {}
            rsp.sns.forEach(function(ele){
                m[ele.value] = ele;
            });
            this.settings.SOCIALS.forEach(function(ele){
                ele.help = false;
                ele.link = m[ele.value].link;
            })
        });
    },
    data: () => ({
        combo_input: "",
        sns: {},
        sns_items: [],
        settings: { },
        site_url: "",

        cards: [
            {
            show: false,
            title: "རྩ་བའི་ཆ་འཕྲིན།",
            fields: [
                { icon: "home", key: "site_title", label: "དྲ་ཚིགས་ཁ་བྱང་།", },
                { icon: "mdi-copyright", key: "HEADER", label: "དྲ་ལམ་སྤྱི་བསྒྲགས།", type: 'textarea' },
                { icon: "mdi-copyright", key: "FOOTER", label: "དྲ་ཚིགས་ཞབས་མཆན།", type: 'textarea' },
            ],
            groups: [
            {
                key: "INVITE_MODE",
                label: "སྒེར་སྤྱོད་རྣམ་པའི་ཁ་ཕྱེས།",
                fields: [
                    { icon: "lock", key: "INVITE_CODE", label: "གསང་གྲངས།" },
                    { icon: "person", key: "INVITE_MESSAGE", type: 'textarea', label: "དྲན་སྐུལ་གྱི་སྐད་ཆ།" },
                ],
            },
            ],
        },
        {
            show: false,
            title: "སྤྱོད་མཁན་སྒྲིག་འགོད།",
            fields: [
                { icon: "", key: "ALLOW_GUEST_READ", label: "ཐོ་འགོད་མ་བྱེད་པར་ལྟ་ཀློག་གི་ཆོག་མཆན་ཐོབ་པར་བགྱིས།", type: 'checkbox' },
                { icon: "", key: "ALLOW_GUEST_DOWNLOAD", label: "ཐོ་འགོད་མ་བྱེད་པར་ཕབ་ལེན་གྱི་ཆོག་མཆན་ཐོབ་པར་བགྱིས།", type: 'checkbox' },
                { icon: "", key: "ALLOW_GUEST_PUSH", label: "ཐོ་འགོད་མ་བྱེད་པར་སྒྲིག་ཆས་མཉམ་མཐུད་ཆོག་མཆན་ཐོབ་པར་བགྱིས།", type: 'checkbox' },
            ],
            groups: [
            {
                key: "ALLOW_REGISTER",
                label: "ཡིག་ཟམ་ནས་ཐོ་འགོད་བྱེད་རུང་།",
                fields: [
                    { icon: "info", key: "SIGNUP_MAIL_TITLE", label: "འཕྲིན་ཡིག་གི་ཁ་བྱང་གཏན་ཁེལ།" },
                    { icon: "info", key: "SIGNUP_MAIL_CONTENT", label: "འཕྲིན་ཡིག་གི་ནང་དོན་བཀོད་སྒྲིག", type: 'textarea' },
                    { icon: "info", key: "RESET_MAIL_TITLE", label: "གསང་གྲངས་བསྐྱར་སྒྲིག་གི་འཕྲིན་ཡིག་ཁ་བྱང་།" },
                    { icon: "info", key: "RESET_MAIL_CONTENT", label: "གསང་གྲངས་བསྐྱར་སྒྲིག་གི་འཕྲིན་ཡིག་གཞུང་དངོས།", type: 'textarea' },
                ],
            },
            ],
        },

        {
            show: false,
            title: 'སྤྱི་ཚོགས་འབྲེལ་ལམ་ནས་ཐོ་འཇུག',
            fields: [ ],
            show_socials: true,
        },
        {
            show: false,
            title: "ཡིག་ཟམ་ཞབས་ཞུ།",
            subtitle: 'ཡིག་ཟམ་ཐོ་འགོད་དང་སྒྲིག་ཆས་མཉམ་མཐུད་བཅས་ལ་སྒྲིག་འགོད་འདིར་རེག་ལས་ཡོད།',
            fields: [
                { icon: "email", key: "smtp_server", label: "ཡིག་ཟམ་ཞབས་ཞུ་སྒྲིག་ཆས་ཀྱི་དྲ་གནས།" },
                { icon: "person", key: "smtp_username", label: "ཡིག་ཟམ་ཞབས་ཞུ་སྒྲིག་ཆས་ཀྱི་སྤྱོད་མཁན་མིང་།（དཔེར་ན 123456@qq.com）" },
                { icon: "lock", key: "smtp_password", label: "ཡིག་ཟམ་ཞབས་ཞུ་སྒྲིག་ཆས་ཀྱི་གསང་གྲངས།" },
                { icon: "info", key: "smtp_encryption", label: "ཡིག་ཟམ་ཞབས་ཞུ་སྒྲིག་ཆས་ཀྱི་བདེ་འཇགས་རང་བཞིན།", type: 'select',
                    items: [{text: "SSL", value: "SSL"}, {text: "TLSཡིན་རིང་གོ་ཆོད་པའི་གདམ་གསེས།", value: "TLS"} ]
                },
            ],
            buttons: [
                { icon: "email", label: "ཚོད་ལྟའི་ཡིག་ཟམ།", action: "test_email" },
            ],
        },
        {
            show: false,
            title: "དཔེ་དེབ་རིགས་བཀར་རྟགས་བྱང་།",
            subtitle: 'རྟགས་བྱང་ནས་རིགས་བཀར་བཏང་ཟིན་པའི་དཔེ་་ཆ་དེ་ཉིད་དྲ་ངོས་ནས་རིགས་བཀར་ནང་དོན་ལྟར་དཀར་ཆག་གསར་བ་བཟོས་འགྲོ། རབ་ཡིན་ན་མཛད་པ་པོ་དང་དཔེ་རྒྱུན་ལ་ཡང་རྟགས་བྱང་བཟོ་རོགས།',
            fields: [
                { icon: "person", key: "BOOK_NAV", type: 'textarea', label: "རིགས་བཀར་རྣམ་གྲངས།" },
            ],
        },
        {
            show: false,
            title: 'མཛའ་འབྲེལ་དྲ་ཚིགས།',
            fields: [ ],
            show_friends: true,
        },

        {
            show: false,
            title: "དྲ་སྒང་གི་ཆ་འཕྲིན་ཡོང་ཁུངས།",
            fields: [
                { icon: "", key: "auto_fill_meta", label: "རང་འགུལ་སྒོས་དྲ་སྒང་ནས་ཆ་འཕྲིན་ཐོབ་པར་བགྱིས། བོད་ཡིག་ལ་གོ་མི་ཆོད།", type: 'checkbox' },
                { icon: "info", key: "douban_baseurl", label: "Doubanདྲ་ཚིགས་ཀྱི་ནང་འདྲེན་API" },
                { icon: "info", key: "douban_max_count", label: "Doubanདྲ་ཚིགསནས་ཐོན་པའི་ཁ་གྲངས།" },
            ],
            tips: [
                {
                    text: "གལ་ཏེ་མི་ཤེས་ན་འདིའི་སྐོར་བསྐྱུར་ཤོག བོད་ཡིག་དཔེ་དེབ་ལ་མིག་སྔར་ནུས་པ་འདི་བེད་སྤྱོད་གཏོང་ཐབས་མེད།",
                    link: "https://github.com/khangey",
                }
            ],
        },

        {
            show: false,
            title: "མཐ་རིམ་སྒྲིག་འགོད།",
            fields: [
                { icon: "home", key: "static_host", label: "CDNཁོངས་མིང་།" },
                // རྗེས་སུ་སྒྲིག་འགོད་འདི་ཉིད་གདམ་གསེས་ཅན་ཏུ་བཟོས་ཆོག
                { icon: "info", key: "BOOK_NAMES_FORMAT", label: "དཀར་ཆག་དང་དཔེ་དེབ་མིང་གི་རྣམ་པ།", type: 'select',
                    items: [{text: "དབྱིན་ཡིག་གི་གསལ་བྱེད་བེད་སྤྱོད།", value: "en"}, {text: "བོད་ཡིག་གི་གསལ་བྱེད་བེད་སྤྱོད།", value: "utf8"} ]
                },
                { icon: "info", key: "avatar_service", label: "www.gravatar.com འམ cravatar.cn ནས་མགོ་པར་ཞབས་ཞུ་སྤྱོད་རུང་།" },
                { icon: "info", key: "MAX_UPLOAD_SIZE", label: "ཡིག་ཆའི་ཆེ་ཆུང་ཚད་བཀག དཔེར་ན100MBལས་བརྒལ་མི་རུང་བ།" },
                { icon: "lock", key: "cookie_secret", label: "COOKIEརང་གྲུབ་གསང་གྲངས།" },
                { icon: "info", key: "scan_upload_path", label: "ཡིག་ཆ་གང་མང་ནང་འདྲེན་བྱེད་ཆོག" },
                { icon: "info", key: "push_title", label: "ཡིག་ཟམ་སྤྱི་བསྒྲགས་ཀྱི་ཁ་བྱང་།" },
                { icon: "info", key: "push_content", label: "ཡིག་ཟམ་སྤྱི་བསྒྲགས་ཀྱི་གཞུང་དངོས།" },
                { icon: "info", key: "convert_timeout", label: "དཔེ་དེབ་རྣམ་གཞག་བརྗེ་བསྒྱུར་གྱི་ཆེས་འགོར་པའི་ཆུ་ཚོད།" },
                { icon: "", key: "autoreload", label: "སྒྲིག་འགོད་བྱེད་ཐེངས་རེར་རང་འགུལ་ནས་ཞབས་ཞུ་སྒྲིག་ཆས་ནུས་སློང་བྱ་ངེས།", type: 'checkbox' },
            ],
            tips: [
                {
                    text: "དྲ་ཚིགས་ཀྱི་རྟགས་རིས་བརྗེ་དགོས་ན། གཤམ་གྱི་དྲ་ལམ་ཕྱོགས་སྟོན་ནས་བཟོ་བཅོས་བྱོས།",
                    link: "https://github.com/khangey/Palyonbook/blob/master/document/README.zh_CN.md#logo",
                }
            ],
        },

        {
            show: false,
            title: "SSLདཔང་ཡིག་དོ་དམ།",
            fields: [],
            show_ssl: true,
        },

        ],
    }),
    methods: {
        save_settings: function() {
            this.$backend("/admin/settings", {
                method: 'POST',
                body: JSON.stringify(this.settings),
            })
            .then( rsp => {
                if ( rsp.err != 'ok' ) {
                    this.$alert('error', rsp.msg);
                } else {
                    this.$alert('success', 'གསོག་འཇོག་བྱས་ཚར། ཏོག་ཙམ་འགོར་རྗེས་ནུས་པ་ཐོན་ངེས།');
                }
            });
        },
        show_sns_config: function(s) {
            var msg = `དྲ་ལམ${s.text}ལ་བསྐྱོད་ནས<a :href="${s.link}" target="_blank">དྲ་ངོས་སྒྲིག་འགོད་བྱོས།</a>གསང་གྲངས་ལེན་པ་དང་（callback URL）ལ་བརྗེ་ཞིག
            <code>${this.site_url}/auth/complete/${s.value}.do</code>`;
            this.$alert("success", msg);
        },
        test_email: function() {
            var data = new URLSearchParams();
            data.append('smtp_server', this.settings['smtp_server']);
            data.append('smtp_username', this.settings['smtp_username']);
            data.append('smtp_password', this.settings['smtp_password']);
            data.append('smtp_encryption', this.settings['smtp_encryption']);
            this.$backend("/admin/testmail", {
                method: 'POST',
                body: data,
            }).then( rsp => {
                if ( rsp.err != 'ok' ) {
                    this.$alert('error', rsp.msg);
                } else {
                    this.$alert('success', rsp.msg);
                }
            });
        },
        run: function(func) {
            this[func]();
        },
    },
  }
</script>

<style>
.cursor-pointer {
    cursor: pointer;
}
</style>

