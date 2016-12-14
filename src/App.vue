<template>
  <div id="app">
    <form @submit.prevent="submit">
      <input v-model="inputValue" type="text" placeholder="Enter a GitHub username...">
      <button>Go!</button>
    </form>
    <h1 v-text="username"></h1>
    <img v-if="avatar" :src="avatar" class="avatar">
    <h2 v-if="faveLang">😍 Favourite Language: {{faveLang}} 😍</h2>
    <h3 v-if="followers.length > 0">Followers ({{followers.length}}):</h3>
    <ul v-if="followers.length > 0">
      <li v-for="follower in followers">
        {{follower}}
      </li>
    </ul>
  </div>
</template>

<script>
import Vue from 'vue'
import axios from 'axios'
import VueAxios from 'vue-axios'
import _ from 'lodash'

Vue.use(VueAxios, axios)

export default {
  name: 'app',

  data() {
    return {
      inputValue: '',
      username: '',
      avatar: '',
      followers: [],
      faveLang: '',
      urlBase: 'https://api.github.com/users',
    }
  },

  methods: {
    submit() {
      if (!!this.inputValue) {
        const api = `${this.urlBase}/${this.inputValue}`

        this.fetchUser(api)
      }
    },

    fetchUser(api) {
      Vue.axios.get(api).then((response) => {
        const { data } = response
        
        this.inputValue = ''
        this.username = data.login
        this.avatar = data.avatar_url

        this.fetchFollowers()
        this.fetchFaveLang()
      }).catch(error => {
        console.warn('ERROR:', error)
      })
    },

    fetchFollowers() {
      Vue.axios.get(`${this.urlBase}/${this.username}/followers`).then(followersResponse => {
        this.followers = followersResponse.data.map(follower => {
          return follower.login
        })
      })
    },

    fetchFaveLang() {
      Vue.axios.get(`${this.urlBase}/${this.username}/repos`).then(reposResponse => {
        const langs = reposResponse.data.map(repo => {
          return repo.language
        })

        // I stole this:
        const faveLang = _.chain(langs).countBy().toPairs().maxBy(_.last).head().value()
        if (faveLang !== 'null') {
          this.faveLang = faveLang
        }
      })
    }
  }
}
</script>

<style lang="stylus">
body
  background-color goldenrod

#app
  display flex
  align-items center
  flex-flow column
  font-family Comic Sans MS

input
  width 320px

input,
button
  font-size 25px

h1
  font-size 44px

.avatar
  height 200px
  width 200px
  border-radius 10%
</style>
