<template>
  <div>
    <h1>Messages Basic Table</h1>
    
    <table style="width:100%">
        <tr>
            <th>Date</th>
            <th>Name</th>
            <th>Message</th>
        </tr>
        <tr v-for="entry in cleanData" :key="entry.sha">
            <td> {{ entry.commit.author.date }} </td>
            <td> {{ entry.commit.author.name }} </td>
            <td> {{ entry.commit.message }} </td>
        </tr>
    </table>
    <!---
    <br><br>

    <h2>V Data Table</h2>
    <v-data-table 
        :headers="headers"
        :items="data"
        :search="search">
    </v-data-table>

    <br><br>

    <h3>Data Table</h3>
    <data-table :data="data"/>

    --->
  </div>
</template>

<script>

export default {
  props: {
      /*headers: [
          {text: 'Date', value: 'commit.committer.date'},
          {text: 'Committer', value: 'commit.committer.name'},
          {text: 'Message', value: 'commit.message'}
      ],*/
      data: Array
  },
  computed: {
      cleanData: function() {
          let tableData = this.data
          return tableData
      }
  }
};
</script>

<style scoped>



</style>