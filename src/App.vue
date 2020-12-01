<template>
  <v-app id="inspire">
    <v-app-bar app dense v-if="false">
      <v-app-bar-nav-icon @click="drawer = !drawer"></v-app-bar-nav-icon>

      <v-toolbar-title>
        <router-link class="pointer" tag="span" to="/"> Vue Application </router-link>
      </v-toolbar-title>
      <v-spacer></v-spacer>
      <v-toolbar-items class="hidden-sm-and-down">
        <v-btn flat v-for="link of links" :key="link.title" :to="link.url">
          <v-icon left>{{ link.icon }}</v-icon>
          {{ link.title }}
        </v-btn>
        <v-btn flat @click="onLogout" v-if="isUserLoggedIn">
          <v-icon left>exit_to_app</v-icon>
          Logout
        </v-btn>
      </v-toolbar-items>
    </v-app-bar>

    <v-main class="grey lighten-3">
      <router-view></router-view>
      <template v-if="error">
        <v-snackbar class="error-msg" :timeout="0" :multi-line="true" color="error" @input="closeError" :value="true">
          {{ error }}
          <v-btn flat dark @click.native="closeError">Close</v-btn>
        </v-snackbar>
      </template>
    </v-main>
  </v-app>
</template>

<script>
export default {
  data() {
    return {
      drawer: false,
    };
  },
  computed: {
    error() {
      return this.$store.getters.error;
    },
    isUserLoggedIn() {
      return this.$store.getters.isUserLoggedIn;
    },
    links() {
      if (this.isUserLoggedIn) {
        return [
          { title: "Orders", icon: "bookmark_border", url: "/orders" },
          { title: "New ad", icon: "note_add", url: "/new" },
          { title: "My ads", icon: "list", url: "/list" },
        ];
      } else {
        return [
          { title: "Login", icon: "lock", url: "/login" },
          { title: "Registration", icon: "face", url: "/registration" },
        ];
      }
    },
  },
  methods: {
    closeError() {
      this.$store.dispatch("clearError");
    },
    onLogout() {
      this.$store.dispatch("logoutUser");
      this.$router.push("/");
    },
  },
};
</script>

<style scoped>
.pointer {
  cursor: pointer;
}
.error-msg {
  margin-bottom: 50px;
  box-sizing: content-box;
}
</style>
