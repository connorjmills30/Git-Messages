<template>
  <!-- <img alt="Vue logo" src="./assets/logo.png"> -->
  <!-- <HelloWorld msg="Welcome to Your Vue.js App"/> -->
  <h1>Git Messages</h1>

  <div id="input">
    <div id="username">
      <label>Username: </label>
      <input type="text" v-model="username"> 
    </div>

    <br>
    <input id="search" type="button" value="Search" @click=doSomething>
    <br><br>

    <div id="repo">
      <label>Repo: </label>
      <select class="repo-choice" @change="changeRepo($event)">
        <option value="" selected disabled>Repos</option>
        <option v-for="repo in Repos" :value="repo.name" :key="repo.id">{{ repo.name }}</option>
      </select>
    </div>

    <br>

    
    
  </div>
</template>

<script>
//import HelloWorld from './components/HelloWorld.vue'
console.log('begin script');

export default {
  name: 'App',
  components: {
    //HelloWorld
  },
  data() {
    return {
      username: 'colbybhearn',
      Repos: [],
      selectedRepo: null
    }
  },
  methods: {
    doSomething() {
      // get the input
      
      
      var urlRepos = "https://api.github.com/users/" + this.username + "/repos";
      console.log(urlRepos);
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
    changeRepo(event) {
      
      this.selectedRepo = event.target.options[event.target.options.selectedIndex].text;
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
