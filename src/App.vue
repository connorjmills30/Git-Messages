<template>
  <!-- <img alt="Vue logo" src="./assets/logo.png"> -->
  <!-- <HelloWorld msg="Welcome to Your Vue.js App"/> -->
  <h1>Git Messages</h1>

  <div id="input">
    <div id="username">
      <label>Username: </label>
      <input type="text" v-model="userInput" @keyup.enter=listRepos>
      <input id="search" type="button" value="List Repos" @click=listRepos>
    </div>

    <br>
    
    <div id="repo">
      <label >Commit Messages for </label>
      <select class="repo-choice" name="Repos" @change="fetchMessages" v-model="selectedRepo">
        <option selected="true" disabled="true">Repos</option>
        <option v-for="repo in Repos" :value="repo.name" :key="repo.id">{{ repo.name }}</option>
      </select>

      <label> on </label>

      <month-picker-input :default-month="preferredMonth" :default-year="preferredYear" :input-pre-filled="true" @change="updateDate"></month-picker-input>

    </div>

  </div>

  <br><br>

  <Messages v-bind:data="repoData" v-bind:user="username"/>
  
</template>

<script>
import Messages from './components/Messages.vue'
import { MonthPickerInput } from 'vue-month-picker'
import { request } from "@octokit/request";

export default {
  name: 'App',
  components: {
    Messages,
    MonthPickerInput
  },
  data() {
    return {
      userInput: '',
      username: '',
      token: "",
      Repos: [],
      selectedRepo: "Repos",
      repoData: [],      
      currentMonth: new Date().getMonth(),//((Date.now().getMonth()-1 % 11 ) + 11 ) % 11,
      currentYear: new Date().getFullYear(),
    }
  },
  computed: {
    preferredMonth: function() {
      if (this.currentMonth == 0) {
        return 12;
      } else {
        return this.currentMonth;
      }
    },
    preferredYear: function() {
      return this.currentYear;
    }
  },
  methods: {
    listRepos() {
      // search button pressed; get github repos for the user

      // clear Repos and user state before we search for a new set
      this.username = "";
      this.token = "";
      this.Repos = [];
      this.selectedRepo = "Repos"
      
      // is the userInput a token, username, or an organization?
      const input = this.userInput;

      const requestWithAuth = request.defaults({
        headers: {
          authorization: "token " + input,
        },
      });
      requestWithAuth("GET /user")
        .then(response => {
          //console.log('userinput is a token')
          // userInput is a token => set state => fetch repos
          if(response.data && response.data.login) {
            this.username = response.data.login;
          }
          this.token = input;
          this.fetchReposWithToken();
        })
        .catch(error => {
          console.log(error);
          // userinput is not a token; is it a username?
          request("GET /users/{username}", {username: input})
            .then(response => {
              //console.log("input is a username");
              if(response.data && response.data.login) {
                this.username = response.data.login;
              }
              this.fetchReposWithUsername();
            })
            .catch(error => {
              console.log(error);
              //request("GET /orgs/{org}")
            })
        }) 

    },
    fetchMessages() {
      // called when Repo dropdown changes or vue-month-picker changes
      if(!this.selectedRepo || !this.username) return; 
      
      let sinceDate =  this.preferredYear + "-" + this.preferredMonth + "-01T00:00:00Z"
      let untilDate = this.preferredYear + "-" + this.preferredMonth + "-31T59:59:59Z"

      if (this.token) {
        const requestWithAuth = request.defaults({
          headers: {
            authorization: "token " + this.token,
          },
        });
        const getCommits =  "GET /repos/" + this.username + "/" + this.selectedRepo + "/commits?since=" + sinceDate + "&until=" + untilDate;
        requestWithAuth(getCommits)
          .then(response => {
            //console.log(response.data);
            this.repoData = response.data;
          })  
      } else if (this.username) {
        
        request("GET /repos/{owner}/{repo}/commits", {
          owner: this.username,
          repo: this.selectedRepo,
          since: sinceDate,
          until: untilDate
        }).then(response => {
          //console.log(response.data);
          this.repoData = response.data;
        })
      }
    },
    updateDate(date) {
      // update date state
      this.currentMonth = date.monthIndex;
      this.currentYear = date.year;

      // fetch new messages
      this.fetchMessages();
    },
    fetchReposWithToken() {
      
      const requestWithAuth = request.defaults({
        headers: {
          authorization: "token " + this.token,
        },
      });
      
      requestWithAuth("GET /user/repos")
        .then(response => {
          response.data.forEach(repo => {
            this.Repos.push(repo)
          })
        })  
    },
    fetchReposWithUsername() {
      
      request("GET /users/{username}/repos", {username: this.username})
        .then(response => {
          if (response.data ) {
            response.data.forEach(repo => this.Repos.push(repo));
          }
        })
        .catch(error => {
          console.log(error);
        })

    }
  }
}

</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

#input {
  color: #075f0b;
}

</style>
