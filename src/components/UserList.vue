<template>
  <v-data-table
    v-model="selected"
    :headers="headers"
    :items="userList.items"
    :single-select="singleSelect"
    :server-items-length="userList.totalCount"
    :options.sync="options"
    :footer-props="{itemsPerPageOptions:[10,30,50,100]}"
    item-key="name"
    show-select
    class="elevation-1"
  >
    <template v-slot:top>
      <v-row class="pa-md-4 mx-lg-auto">
        <v-btn>Create new +</v-btn>
        <v-col cols="12" sm="6" md="3">
          <v-text-field class="float-right" label="Search by full name"></v-text-field>
        </v-col>
        <v-col cols="12" sm="6" md="3">
          <v-select v-model="value" :items="roleItems" attach chips label="Chips" multiple></v-select>
        </v-col>
      </v-row>
    </template>
  </v-data-table>
</template>

<script>
import Vue from "vue";
import axios from "axios";
export default Vue.extend({
  name: "UserList",

  data() {
    return {
      isLoading: false,
      userList: { items: [] },
      singleSelect: false,
      selected: [],
      options: {},
      roleItems: ["foo", "bar", "fizz", "buzz"],
      headers: [
        {
          text: "Email",
          align: "start",
          sortable: true,
          value: "email"
        },
        { text: "Full name", value: "fullName" },
        { text: "Roles", value: "roles" },
        { text: "Actions", value: "actions" }
      ]
    };
  },
  watch: {
    options: {
      handler() {
        console.log(this.options);
        this.getUsers();
      },
      deep: true
    }
  },
  mounted() {
    this.getUsers();
  },
  methods: {
    getUsers() {
      let userListUrl = `https://localhost:5001/api/user/list`;
      const { sortBy, sortDesc, page, itemsPerPage } = this.options;
      let params = {
        sortBy: sortBy[0],
        descending: sortDesc[0],
        page: page,
        pageSize: itemsPerPage
      };
      const headers = {
        "Content-Type": "application/json",
        Authorization:
          "Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJqdGkiOiIyNGE1NzY5Mi00MDA5LTQ2NDYtODRkOS0yZDMzMWI4MTgwYmUiLCJzdWIiOiJkZXZlbG9wZXIuYnJpZ2h0c29mdEBnbWFpbC5jb20iLCJodHRwOi8vc2NoZW1hcy5taWNyb3NvZnQuY29tL3dzLzIwMDgvMDYvaWRlbnRpdHkvY2xhaW1zL3JvbGUiOiJBZG1pbiIsImV4cCI6MTYyMTYwNTQ2MSwiaXNzIjoiQnJpZ2h0c29mdC5FSGVhcnRCb29raW5nIiwiYXVkIjoiQnJpZ2h0c29mdC5FSGVhcnRCb29raW5nIn0.9ix0zQ2lks9zcHvXHHgJGFtNz91FCO7ZXFBzZtr30Fw"
      };
      axios
        .get(userListUrl, {
          headers: headers,
          params: params
        })
        .then(response => {
          this.userList = response.data;
        });
    }
  }
});
</script>
