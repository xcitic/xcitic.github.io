<template>
  <div id="app">
    <div class="container">

    <h1><i class="fas fa-star"/> Githubination </h1>


    <div class="card mb-4 mt-4 table-responsive text-nowrap elevation-4">

      <table class="table table-hover">
        <thead class="thead-dark">
          <th scope="col">Rank</th>
          <th scope="col">Repo Name</th>
          <th scope="col">Stars</th>
          <th scope="col">Forks</th>
          <th scope="col">Avatar</th>
          <th scope="col">Github Link</th>
          <th scope="col">Homepage</th>
        </thead>
        <tbody>
          <tr v-for="repo in showingRepos" :key="repo.id">
            <!-- Get the rank by location the current repo in the repos array and return the index + 1 -->
            <th scope="row">{{repos.indexOf(repo) + 1}}</th>
            <td>{{repo.name}}</td>
            <!-- Show the numbers with comma seperators for thousand -->
            <td>{{repo.stargazers_count.toLocaleString() }}</td>
            <td>{{repo.forks_count.toLocaleString() }}</td>
            <!-- If there exists an avatar, return it as an img tag -->
            <td> <img v-if="repo.owner.avatar_url" class="avatar" :src="repo.owner.avatar_url"/> </td>
            <td> <a :href="repo.html_url" target="_blank">Link</a> </td>
            <!-- If there exists a homepage, return it as an achor tag -->
            <td>
              <a v-if="repo.homepage" :href="repo.homepage" target="_blank">Homepage</a>
              <p v-else>No homepage</p>
            </td>
          </tr>
        </tbody>
      </table>
      <!-- While loading show a spinner -->
      <div v-if="loading" class="justify-content-center">
        <i class="fas fa-3x fa-spin fa-spinner"></i>
      </div>

      <hr />
      <!-- Paginations -->
      <ul class="pagination justify-content-center">
        <li class="page-item" :class="page < 1 ? 'disabled' : ''">
          <a class="page-link" @click="prevPage">Previous</a>
        </li>
        <li class="page-item">Page {{page + 1}} of {{maxPage}}</li>
        <li class="page-item" :class="(page + 1) == maxPage ? 'disabled' : ''">
          <a class="page-link" @click="nextPage">Next</a>
        </li>
      </ul>
    <!--./Pagination -->
    </div><!--./Card -->



    </div>
  </div>
</template>

<script>
import axios from 'axios'

const git_url = 'https://api.github.com/search/repositories?q=language:javascript&sort=stars&order=desc&per_page=100&page='

export default {
  name: 'app',
  data () {
    return {
      loading: true,
      repos: [],
      showingRepos: [],
      page: 0,
      maxPage: 0,
      perPage: 20,
      gitPage: 1,
    }
  },

  // when component is mounted, run the fetchRepos method to populate the repos array.
  // then run the updateShowing method to return 20 Repos in the showingRepos array.
  // then calculate the total pages in the pagination
  mounted() {
    this.fetchRepos()
    .then(() => this.updateShowing())
    .then(() => this.calculatePages())
  },

  methods: {
    fetchRepos() {
      // create a new Promise that handles async callback to the github api
      return new Promise((resolve,reject) => {
        axios.get(git_url + this.gitPage)
        .then(({data}) => {
          // add new data from api call to the repos array
          this.repos = [...this.repos, ...data.items]
          resolve()
        })
        // if any error occurs, catch them and print out to the console and return a reject from the promise
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
    },

    calculatePages() {
      this.maxPage = this.repos.length / this.perPage
    },

    nextPage() {
      // when 1 click away from maxPage, fetch more repos and add them to the repos array.
      if(this.page  == (this.maxPage - 3))
      {
        this.gitPage++
        this.fetchRepos()
            .then(() => this.calculatePages())
      }
      // show loading spinner
      this.loading = true
      //increment page count
      this.page++
      // re-run function to repopulate showingRepos array with next 20 repos
      this.updateShowing()
      // scroll to top of the list
      window.scrollTo(0,0)
    },

    prevPage() {
      this.loading = true
      // decrement page count
      this.page--
      // re-run function to repopulate showingRepos array with previous 20 repos
      this.updateShowing()
      // scroll to top
      window.scrollTo(0,0)
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

a:hover {
  cursor:pointer;
}

.avatar {
  max-width: 50px;
}
</style>
