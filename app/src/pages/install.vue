<template>
    <v-row align-center justify=center>
        <v-col xs=12 sm=8 md=4>
            <v-card class="elevation-12">
                <v-toolbar dark color="primary">
                    <v-toolbar-title>དཔལ་ཡོན་དཔེ་དེབ་མ་ལག་སྒྲིག་འཇུག</v-toolbar-title>
                </v-toolbar>
                <v-card-text>
                    <v-form ref="form" @submit.prevent="do_intall">
                        <v-text-field required prepend-icon="home" v-model="title" label="དྲ་ཚིགས་ཀྱི་ཁ་བྱང་།" type="text"></v-text-field>
                        <v-text-field required prepend-icon="person" v-model="username" label="དོ་དམ་པའི་ཁོངས་མིང་།"   type="text" autocomplete="new-username" :rules="[rules.user]"  ></v-text-field>
                        <v-text-field required prepend-icon="lock"   v-model="password" label="དོ་དམ་པའི་གསང་གྲངས།" type="text" autocomplete="new-password" :rules="[rules.pass]"  ></v-text-field>
                        <v-text-field required prepend-icon="email"  v-model="email"    label="དོ་དམ་པའི་ཡིག་ཟམ།"    type="text" autocomplete="new-email"    :rules="[rules.email]" ></v-text-field>
                        <v-checkbox v-model="invite" label="སྒེར་གྱི་རྣམ་པ་ཁ་ཕྱེས།"></v-checkbox>
                        <template v-if="invite">
                            <v-text-field required prepend-icon="lock"  v-model="code"    label="སྒེར་གྱི་དྲ་འཇུག་ཨང་གྲངས།"    type="text" autocomplete="new-code" ></v-text-field>
                        </template>
                    </v-form>
                    <v-alert type="info" v-if="tips" v-html="tips"></v-alert>
                </v-card-text>

                <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn @click="do_install" color="primary">སྒྲིག་འགོད་བཏང་ཚར།</v-btn>
                    <v-spacer></v-spacer>
                </v-card-actions>
            </v-card>
        </v-col>
    </v-row>
</template>

<script>
export default {
    data: () => ({
        e1: 1,
        username: "admin",
        password: "",
        email: "",
        code: "",
        invite: false,
        title: "TaleBook",
        tips: "",
        retry: 20,
        rules: {
            user: v => ( 20 >= v.length && v.length >= 5) || '6 ~ 20 characters',
            pass: v => ( 20 >= v.length && v.length >= 8) || '8 ~ 20 characters',
            email: function (email) {
                var re = /^(([^<>()[\]\\.,;:\s@"]+(\.[^<>()[\]\\.,;:\s@"]+)*)|(".+"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,}))$/;
                return re.test(email) || "Invalid email format";
            },
        },

    }),
    asyncData({ store }) {
        store.commit("navbar", false);
    },
    created() {
        this.$store.commit("navbar", false);
    },
    methods: {
        check_install: function() {
            fetch("/api/index").then( rsp => {
                if ( rsp.status == 200 ) {
                    this.tips += "APIའགྲིག་འདུག<br/>སྒྲིག་འཇུག་ལེགས་འགྲུབ་བྱུང་།";
                } else {
                    this.retry -= 1;
                    if ( this.retry > 0 ) {
                        setTimeout( () => {
                            this.check_install();
                        }, 1000);
                    } else {
                        this.tips += "འགོར་སོང་། ཡང་བསྐྱར་ཚོད་ལྟ་བྱོས།";
                    }
                    return;
                }

                // force refresh index.html
                fetch("/?r="+Math.random()).then( rsp => {
                    rsp.text();
                    this.$store.commit("navbar", true);
                    this.$router.push("/");
                });

            });
        },
        do_install: function() {
            if ( ! this.$refs.form.validate() ) {
                return false;
            }

            var data = new URLSearchParams();
            data.append('username', this.username);
            data.append('password', this.password);
            data.append('email', this.email);
            data.append('code', this.code);
            data.append('invite', this.invite);
            data.append('title', this.title);
            this.tips = "སྒྲིག་འགོད་ཡིག་ཆ་འཇུག་བཞིན་ཡོད།";
            this.$backend('/admin/install', {
                method: 'POST',
                body: data,
            })
            .then( rsp => {
                if ( rsp.err != 'ok' ) {
                    this.$alert("error", rsp.msg);
                } else {
                    this.tips += "སྒྲིག་འགོད་ཡིག་ཆ་ལེགས་འགྲུབ་བྱུང་།<br/>ཞབས་ཞུ་སྒྲིག་ཆས་ལ་ཚོད་ལྟ་བྱེད་བཞིན་ཡོད།";
                    setTimeout( () => {
                        this.check_install();
                    }, 5000);
                }
            });
        },
    },
}
</script>

