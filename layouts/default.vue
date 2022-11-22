<template>
  <v-app dark>
    <v-app-bar :clipped-left="clipped" fixed app>
      <v-toolbar-title v-text="title" />
      <v-spacer />
    </v-app-bar>
    <v-main>
      <v-container fill-height>
        <Nuxt />
      </v-container>
    </v-main>
    <v-bottom-navigation :value="value" color="primary">
      <v-btn @click="logout">
        <span>Logout</span>
        <v-icon>mdi-logout-variant</v-icon>
      </v-btn>
    </v-bottom-navigation>
  </v-app>
</template>

<script>
export default {
  name: 'DefaultLayout',
  data() {
    return {
      clipped: false,
      fixed: false,
      title: 'Phone book',
      value: 1,
    }
  },
  methods: {
    async logout() {
      await this.$auth.logout();
      this.$router.push('/login');
    }
  }
}
</script>

<style scoped>
.v-item-group.v-bottom-navigation .v-btn {
  height: auto !important;
}
</style>
