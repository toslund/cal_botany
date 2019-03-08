<template>
  <v-app id="inspire">
    <v-navigation-drawer
      v-model="drawer"
      fixed
      app
    >
      <v-list dense>
        <v-list-tile @click="fakeRouter('welcome')">
          <v-list-tile-action>
            <v-icon>home</v-icon>
          </v-list-tile-action>
          <v-list-tile-content>
            <v-list-tile-title>About</v-list-tile-title>
          </v-list-tile-content>
        </v-list-tile>
        <v-list-tile @click="fakeRouter('quiz')">
          <v-list-tile-action>
            <v-icon>local_florist</v-icon>
          </v-list-tile-action>
          <v-list-tile-content>
            <v-list-tile-title>Study Flora</v-list-tile-title>
          </v-list-tile-content>
        </v-list-tile>
      </v-list>
    </v-navigation-drawer>
    <v-toolbar class="light-green darken-4 white--text" dark fixed app>
      <v-toolbar-side-icon @click.stop="drawer = !drawer"></v-toolbar-side-icon>
      <v-toolbar-title>Learn California Plants</v-toolbar-title>
    </v-toolbar>
    <v-content>
      <v-container fluid fill-height>
        <v-layout
          justify-center
          align-center
        >
          <v-flex text-xs-center>
            <welcome v-if="fakeRoute == 'welcome'"></welcome>
            <quiz v-show="fakeRoute == 'quiz'"></quiz>
          </v-flex>
        </v-layout>
      </v-container>
    </v-content>
    <v-footer class="light-green darken-4 white--text " app>
      <v-layout
      justify-center
      row
      wrap
      >
        <span class="white--text">toslund@irconservancy.org</span>
      </v-layout>
    </v-footer>
  </v-app>
</template>

<script>
import Quiz from './components/Quiz.vue';
import Welcome from './components/Welcome.vue';


export default {
  name: 'App',
  components: {
    Welcome,
    Quiz,
  },
  data () {
    return {
      drawer: null,
      fakeRoute: 'welcome'
      //
    }
  },
  methods: {
    fakeRouter: function(route) {
      if (this.fakeRoute === route) { return null; }
      else { this.fakeRoute = route; }
    }
  }
}
</script>
