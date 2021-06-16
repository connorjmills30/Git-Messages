<template>
  <div>
    <table style="width:90%">
        <tr>
            <th>Day</th>
            <!--<th>Name</th>-->
            <th>Message</th>
        </tr>
        <tr v-for="(entry) in formattedData" :key="entry.day">
            <td class="day-column"> {{ entry.day }} </td>
            <!--<td> {{ entry.author }} </td>-->
            <td class="message-column"> {{ entry.message }} </td>
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
        data: Array,
        user: String
    },
    computed: {
        formattedData: function() {
            let tableData = [];
            
            // cannot change component prop directly; reverse a copy instead
            let dataCopy = this.data;   
            if (dataCopy) dataCopy.reverse();
            
            dataCopy.forEach(commit => {
            //this.data.forEach(commit => {
                let author_name = commit.author.login;
                if (author_name == this.user) { 

                    let commit_date = commit.commit.author.date;   // ISO: "2021-06-15T20:23:56Z"
                    let commit_day = new Date(commit_date).getDate();
                    let commit_message = commit.commit.message;

                    const found = tableData.find(entry => entry.day == commit_day);
                    if (found) {
                        found.message = found.message + "; " + commit_message;
                    } else {
                        let entry = {
                            day: commit_day,
                            message: commit_message
                        };
                        tableData.push(entry);
                    }
                }
            })
            
            tableData.sort((firstEntry, secondEntry) => firstEntry.day - secondEntry.day);
            return tableData
      }
    }
};
</script>

<style scoped>

table {
    border: 3px solid black;
    width: 90%;
    border-collapse: collapse;
}

th {
    height: 30px;
}
tr, tr, td {
    border: 1px solid grey;
}

.day-column{
    width: 50px;
}
.message-column{
    padding: 8px;
}

tr:nth-child(even) {
    background-color: lightgrey;
}
tr:nth-child(odd) {
    background-color: lightblue;
}

</style>