<template>
    <v-row>
        <v-col cols=12 class='text-center'>
            <v-divider class='mt-10 mb-3' ></v-divider>
            <p class='mb-0 text-center footer-text' v-html="footer_text"></p>
            <p>
            <v-btn small text target="_blank" href="https://palyon.net">དཔལ་ཡོན་དྲ་མང་།</v-btn>
            | <v-btn small text target="_blank" href="https://mt.tibetbaiyun.com">དཔལ་ཡོན་རིག་ནུས་ཡིག་སྒྱུར།</v-btn>
            | <v-btn small text target="_blank" href="http://www.tibetanfont.com">བོག་ཡིག་གློག་རྡུལ་ཡིག་གཟུགས།</v-btn>
            </p>
        </v-col>
    </v-row>
</template>

<script>
export default {
    name: 'AppFooter',
    computed: {
        footer_text: function() {
            if ( this.$store.state.sys.footer != undefined ) {
                return this.$store.state.sys.footer;
            }
            return this.footer;
        },
    },
    data: () => ({
        footer: "དྲ་ཚིགས་དང་དཔེ་ཆའི་མཛོད་འདི་ནི་བོད་ལྗོངས་དཔལ་ཡོན་ཆ་འཕྲིན་ཚན་ཡོད་ཀུང་སིས་བཟོས། ｜ 藏ICP备2023000237号-10",
    })
}
</script>

<style>

</style>
