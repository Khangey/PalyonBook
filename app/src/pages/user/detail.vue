<template>
    <v-form ref="form" @submit.prevent="save">
        <v-row align=start>
            <v-col cols=3><v-subheader class="pa-0 float-right" >མགོ་པར།</v-subheader></v-col>
            <v-col cols=9>
                <v-img class="float-left" height=80 contain :src="user.avatar"></v-img>
                <v-subheader class="" >
                    <a href="https://cravatar.cn/avatar" target="_blank">བརྗེ་བསྒྱུར།</a>
                </v-subheader>
            </v-col>

            <v-col cols=3><v-subheader class="pa-0 float-right" >སྤྱོད་མཁན་གྱི་མིང་།</v-subheader></v-col>
            <v-col cols=9><p class="pt-3 mb-0">{{user.username}}</p></v-col>

            <v-col cols=3><v-subheader class="pa-0 float-right" >ཡིག་ཟམ།</v-subheader></v-col>
            <v-col cols=9><p class="pt-3 mb-0">{{user.email}}<a href='#' v-if="!user.is_active" @click='send_active_email'>གདན་ཞུའི་ཡིག་ཟམ་ཡང་བསྐྱར་བསྐུར་རོགས།</a></p></v-col>

            <v-col cols=3><v-subheader class="pa-0 float-right" >གསང་གྲངས།</v-subheader></v-col>
            <v-col cols=9>
                <v-subheader class="pa-0" >
                    <a href="#" @click.stop="show_pass = ! show_pass">བརྗེ་བསྒྱུར།</a>
                </v-subheader>
                <div v-if="show_pass">
                    <v-text-field solo v-model="user.password0" label="མིག་སྔར་གྱི་གསང་གྲངས།" type="password" autocomplete="new-password0" :rules="[rules.pass]" ></v-text-field>
                    <v-text-field solo v-model="user.password1" label="གསང་གྲངས་གསར་བ།"   type="password" autocomplete="new-password1" :rules="[rules.pass]" ></v-text-field>
                    <v-text-field solo v-model="user.password2" label="གསང་གྲངས་གཏན་ཁེལ།" type="password" autocomplete="new-password2" :rules="[valid]"      ></v-text-field>
                </div>
            </v-col>

            <v-col cols=3><v-subheader class="pa-0 float-right" >དྲ་མིང་།</v-subheader></v-col>
            <v-col cols=9> <v-text-field solo v-model="user.nickname" label="དྲ་མིང་།" type="text" autocomplete="new-nickname" :rules="[rules.nick]"></v-text-field> </v-col>

            <v-col cols=3><v-subheader class="pa-0 float-right" >Kindleདྲ་གནས།</v-subheader></v-col>
            <v-col cols=9> <v-text-field solo v-model="user.kindle_email" label="Kindle" type="text" autocomplete="new-email" :rules="[rules.email]"></v-text-field> </v-col>
            <v-col cols=12>
                <div class="text-center">
                    <v-btn dark large rounded color="orange" @click="save">གསོག་འཇོག</v-btn>
                </div>
            </v-col>
        </v-row>
    </v-form>
</template>

<script>
export default {
    data: () => ({
        user: {},
        show_pass: false,
        rules: {
            pass: v => v == undefined || v.length == 0 || v.length >= 8 || 'Min 8 characters',
            nick: v => v == undefined || v.length == 0 || v.length >= 2 || 'Min 2 characters',
            email: function (email) {
                var re = /^(([^<>()[\]\\.,;:\s@"]+(\.[^<>()[\]\\.,;:\s@"]+)*)|(".+"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,}))$/;
                return email == undefined || email.length == 0 || re.test(email) || "Invalid email format";
            },
        },
    }),
    async asyncData({ params, app, res }) {
        if ( res !== undefined ) {
            res.setHeader('Cache-Control', 'no-cache');
        }
        return app.$backend("/user/info?detail=1");
    },
    head: () => ({
        title: "སྤྱོད་མཁན་དོ་དམ།",
    }),
    created() {
        this.init(this.$route);
    },
    beforeRouteUpdate(to, from, next) {
        this.init(to, next);
    },
    methods: {
        valid: function(v) {
            return v == this.user.password1 || "Password are not same."
        },
        save: function() {
            if ( ! this.$refs.form.validate() ) {
                return false;
            }
            var d = {
                'password0': this.user.password0,
                'password1': this.user.password1,
                'password2': this.user.password2,
                'nickname': this.user.nickname,
                'kindle_email': this.user.kindle_email,
            }
            this.$backend('/user/update', {
                method: 'POST',
                body: JSON.stringify(d),
            })
            .then( rsp => {
                if ( rsp.err != 'ok' ) {
                    this.failmsg = rsp.msg;
                } else {
                    this.$store.commit("navbar", true);
                    this.$router.push("/");
                }
            });
        },
        send_active_email: function() {
            this.$backend('/user/active/send')
            .then( rsp => {
                if ( rsp.err == 'ok' ) {
                    this.$alert("success", "གདན་ཞུའི་ཡིག་ཟམ་བསྐུར་ཟིན།");
                } else {
                    this.$alert("danger", rsp.msg);
                }
            });
        },
        init(route, next) {
            this.$store.commit('navbar', true);
            this.$backend("/user/info?detail=1")
            .then( rsp => {
                rsp.user.password0 = "";
                rsp.user.password1 = "";
                rsp.user.password2 = "";
                this.user = rsp.user;
            });
            if ( next ) next();
        },
    },
}
</script>

<style></style>
