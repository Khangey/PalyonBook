<template>
<v-row justify=center class='fill-center'>
    <v-col xs=12 sm=8 md=4>
        <v-card class="elevation-12">
            <v-toolbar dark color="primary">
                <v-toolbar-title align-center >ཐོ་འགོད་ལེགས་འགྲུབ།</v-toolbar-title>
            </v-toolbar>
            <v-card-text>
                <p class="mt-12 mb-8 text-center">
                ཁོངས་མིང་ཐོ་འགོད་ལེགས་འགྲུབ་བྱུང་། དྲ་ངོས་སུ་སོང་ནས་བེད་སྤྱོད་གནང་རོགས།
                </p>
            </v-card-text>

            <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn to="/" color="primary">Go</v-btn>
                <v-spacer></v-spacer>
            </v-card-actions>

        </v-card>
    </v-col>
</v-row>
</template>

<script>
export default {
    created() {
        this.$store.commit('navbar', false);
    },
    data: () => ({
    }),
    head: () => ({
        title: 'ཐོ་འགོད་ལེགས་འགྲུབ།',
    }),
}
</script>

