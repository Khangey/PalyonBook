<template>
    <v-card>
        <v-card-title> སྤྱོད་མཁན་དོ་དམ། </v-card-title>
        <v-data-table
            :headers="headers"
            :items="items"
            :options.sync="options"
            :server-items-length="total"
            :loading="loading"
            :items-per-page="10"
            :footer-props="{ 'items-per-page-options': [10, 50, 100] }"
            class="elevation-1"
        >
            <template v-slot:item.login_ip="{ item }">
                {{ item.extra.login_ip }}
            </template>
            <template v-slot:item.detail="{ item }">
                <span v-if="item.extra.visit_history"> ལྟ་ཞིབ་ཁ་གྲངས་ནི་{{ item.extra.visit_history.length }} </span>
                <span v-if="item.extra.read_history"> ལྟ་ཀློག་ཁ་གྲངས་ནི་{{ item.extra.read_history.length }} </span>
                <span v-if="item.extra.push_history"> མཉམ་སྤྱོད་ཁ་གྲངས་ནི་{{ item.extra.push_history.length }} </span>
                <span v-if="item.extra.download_history"> ཕབ་ལེན་ཁ་གྲངས་ནི་{{ item.extra.download_history.length }} </span>
                <span v-if="item.extra.upload_history"> སྐྱེལ་འཇོག་ཡར་སྤྲོད་ཁ་གྲངས་ནི་{{ item.extra.upload_history.length }} </span>
            </template>
            <template v-slot:item.actions="{ item }">
                <v-menu offset-y right>
                    <template v-slot:activator="{ on }">
                        <v-btn color="primary" small v-on="on">བཅོས་སྒྲིག<v-icon small>more_vert</v-icon></v-btn>
                    </template>
                    <v-list dense>
                        <v-subheader>སྤྱོད་མཁན་དབང་ཚད་བཟོ་བཅོས།</v-subheader>
                        <template v-for="perm in permissions">
                            <v-list-item :key="'disable-' + perm.name" v-if="item[perm.name]">
                                <v-list-item-title
                                    ><v-icon color="success">mdi-account-check</v-icon> ཆོག་མཆན་བྱིན་ཡོད།{{ perm.text }}
                                </v-list-item-title>
                                <v-list-item-action>
                                    <v-btn
                                        text
                                        small
                                        color="error"
                                        @click="
                                            setuser(item.id, { permission: perm.code.toUpperCase() });
                                            item[perm.name] = !item[perm.name];
                                        "
                                    >
                                        ཁ་རྒྱག
                                    </v-btn>
                                </v-list-item-action>
                            </v-list-item>
                            <v-list-item :key="'enable-' + perm.name" v-else>
                                <v-list-item-title
                                    ><v-icon color="danger">mdi-account-remove</v-icon> བཀག་ཟིན།{{ perm.text }}
                                </v-list-item-title>
                                <v-list-item-action>
                                    <v-btn
                                        text
                                        small
                                        color="primary"
                                        @click="
                                            setuser(item.id, { permission: perm.code.toLowerCase() });
                                            item[perm.name] = !item[perm.name];
                                        "
                                    >
                                        ཁ་ཕྱེས།
                                    </v-btn>
                                </v-list-item-action>
                            </v-list-item>
                        </template>

                        <v-divider></v-divider>
                        <v-subheader>ཐོ་མིང་དོ་དམ།</v-subheader>
                        <v-list-item
                            v-if="!item.is_active"
                            @click="
                                setuser(item.id, { active: true });
                                item.is_active = true;
                            "
                        >
                            <v-list-item-title> ཡིག་ཟམ་མ་བརྒྱུད་པར་ཐད་ཀར་དབང་ཚད་སྤྲོད། </v-list-item-title>
                        </v-list-item>
                        <v-list-item
                            v-if="item.is_admin"
                            @click="
                                setuser(item.id, { admin: false });
                                item.is_admin = !item.is_admin;
                            "
                        >
                            <v-list-item-title> དོ་དམ་པའི་དབང་ཚར་དོར་རྒྱུ།</v-list-item-title>
                        </v-list-item>
                        <v-list-item
                            v-else
                            @click="
                                setuser(item.id, { admin: true });
                                item.is_admin = item.is_admin = !item.is_admin;
                            "
                        >
                            <v-list-item-title> དོ་དམ་པར་བསྐོ་རྒྱུ། </v-list-item-title>
                        </v-list-item>
                        <v-list-item
                            @click="
                                setuser(item.id, { delete: item.username });
                                getDataFromApi()
                            "
                        >
                            <v-list-item-title> སྤྱོད་མཁན་འདི་གསུབ་རྒྱུ། </v-list-item-title>
                        </v-list-item>
                    </v-list>
                </v-menu>
            </template>
        </v-data-table>
    </v-card>
