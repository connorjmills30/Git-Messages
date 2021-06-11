<template>
  <!-- <img alt="Vue logo" src="./assets/logo.png"> -->
  <!-- <HelloWorld msg="Welcome to Your Vue.js App"/> -->
  <h1>Git Messages</h1>

  <div id="input">
    <div id="username">
      <label>Username: </label>
      <input type="text" v-model="username">
      <input id="search" type="button" value="Search" @click=doSomething>
    </div>

    <br>
    
    <div id="repo">
      <label >Commit Messages for </label>
      <select class="repo-choice" name="Repos" @change="changeRepo($event)" v-model="selectedRepo">
        <option selected="true" disabled="true">Repos</option>
        <option v-for="repo in Repos" :value="repo.name" :key="repo.id">{{ repo.name }}</option>
      </select>

      <label> on </label>

      <month-picker-input :default-month="selectedMonth" :default-year="selectedYear" :input-pre-filled="true"></month-picker-input>

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
      selectedMonth: new Date().getMonth(),//((Date.now().getMonth()-1 % 11 ) + 11 ) % 11,
      selectedYear: new Date().getFullYear()
    }
  },
  methods: {
    doSomething() {
      // search button pressed; get github repos owned by provided username 
      // need to clear Repos?
      this.Repos = [];
      this.selectedRepo = "Repos"
      //if (!this.username) return;
      var urlRepos = "https://api.github.com/users/" + this.username + "/repos";
      
      fetch(urlRepos)
        .then(response => {
          console.log(response);
          return response.json();
        })
        .then(data => {
          console.log(data);
          data.forEach(repo => {
              console.log(repo.name, '(', repo.id, ')');
              this.Repos.push(repo)
          })
        });
    },
    changeRepo() {
      // new selection in repo dropdown => fetch repo data
      //this.selectedRepo = event.target.options[event.target.options.selectedIndex].text;
      console.log('repo changed to ', this.selectedRepo);
      var urlCommits = "https://api.github.com/repos/" + this.username + "/" + this.selectedRepo + "/commits";
      console.log(urlCommits);
      
      fetch(urlCommits)
        .then(response => {
          console.log(response);
          return response.json();
        })
        .then(data => {
          console.log(data);
          if(data) {
            this.repoData = data;
            data.forEach(commit => {
              var author = commit.commit.author.name;
              var message = commit.commit.message;
              console.log(author, ": ", message);
              /*
                End here for the day, todo:
                how do we want to display this data? tables? <b-table>
                mess around with ui order and formatting. is it clean and intuative?

                what kind of display options should there be? tabular, raw, verbose, brief, dates?
              */
              // launch/insert/display Messages component?
            });
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
