<template>
  <div id="app">
    <div class="container">
      <table class="table">
        <thead>
          <th>Rank</th>
          <th>Repo Name</th>
          <th>Stars</th>
          <th>Forks</th>
          <th>Logo</th>
          <th>Github Link</th>
          <th>Homepage</th>
        </thead>
        <tbody>
          <div v-if="loading" class="text-center">
            <h1>Loading</h1>
          </div>
          <tr v-for="repo in showingRepos" :key="repo.id">
            <!-- Get the rank by location the current repo in the repos array and return the index + 1 -->
            <td>{{repos.indexOf(repo) + 1}}</td>
            <td>{{repo.name}}</td>
            <td>{{repo.stargazers_count}}</td>
            <td>{{repo.forks_count}}</td>
            <td v-if="repo.owner.avatar_url"> <img class="avatar" :src="repo.owner.avatar_url"/> </td>
            <td> <a :href="repo.html_url" target="_blank">Link</a> </td>
            <td v-if="repo.homepage"><a :href="repo.homepage" target="_blank">Homepage</a> </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  name: 'app',
  data () {
    return {
      loading: true,
      repos: [],
      showingRepos: [],
      page: 0,
      perPage: 20,
    }
  },

  // when component is mounted, run the fetchRepos method to populate the repos array.
  // then run the updateShowing method to return 20 Repos in the showingRepos array.
  mounted() {
    this.fetchRepos()
    .then(() => this.updateShowing())
  },

  methods: {
    fetchRepos() {
      // create a new Promise that handles async callback to the github api
      return new Promise((resolve,reject) => {
        axios.get('https://api.github.com/search/repositories?q=language:javascript&sort=stars&order=desc&per_page=100')
        .then(({data}) => {
          this.repos = data.items
          resolve()
        })
        .catch((err) => {
          console.log(err)
          reject(err)
        })
      })
    },

    updateShowing() {
      // slice(start, end) gets the elements (perPage = 20) from the repos array (of 100 elements)
      // and puts them into showingRepos array
      this.showingRepos = this.repos.slice(this.perPage * this.page, ((this.perPage * this.page) + this.perPage))
      this.loading = false
    }
  }
}
</script>

<style lang="scss">
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

h1, h2 {
  font-weight: normal;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  display: inline-block;
  margin: 0 10px;
}

a {
  color: #42b983;
}

.avatar {
  max-width: 50px;
}
</style>
