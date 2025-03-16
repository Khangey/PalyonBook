<template>
    <v-card>
        <v-card-title> དཔེ་དེབ་ནང་འདྲེན།<v-chip small class="primary">Beta</v-chip> </v-card-title>
        <v-card-text>
        ཡིག་ཆའི་དཀར་ཆག {{ scan_dir }}ཁྲོད་དུ་ནང་འདྲེན་བྱ་རྒྱུའི་དཔེ་དེབ་འཇོག་རོགས། ད་ལན་གོ་ཆོད་པའི་དཔེ་དེབ་ཡིག་ཆའི་རྣམ་གཞག་ནི་azw/azw3/epub/mobi/pdf/txtསོགས་ཡིན།<br/>
        ལས་རིམ་འདི་ལ་ཆུ་ཚོད་ཅུང་ཟད་འགོར་སྲིད་པས་ཏོག་ཙམ་སྒུག་དགོས། དེ་བས་དཔེ་དེབ་ནང་འདྲེན་གྱི་རིམ་པ་འདི་ཡང་དང་བསྐྱར་དུ་བསྣན་མི་དགོས།<br/>
       མ་ཟད་ད་དུང་<a target="_blank" href="https://calibre-ebook.com/">Calibreདཔེ་དེབ་མཉེན་ཆས་</a>སྤྱད་ནས་ཡིག་ཆ་དོ་དམ་བྱ་ཆོག
        </v-card-text>
        <v-card-actions>
            <v-btn :disabled="loading" outlined color="primary" @click="getDataFromApi"><v-icon>mdi-reload</v-icon>གསར་འདྲེན།</v-btn>
            <v-btn :disabled="loading" color="primary" @click="scan_books"><v-icon>mdi-file-find</v-icon>དཔེ་དེབ་ནང་འདྲེན།</v-btn>
            <template v-if="selected.length > 0">
                <v-btn :disabled="loading" color="secondary" @click="import_books"><v-icon>mdi-import</v-icon>གདམ་ཟིན་པའི་དཔེ་དེབ་ནང་འདྲེན།</v-btn>
                <v-btn :disabled="loading" outlined color="primary" @click="delete_record"><v-icon>mdi-delete</v-icon>དོར་གསུབ།</v-btn>
            </template>
            <template v-else>
                <v-btn :disabled="loading" color="warning" @click="import_books"><v-icon>mdi-import</v-icon>དཔེ་དེབ་ཆ་ཚང་ནང་འདྲེན།</v-btn>
            </template>
        </v-card-actions>
        <v-card-text>
            <div v-if="selected.length == 0">ཐག་གཅོད་བྱ་དགོས་པའི་དཔེ་དེབ་གདམ་གསེས་གཏོང་རོགས།</div>
            <div v-else>ཁྱོན་སྡོམ་དཔེ་ཆ{{ selected.length }}གདམ་གསེས་བཏང་།</div>
        </v-card-text>
        <v-tabs v-model="filter_type" @change="getDataFromApi">
            <v-tab href="#todo">ཐག་གཅོད་བྱ་རྒྱུའི་དཔེ་ཆ། ({{ count_todo }})</v-tab>
            <v-tab href="#done">དྲངས་ཟིན་པའི་དཔེ་ཆ། ({{ count_done  }})</v-tab>
        </v-tabs>
        <v-data-table
            dense
            class="elevation-1 text-body-2"
            show-select
            v-model="selected"
            item-key="hash"
            :search="search"
            :headers="headers"
            :items="items"
            :options.sync="options"
            :server-items-length="total"
            sort-by="create_time"
            sort-desc="true"
            :loading="loading"
            :page.sync="page"
            :items-per-page="100"
            :footer-props="{ 'items-per-page-options': [10, 50, 100, 1000, 5000, 10000] }"
        >
            <template v-slot:item.status="{ item }">
                <v-chip small v-if="item.status == 'ready'" class="success">འདྲེན་རུང་བ།</v-chip>
                <v-chip small v-else-if="item.status == 'exist'" class="lighten-4">ད་ཡོད།</v-chip>
                <v-chip small v-else-if="item.status == 'imported'" class="primary">འདྲེན་འཇུག་ལེགས་འགྲུབ།</v-chip>
                <v-chip small v-else-if="item.status == 'new'" class="grey">ནང་འདྲེན་བྱ་རྒྱུའི་དཔེ་དེབ།</v-chip>
                <v-chip small v-else class="info">{{ item.status }}</v-chip>
            </template>
            <template v-slot:item.title="{ item }">
                དཔེ་ཆའི་མིང་།书名：<span v-if="item.book_id == 0"> {{ item.title }} </span>
                <a v-else target="_blank" :href="`/book/${item.book_id}`">{{ item.title }}</a> <br />
                མཛད་པ་པོ།{{ item.author }}
            </template>
        </v-data-table>
    </v-card>
