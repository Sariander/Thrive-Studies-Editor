<template>
  <div id="app">
    <md-app md-mode="fixed">
      <md-app-toolbar class="md-primary">
        <span class="md-title" style="flex: 1">{{ title }}</span>
        <md-button v-if="isLoggedIn" @click="signOut()">Logout</md-button>
        <md-button v-if="!isLoggedIn" @click="goToSignIn()">Login</md-button>
      </md-app-toolbar>

      <md-app-drawer v-if="isLoggedIn" md-permanent="full">
        <md-toolbar class="md-transparent" md-elevation="0">
          Navigation
        </md-toolbar>

        <md-list>
          <md-list-item v-for="tab in tabs" v-bind:key="tab.id" @click="navigateToTab(tab.route, tab.key)">
            <md-icon>{{ tab.icon }}</md-icon>
            <span class="md-list-item-text">{{ tab.text }}</span>
          </md-list-item>
        </md-list>
      </md-app-drawer>

      <md-app-content>
        <router-view></router-view>
      </md-app-content>
    </md-app>
  </div>
</template>

<script>
import Vue from 'vue'
import VueMaterial from 'vue-material'
import 'vue-material/dist/vue-material.min.css'
import 'vue-material/dist/theme/default.css'
import firebase from 'firebase'
import store from './config/store'

Vue.use(VueMaterial)

export default {
  name: 'app',
  beforeCreate: function () {
    firebase.auth().onAuthStateChanged((user) => {
      if (user) {
        this.user = user
        store.commit('login')
      }
      this.loading = false
    })
  },
  data () {
    return {
      loading: true,
      user: null,
      showNavigation: false,
      title: 'Thrive Studies',
      tabs: [
        {
          id: 0,
          text: 'Home',
          key: 'home',
          route: 'Home',
          icon: 'home'
        },
        {
          id: 1,
          text: 'Follow Up',
          key: 'follow-up',
          route: 'thrive',
          icon: 'list'
        },
        {
          id: 2,
          text: 'Thrive 1',
          key: 'thrive-1',
          route: 'thrive',
          icon: 'looks_one'
        },
        {
          id: 3,
          text: 'Thrive 2',
          key: 'thrive-2',
          route: 'thrive',
          icon: 'looks_two'
        },
        {
          id: 4,
          text: 'Thrive 3',
          key: 'thrive-3',
          route: 'thrive',
          icon: 'looks_3'
        }
      ],
      lessonAdd: false,
      seriesAdd: false,
      sectionAdd: false
    }
  },
  computed: {
    isLoggedIn () {
      return store.getters.isLoggedIn
    }
  },
  mounted () {
    this.setTitle(this.$route)
  },
  watch: {
    $route (to, from) {
      this.setTitle(to)
    }
  },
  methods: {
    signInWithGoogle () {
      const provider = new firebase.auth.GoogleAuthProvider()
      firebase.auth().signInWithRedirect(provider).then((result) => {
        this.user = result.user
      }).catch(error => console.log(error))
    },
    signOut () {
      firebase.auth().signOut().then(() => {
        this.user = null
        store.commit('logout')
        this.$router.push({ name: 'Login', params: {} })
      }).catch(error => console.log(error))
    },
    navigateToTab (route, key) {
      this.$router.push({ name: route, params: { category: key } })
    },
    goToSignIn () {
      this.$router.push({ name: 'Login' })
    },
    convertTitle (str) {
      str = str.replace(/-/g, ' ')
      var splitStr = str.toLowerCase().split(' ')
      for (var i = 0; i < splitStr.length; i++) {
        splitStr[i] = splitStr[i].charAt(0).toUpperCase() + splitStr[i].substring(1)
      }
      return splitStr.join(' ')
    },
    setTitle (route) {
      this.lessonAdd = false
      this.seriesAdd = false
      this.sectionAdd = false
      switch (route.name) {
        case 'Home':
        case 'Login':
          this.title = 'Thrive Studies Editor'
          break
        case 'section':
          this.title = 'Thrive Studies Editor'
          this.sectionAdd = true
          break
        case 'sectionEdit':
          this.title = 'Thrive Studies Editor'
          break
        case 'sectionAdd':
          this.title = 'Thrive Studies Editor'
          break
        case 'followup':
          this.lessonAdd = true
          this.title = 'Follow Up'
          break
        case 'thrive':
          this.seriesAdd = true
          this.title = this.convertTitle(route.params.category)
          break
        case 'seriesEdit':
          this.title = this.convertTitle(route.params.category) + ' / ' + this.convertTitle(route.params.seriesName) + ' / Edit Series'
          break
        case 'seriesAdd':
          this.title = this.convertTitle(route.params.category) + ' / Add Series'
          break
        case 'series':
          this.lessonAdd = true
          this.title = this.convertTitle(route.params.category) + ' / ' + this.convertTitle(route.params.seriesName)
          break
        case 'lesson':
          this.title = this.convertTitle(route.params.category) + ' / ' + this.convertTitle(route.params.seriesName) + ' / ' + this.convertTitle(route.params.lessonName)
          break
        case 'lessonEdit':
          this.title = this.convertTitle(route.params.category) + ' / ' + this.convertTitle(route.params.seriesName) + ' / Edit Lesson'
          break
        case 'lessonAdd':
          this.title = this.convertTitle(route.params.category) + ' / ' + this.convertTitle(route.params.seriesName) + ' / Add Lesson'
          break
        case 'itemEdit':
          this.title = this.convertTitle(route.params.category) + ' / ' + this.convertTitle(route.params.seriesName) + ' / ' + this.convertTitle(route.params.lessonName) + ' / Edit Item'
          break
        case 'itemAdd':
          this.title = this.convertTitle(route.params.category) + ' / ' + this.convertTitle(route.params.seriesName) + ' / ' + this.convertTitle(route.params.lessonName) + ' / Add Item'
          break
        default:
      }
    }
  }
}
</script>

<style lang="scss">
.md-app {
  height: 100vh;
  max-height: 100vh;
}

.md-drawer {
  width: 230px;
  max-width: calc(100vw - 125px);
}

.md-app-content {
  padding: 0px;
}

.orange-hightlight-text {
  color: orange
}
</style>
