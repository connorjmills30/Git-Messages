<template>
  <!-- <img alt="Vue logo" src="./assets/logo.png"> -->
  <!-- <HelloWorld msg="Welcome to Your Vue.js App"/> -->
  <h1>Local Git Messages</h1>

  <div id="input">
    <div id="username">
      <label>Username: </label>
      <input type="text" v-model="username">
      <input id="search" type="button" value="Search" @click=doSomething>
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

  <Messages v-bind:data="repoData"/>
  
</template>

<script>
import Messages from './components/Messages.vue'
import { MonthPickerInput } from 'vue-month-picker'

export default {
  name: 'App',
  components: {
    Messages,
    MonthPickerInput
  },
  data() {
    return {
      username: '',
      Repos: [],
      selectedRepo: "Repos",
      repoData: [],      
      dummyRepoData: [
        { 
          "sha": "9876543210",
          "commit": {
            "author": {
              "name": "Connor",
              "date": "2021-06-10T06:24:09Z"
            },
            "message": "first message"
          }
        },
          {
          "sha": "1234567890",
          "commit": {
            "author": {
              "name": "Dante",
              "date": "2021-06-11T06:24:09Z"
            },
            "message": "Dante's second message."
          }
        }        
      ],
      currentMonth: new Date().getMonth(),//((Date.now().getMonth()-1 % 11 ) + 11 ) % 11,
      currentYear: new Date().getFullYear(),
      pat: ""
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
    doSomething() {
      // search button pressed; get github repos owned by provided username 
      if (!this.username) {
        return;
      } else {
        this.testAuthentication();
      }
      // clear Repos before we search for a new set
      this.Repos = [];
      this.selectedRepo = "Repos"
      //if (!this.username) return;
      var urlRepos = "https://api.github.com/users/" + this.username + "/repos";
      
      fetch(urlRepos)
        .then(response => {
          return response.json();
        })
        .then(data => {
          data.forEach(repo => {
              //console.log(repo.name, '(', repo.id, ')');
              this.Repos.push(repo)
          })
        });
    },
    fetchMessages() {
      // called when Repo dropdown changes or vue-month-picker changes
      if(!this.selectedRepo || !this.username) return; 
            
      let sinceKey =  this.preferredYear + "-" + this.preferredMonth + "-01T00:00:00Z"
      let untilKey = this.preferredYear + "-" + this.preferredMonth + "-31T59:59:59Z"
      var urlCommits = "https://api.github.com/repos/" + this.username + "/" + this.selectedRepo + "/commits?since=" + sinceKey 
        + "&until=" + untilKey;
      //console.log(urlCommits);
      
      fetch(urlCommits)
        .then(response => {
          //console.log(response);
          return response.json();
        })
        .then(data => {
          //console.log(data);
          if(data && data.message != "Not Found") {
            this.repoData = data;
          }    
      });
    },
    updateDate(date) {
      //console.log(date);
      // update date state
      this.currentMonth = date.monthIndex;
      this.currentYear = date.year;

      // fetch new messages
      this.fetchMessages();
    },
    testAuthentication() {
      const client_id = "ecb0b5ae0acc89d32cee";
      //const redirect_uri = "The URL in your application where users will be sent after authorization";
      const local_redirect_uri = "localhost:8080";
      const login = this.username;
      //const scope = "space-delimited list of scopes"
      const state = "this should be an unguessable random string but it currently is pretty guessable" // generate this
      //const allow_signup = "true"
      let authRequest = "https://github.com/login/oauth/authorize?login=" + login + "&client_id=" + client_id
        + "&state=" + state + "&redirect_uri=" + local_redirect_uri;

      fetch(authRequest)
        .then(response => {
          console.log(response);
          return response.json();
        })
        .then(data => {
          //console.log(data);
          if(data && data.message != "Not Found") {
            console.log(data);
          }    
      });
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