</template>

<script>
export default {
    data: () => ({
        filter_type: "todo",
        selected: [],
        scan_dir: "/data/books/imports/",
        search: "",
        page: 1,
        items: [],
        total: 0,
        loading: false,
        options: {},
        count_todo: 0,
        count_done: 0,
        headers: [
            { text: "ID", sortable: true, value: "id" },
            { text: "རྣམ་པ།", sortable: true, value: "status" },
            { text: "ཡིག་ཆའི་དཀར་ཆག", sortable: true, value: "path" },
            { text: "ནང་འདྲེན་དོན་ཚན།", sortable: false, value: "title" },
            { text: "ཆུ་ཚོད།", sortable: true, value: "create_time", width: "200px" },
        ],
        progress: {
            done: 0,
            total: 0,
            status: "finish",
        },
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
            data.append("filter", this.filter_type);
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
            this.$backend("/admin/scan/list?" + data.toString())
                .then((rsp) => {
                    if (rsp.err != "ok") {
                        this.items = [];
                        this.total = 0;
                        alert(rsp.msg);
                        return false;
                    }
                    this.items = rsp.items;
                    this.total = rsp.total;
                    this.scan_dir = rsp.scan_dir;
                    this.count_done = rsp.summary.done;
                    this.count_todo = rsp.summary.todo;
                })
                .finally(() => {
                    this.loading = false;
                });
        },
        loop_check_status(url, callback) {
            setTimeout(() => {
                this.$backend(url)
                    .then((rsp) => {
                        if (rsp.err != "ok") {
                            this.$alert("error", rsp.msg);
                            return;
                        }
                        if (callback(rsp)) {
                            setTimeout(() => {
                                this.loop_check_status(url, callback);
                            }, 1000);
                        } else {
                            this.getDataFromApi();
                            this.$alert("info", "ཐག་གཅོད་བྱས་ཚར།");
                        }
                    })
            }, 2000);
        },
        scan_books() {
            this.loading = true;
            this.$backend("/admin/scan/run", {
                method: "POST",
            }).then((rsp) => {
                    if (rsp.err !== "ok") {
                        this.$alert("error", rsp.msg);
                        return;
                    }

                    //this.check_scan_status();
                    this.loop_check_status("/admin/scan/status", (rsp) => {
                        this.scan = rsp.status;
                        this.count_done = rsp.summary.done;
                        this.count_todo = rsp.summary.todo;
                        if (this.scan.new === 0) {
                            this.loading = false;
                            return false;
                        }
                        this.loading = true;
                        return true;
                    });
                })
        },
        import_books() {
            this.loading = true;
            var hashlist = "all";
            if (this.selected.length > 0) {
                hashlist = this.selected.map((v) => {
                        return v.hash;
                    });
            }
            this.$backend("/admin/import/run", {
                method: "POST",
                body: JSON.stringify({
                    hashlist: hashlist,
                }),
            })
                .then((rsp) => {
                    if (rsp.err !== "ok") {
                        this.$alert("error", rsp.msg);
                    }

                    //this.check_import_status();
                    this.loop_check_status("/admin/import/status", (rsp) => {
                        this.import = rsp.status;
                        this.count_done = rsp.summary.done;
                        this.count_todo = rsp.summary.todo;
                        if (this.import.ready === 0) {
                            this.loading = false;
                            return false;
                        }
                        this.loading = true;
                        return true;
                    });
                })
        },
        delete_record() {
            console.log(this.selected);
            this.loading = true;
            this.$backend("/admin/scan/delete", {
                method: "POST",
                body: JSON.stringify({
                    hashlist: this.selected.map((v) => {
                        return v.hash;
                    }),
                }),
            })
                .then((rsp) => {
                    if (rsp.err !== "ok") {
                        this.$alert("error", rsp.msg);
                    }
                    this.getDataFromApi();
                })
                .finally(() => {
                    this.loading = false;
                });
        },
        mark_as(status) {
            this.loading = true;
            this.$backend("/admin/scan/mark", {
                method: "POST",
                body: JSON.stringify({ hashlist: this.selected, status: status }),
            })
                .then((rsp) => {
                    if (rsp.err !== "ok") {
                        this.$alert("error", rsp.msg);
                    }
                })
                .finally(() => {
                    this.loading = false;
                });
            this.items.map((v) => {
                if (this.selected.indexOf(v.hash)) {
                    v.status = status;
                }
            });
        },
    },
};
</script>