</template>

<script>
export default {
    data: () => ({
        page: 1,
        items: [],
        total: 0,
        loading: true,
        options: { sortBy: ["access_time"], sortDesc: [true] },
        headers: [
            { text: "ID", sortable: true, value: "id" },
            { text: "སྤྱོད་མཁན་མིང་།", sortable: true, value: "username" },
            { text: "དྲ་མིང་།", sortable: false, value: "name" },
            { text: "ཡིག་ཟམ།", sortable: true, value: "email" },
            { text: "ཐོ་འགོད་བྱེད་ཐབས།", sortable: false, value: "provider" },
            { text: "ཐོ་འགོད་དུས་ཚོད།", sortable: true, value: "create_time" },
            { text: "ཐོ་འཇུག་དུས་ཚོད།", sortable: true, value: "access_time" },
            { text: "ཐོ་འཇུག་དྲ་གནས་IP", sortable: false, value: "login_ip" },
            { text: "ཞིབ་ཆ།", sortable: false, value: "detail" },
            { text: "བཅོས་སྒྲིག", sortable: false, value: "actions" },
        ],
        permissions: [
            { code: "l", name: "can_login", text: "ཐོ་འཇུག" },
            { code: "u", name: "can_upload", text: "སྐྱེལ་འཇོག" },
            { code: "s", name: "can_save", text: "ཕབ་ལེན།" },
            { code: "e", name: "can_edit", text: "བཟོ་བཅོས།" },
            { code: "d", name: "can_delete", text: "དོར་གསུབ།" },
            { code: "p", name: "can_push", text: "མཉམ་སྤྱོད།" },
            { code: "r", name: "can_read", text: "དྲ་ལམ་ལྟ་ཀློག" },
        ],
    }),
    watch: {
        options: {
            handler() {
                this.getDataFromApi();
            },
            deep: true,
        },
    },
    mounted() {
        this.getDataFromApi();
    },
    computed: {
        pageCount: function () {
            return parseInt(this.total / 20);
        },
    },
    methods: {
        getDataFromApi() {
            this.loading = true;
            const { sortBy, sortDesc, page, itemsPerPage } = this.options;

            var data = new URLSearchParams();
            if (page != undefined) {
                data.append("page", page);
            }
            if (sortBy != undefined) {
                data.append("sort", sortBy);
            }
            if (sortDesc != undefined) {
                data.append("desc", sortDesc);
            }
            if (itemsPerPage != undefined) {
                data.append("num", itemsPerPage);
            }
            this.$backend("/admin/users?" + data.toString())
                .then((rsp) => {
                    if (rsp.err != "ok") {
                        this.items = [];
                        this.total = 0;
                        alert(rsp.msg);
                        return false;
                    }
                    this.items = rsp.users.items;
                    this.total = rsp.users.total;
                })
                .finally(() => {
                    this.loading = false;
                });
        },
        setuser(uid, action) {
            action.id = uid;
            this.$backend("/admin/users", {
                body: JSON.stringify(action),
                method: "POST",
            }).then((rsp) => {
                if (rsp.err != "ok") {
                    this.$alert("error", rsp.msg);
                }
            });
        },
    },
};
</script>
