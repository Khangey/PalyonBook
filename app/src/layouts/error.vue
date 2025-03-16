<template>
  <v-app dark>
    <h1 v-if="error.statusCode === 404">
      {{ pageNotFound }}
    </h1>
    <h1 v-else>
      {{ otherError }}
    </h1>
    <NuxtLink to="/"> Home page </NuxtLink>
  </v-app>
</template>

<script>
export default {
  name: "EmptyLayout",
  layout: "empty",

  props: {
    error: {
      type: Object,
      default: null,
    },
  },
  created() {
    //this.$store.commit("puremode", true);
  },

  data() {
    return {
      pageNotFound: "404 དྲ་ངོས་རྙེད་མ་བྱུང་།",
      otherError: "ནོར་འཁྲུལ་ག་གེ་མོ་བྱུང་འདུག",
    };
  },
  head() {
    const title = this.error.statusCode === 404 ? this.pageNotFound : this.otherError;
    return {
      title,
    };
  },
};
</script>

<style scoped>
h1 {
  font-size: 20px;
}
.msg {
  text-align: center;
}
</style>

