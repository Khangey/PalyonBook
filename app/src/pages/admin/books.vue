<template>
    <v-card>
        <v-card-title> དཔེ་དེབ་དོ་དམ། <v-chip small class="primary">Beta</v-chip> </v-card-title>
        <v-card-text> འདིར་མངོན་པ་ནི་དཔེ་དེབ་ཀྱི་ཆ་འཕྲིན་ཆ་ཚང་མ་ཡིན་པས། དེར་རྡེབ་ན་མྱུར་དུ་བཟོ་བཅོས་གཏོང་ཐུབ། དཔེ་དེབ་ཀྱི་ས་འཕྲིན་ཆ་ཚང་ཞིབ་མོའི་ནང་དོན་ལ་གཟིགས་རོགས།</v-card-text>
        <v-card-actions>
            <v-btn :disabled="loading" outlined color="primary" @click="getDataFromApi"><v-icon>mdi-reload</v-icon>གསར་འདྲེན།</v-btn>
            <v-btn :disabled="loading" outlined color="info" @click="show_dialog_auto_file"><v-icon>mdi-info</v-icon>དེབ་ཀྱི་ཆ་འཕྲིན་རང་འགུལ་གསར་འདྲེན།</v-btn>
            <v-spacer></v-spacer>
            <v-text-field cols="2" dense v-model="search" append-icon="mdi-magnify" label="འཚོལ་བཤེར།" single-line hide-details></v-text-field>
        </v-card-actions>
        <v-data-table
            dense
            class="elevation-1 text-body-2"
            show-select
            v-model="books_selected"
            item-key="id"
            :search="search"
            :headers="headers"
            :items="items"
            :options.sync="options"
            :server-items-length="total"
            :loading="loading"
            :items-per-page="100"
            :footer-props="{ 'items-per-page-options': [10, 50, 100] }"
        >
            <template v-slot:item.status="{ item }">
                <v-chip small v-if="item.status == 'ready'" class="success">འདྲེན་ཆོག་པ།</v-chip>
                <v-chip small v-else-if="item.status == 'exist'" class="lighten-4">ད་ཡོད་དཔེ་དེབ།</v-chip>
                <v-chip small v-else-if="item.status == 'imported'" class="primary">དྲངས་ཟིན་པ།</v-chip>
                <v-chip small v-else-if="item.status == 'new'" class="grey">བཤེར་འབེབ་མ་ཟིན་པ།</v-chip>
                <v-chip small v-else class="info">{{ item.status }}</v-chip>
            </template>
            <template v-slot:item.img="{ item }">
                <a target="_blank" :href="item.img"><v-img :src="item.thumb" class="my-1" max-height="80"  :aspect-ratio="3/4" /></a>
            </template>
            <template v-slot:item.id="{ item }">
                <a target="_blank" :href="`/book/${item.id}`">{{ item.id }}</a>
            </template>
            <template v-slot:item.title="{ item }">
                <v-edit-dialog large :return-value.sync="item.title" @save="save(item, 'title')" save-text="གསོག་འཇོག" cancel-text="དོར།">
                    <span class="three-lines" style="max-width: 200px; min-width: 120px; ">{{ item.title }}</span>

                    <template v-slot:input>
                        <div class="mt-4 text-h6">ས་མིག་བཟོ་བཅོས།</div>
                        <v-textarea v-model="item.title" label="དེབ་མིང་།" style="min-width: 400px" counter></v-textarea>
                    </template>
                </v-edit-dialog>
            </template>

            <template v-slot:item.author="{ item }">
                <v-edit-dialog large :return-value.sync="item.author" @save="save(item, 'authors')" save-text="གསོག་འཇོག" cancel-text="དོར།">
                    <span class="three-lines" style="max-width: 200px" v-if="item.authors">{{ item.authors.join("/") }}</span>
                    <span v-else> - </span>
                    <template v-slot:input>
                        <!-- AUTHORS -->
                        <div class="mt-4 text-h6">ས་མིག་བཟོ་བཅོས།</div>
                        <v-combobox
                            v-model="item.authors"
                            :items="item.authors"
                            label="མཛད་པ་པོ།"
                            :search-input.sync="tag_input"
                            hide-selected
                            multiple
                            small-chips
                        >
                            <template v-slot:no-data>
                                <v-list-item>
                                    <span v-if="!tag_input">མིང་གསར་བ་བསྐོང་རོགས།</span>
                                    <div v-else>
                                        <span class="subheading">ཁ་སྣོན།</span>
                                        <v-chip color="green lighten-3" label small rounded> {{ tag_input }} </v-chip>
                                    </div>
                                </v-list-item>
                            </template>
                            <!-- tag chip & close -->
                            <template v-slot:selection="{ attrs, item, parent, selected }">
                                <v-chip v-bind="attrs" color="green lighten-3" :input-value="selected" label small>
                                    <span class="pr-2"> {{ item }} </span>
                                    <v-icon small @click="parent.selectItem(item)">close</v-icon>
                                </v-chip>
                            </template>
                        </v-combobox>
                    </template>
                </v-edit-dialog>
            </template>

            <template v-slot:item.rating="{ item }">
                <v-edit-dialog large :return-value.sync="item.rating" @save="save(item, 'rating')" save-text="གསོག་འཇོག" cancel-text="དོར།">
                    <span v-if="item.rating != null">སྐར་མ་{{ item.rating }}</span>
                    <span v-else> - </span>
                    <template v-slot:input>
                        <div class="mt-4 text-h6">ས་མིག་བཟོ་བཅོས།</div>
                        <v-rating label="སྐར་མ་ཆ་འཇོག" v-model="item.rating" color="yellow accent-4" length="10" dense></v-rating>
                    </template>
                </v-edit-dialog>
            </template>

            <template v-slot:item.publisher="{ item }">
                <v-edit-dialog
                    large
                    :return-value.sync="item.publisher"
                    @save="save(item, 'publisher')"
                    save-text="གསོག་འཇོག"
                    cancel-text="དོར།"
                >
                    {{ item.publisher }}
                    <template v-slot:input>
                        <div class="mt-4 text-h6">ས་མིག་བཟོ་བཅོས།</div>
                        <v-text-field v-model="item.publisher" label="པར་ཁང་།" counter></v-text-field>
                    </template>
                </v-edit-dialog>
            </template>

            <template v-slot:item.tags="{ item }">
                <v-edit-dialog large :return-value.sync="item.tags" @save="save(item, 'tags')" save-text="གསོག་འཇོག" cancel-text="དོར།">
                    <span style="width: 200px" class="three-lines" v-if="item.tags">{{ item.tags.join("/") }}</span>
                    <span v-else> - </span>
                    <template v-slot:input>
                        <!-- TAGS -->
                        <div class="mt-4 text-h6">ས་མིག་བཟོ་བཅོས།</div>
                        <v-combobox
                            v-model="item.tags"
                            :items="item.tags"
                            label="རྟགས་བྱང་།"
                            :search-input.sync="tag_input"
                            hide-selected
                            multiple
                            small-chips
                        >
                            <template v-slot:no-data>
                                <v-list-item>
                                    <span v-if="!tag_input">རྟགས་བྱང་གསར་བའི་མིང་བསྐོང་རོགས།</span>
                                    <div v-else>
                                        <span class="subheading">རྟགས་བྱང་ཁ་སྣོན།</span>
                                        <v-chip color="green lighten-3" label small rounded> {{ tag_input }} </v-chip>
                                    </div>
                                </v-list-item>
                            </template>
                            <!-- tag chip & close -->
                            <template v-slot:selection="{ attrs, item, parent, selected }">
                                <v-chip v-bind="attrs" color="green lighten-3" :input-value="selected" label small>
                                    <span class="pr-2"> {{ item }} </span>
                                    <v-icon small @click="parent.selectItem(item)">close</v-icon>
                                </v-chip>
                            </template>
                        </v-combobox>
                    </template>
                </v-edit-dialog>
            </template>

            <template v-slot:item.comments="{ item }">
                <v-edit-dialog large :return-value.sync="item.comments" @save="save(item, 'comments')" save-text="གསོག་འཇོག" cancel-text="དོར།">
                    <span :title="item.comments" style="width: 300px" class="three-lines">{{ item.comments.substr(0, 80) }}</span>
                    <template v-slot:input>
                        <div class="mt-4 text-h6">ས་མིག་བཟོ་བཅོས།</div>
                        <v-textarea v-model="item.comments" label="ངོ་སྤྲོད་མདོ་བསྡུས།"></v-textarea>
                    </template>
                </v-edit-dialog>
            </template>

            <template v-slot:item.actions="{ item }">
                <v-menu offset-y right>
                    <template v-slot:activator="{ on }">
                        <v-btn color="primary" small v-on="on">བེད་སྤྱོད། <v-icon small>more_vert</v-icon></v-btn>
                    </template>
                    <v-list dense>
                        <v-list-item @click="delete_book(item)">
                            <v-list-item-title>དཔེ་དེབ་དོར་གསུབ།</v-list-item-title>
                        </v-list-item>
                    </v-list>
                </v-menu>
            </template>
        </v-data-table>

        <!-- 小浮窗提醒 -->
        <v-snackbar v-model="snack" top :timeout="3000" :color="snackColor">
            {{ snackText }}

            <template v-slot:action="{ attrs }">
                <v-btn v-bind="attrs" text @click="snack = false">ཁ་རྒྱག</v-btn>
            </template>
        </v-snackbar>

        <!-- 提醒拉取图书的规则说明 -->
        <v-dialog v-model="meta_dialog" persistent transition="dialog-bottom-transition" width="500">
            <v-card>
                <v-toolbar flat dense dark color="primary"> དྲན་སྐུལ།</v-toolbar>
                <v-card-title></v-card-title>
                <v-card-text>
                    <p> དྲ་ལམ་ནས་དཔེ་དེབ་མང་པོ་འདྲེན་འཇུག་བྱ་ངེས་ཡིན། འདིའི་སྐོར་བོད་ཡིག་དཔེ་དེབ་ལ་འབྲེལ་བ་མེད། མཉམ་འཇོག་དགོས་རྒྱུ་ལ།</p>
                    <p> 1. མ་ལག་སྒྲིག་འགོད་ནས་འབྲེལ་ཡོད་འཇུག་སྒོའི་ཁ་ཕྱེས་དགོས།</p>
                    <p> 2. ཕྱི་ཐུམ་མེད་པ་དང་ངོ་སྤྲོད་མདོ་བསྡུས་མེད་པའི་དཔེ་ཆ་ལ་ནུས་པ་ཐོན་ངེས།</p>
                    <p> 3. སྐར་ཆ་གཅིག་ལ་དཔེ་དེབ་གཅིག་མ་གཏོགས་མི་ཁེབས། </p>
                    <br></br>
                    <template v-if="progress.total > 0">
                        <p>ད་ལྟ་ནི་<v-btn small text link @click="refresh_progress">གསར་འདྲེན་བྱས།</v-btn></p>
                        <p>ཁྱོན་ནས་དཔེ་དེབ་ཁ་གྲངས་ {{ progress.total }}ཡོད་ལ།， {{  progress.done }} གསར་འདྲེན་བཏང་ཚར།，{{ progress.fail }} གསར་འདྲེན་བཏང་མ་ཚར།，དཔེ་དེབ་{{  progress.skip }}ལྷག་ཐག་གཅོད་བྱེད་མི་དགོས། </p>
                    </template>
                    <p v-else> ད་དུང་སྐར་མ {{auto_fill_mins}} ལ་ཐག་གཅོད་བྱ་དགོས་པས་བར་སྐབས་འདིར་དྲ་ངོས་སྒོ་མ་རྒྱག་རོགས།</p>

                    <template v-else>

                    </template>
                </v-card-text>
                <v-card-actions>
                    <v-btn @click="meta_dialog = !meta_dialog">དོར།</v-btn>
                    <v-spacer></v-spacer>
                    <v-btn color="primary" @click="auto_fill">མགོ་རྩོམ་ན་ཆོག</v-btn>
                </v-card-actions>
            </v-card>
        </v-dialog>

</v-card>
</template>

<script>
export default {
    data: () => ({
        snack: false,
        snackColor: "",
        snackText: "",
        meta_dialog: false,

        books_selected: [],
        tag_input: null,
        search: "",
        page: 1,
        items: [],
        total: 0,
        loading: false,
        options: { sortBy: ["id"], sortDesc: [true] },
        headers: [
            { text: "ཕྱི་ཤོག", sortable: false, value: "img", width: "80px" },
            { text: "ID", sortable: true, value: "id", width: "80px" },
            { text: "དཔེ་ཆའི་མིང་།", sortable: true, value: "title" },
            { text: "མཛད་པ་པོ།", sortable: true, value: "author", width: "100px" },
            { text: "ཆ་འཇོག་སྐར་མ།", sortable: false, value: "rating", width: "60px" },
            { text: "པར་ཁང་།", sortable: false, value: "publisher" },
            { text: "རྟགས་བྱང་།", sortable: true, value: "tags", width: "100px" },
            { text: "ངོ་སྤྲོད་མདོ་བསྡུས།", sortable: true, value: "comments" },
            { text: "བཅོས་སྒྲིག", sortable: false, value: "actions" },
        ],
        progress: {
            skip: 0,
            fail: 0,
            done: 0,
            total: 0,
            status: "finish",
        },
    }),
    created() {},
    watch: {
        options: {
            handler() {
                this.getDataFromApi();
            },
            deep: true,
        },
    },
    computed: {
        auto_fill_mins: function() {
            return Math.floor(this.total/60) + 1;
        }
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
            if (this.search != undefined) {
                data.append("search", this.search);
            }
            this.$backend("/admin/book/list?" + data.toString())
                .then((rsp) => {
                    if (rsp.err != "ok") {
                        this.items = [];
                        this.total = 0;
                        this.$alert("error", rsp.msg);
                        return false;
                    }
                    this.items = rsp.items;
                    this.total = rsp.total;
                })
                .finally(() => {
                    this.loading = false;
                });
        },
        refresh_progress() {
            this.$backend("/admin/book/fill", {
                method: "GET",
            })
            .then((rsp) => {
                this.progress = rsp.status;
            })
        },
        show_dialog_auto_file() {
            this.meta_dialog = true;
            this.refresh_progress();
        },
        auto_fill() {
            this.$backend("/admin/book/fill", {
                method: "POST",
                body: JSON.stringify({"idlist": "all"}),
            })
            .then((rsp) => {
                this.meta_dialog = false;
                if (rsp.err != "ok") {
                    this.$alert("error", rsp.msg);
                }
                this.snack = true;
                this.snackColor = "success";
                this.snackText = rsp.msg;
            })
        },
        delete_book(book) {
            this.loading = true;
            this.$backend("/book/" + book.id + "/delete", {
                method: "POST",
                body: "",
            })
                .then((rsp) => {
                    if (rsp.err != "ok") {
                        this.$alert("error", rsp.msg);
                    }
                    this.snack = true;
                    this.snackColor = "success";
                    this.snackText = rsp.msg;

                    this.getDataFromApi();
                })
                .finally(() => {
                    this.loading = false;
                });
        },
        save(book, field) {
            var edit = {};
            edit[field] = book[field];
            this.saving = true;
            this.$backend("/book/" + book.id + "/edit", {
                method: "POST",
                body: JSON.stringify(edit),
            }).then((rsp) => {
                if (rsp.err == "ok") {
                    this.snack = true;
                    this.snackColor = "success";
                    this.snackText = rsp.msg;
                } else {
                    this.$alert("error", rsp.msg);
                }
            });
        },
    },
};
</script>

<style>
.three-lines {
    overflow: hidden;
    text-overflow: ellipsis;
    display: -webkit-box;
    -webkit-box-orient: vertical;
    -webkit-line-clamp: 3;
    white-space: normal;
}
</style>
