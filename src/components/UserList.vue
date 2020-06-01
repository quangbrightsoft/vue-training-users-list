<template>
  <v-data-table
    :loading="isLoading"
    v-model="selected"
    :headers="headers"
    :items="userList.items"
    :single-select="singleSelect"
    :server-items-length="userList.totalCount"
    :options.sync="options"
    :footer-props="{ itemsPerPageOptions: [10, 30, 50, 100] }"
    item-key="name"
    show-select
    class="elevation-1"
  >
    <template v-slot:item.email="{ item }">
      <router-link :to="'/user/edit/' + item.id">{{ item.email }}</router-link>
    </template>
    <template v-slot:item.actions="{ item }">{{ item.id }}</template>
    <template v-slot:top>
      <v-row class="pa-md-4 mx-lg-auto">
        <v-btn href="#/user/edit/0">Create new +</v-btn>
        <v-col cols="12" sm="6" md="3">
          <v-text-field
            class="float-right"
            v-model="fullNameFilter"
            label="Search by full name"
            @change="getUsers()"
          ></v-text-field>
        </v-col>
        <v-col cols="12" sm="6" md="3">
          <v-select
            v-model="selectedRoles"
            :items="roleItems"
            attach
            chips
            label="Filter"
            multiple
            @change="getUsers()"
          ></v-select>
        </v-col>
      </v-row>
    </template>
  </v-data-table>
</template>

<script lang="ts">
import Vue from "vue";
import axios from "axios";
import qs from "qs";
export default Vue.extend({
  name: "UserList",

  data() {
    return {
      isLoading: false,
      userList: { items: [] },
      singleSelect: false,
      selected: [],
      fullNameFilter: "",
      selectedRoles: [],
      options: { sortBy: [], descending: [] },
      roleItems: ["Admin", "PowerUser", "Patient", "Doctor", "BmaSkill"],
      headers: [
        {
          text: "Email",
          align: "start",
          sortable: true,
          value: "email",
        },
        { text: "Full name", value: "fullName" },
        { text: "Roles", value: "roles" },
        { text: "Actions", value: "actions" },
      ],
    };
  },
  watch: {
    options: {
      handler() {
        this.getUsers();
      },
      deep: true,
    },
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
        pageSize: itemsPerPage,
        roles: this.selectedRoles,
        search: this.fullNameFilter,
      };
      const headers = {
        "Content-Type": "application/json",
        Authorization:
          "Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJqdGkiOiIyNGE1NzY5Mi00MDA5LTQ2NDYtODRkOS0yZDMzMWI4MTgwYmUiLCJzdWIiOiJkZXZlbG9wZXIuYnJpZ2h0c29mdEBnbWFpbC5jb20iLCJodHRwOi8vc2NoZW1hcy5taWNyb3NvZnQuY29tL3dzLzIwMDgvMDYvaWRlbnRpdHkvY2xhaW1zL3JvbGUiOiJBZG1pbiIsImV4cCI6MTYyMTYwNTQ2MSwiaXNzIjoiQnJpZ2h0c29mdC5FSGVhcnRCb29raW5nIiwiYXVkIjoiQnJpZ2h0c29mdC5FSGVhcnRCb29raW5nIn0.9ix0zQ2lks9zcHvXHHgJGFtNz91FCO7ZXFBzZtr30Fw",
      };
      this.isLoading = true;
      axios
        .get(userListUrl, {
          headers: headers,
          params: params,
          paramsSerializer: (params) => {
            return qs.stringify(params);
          },
        })
        .then((response) => {
          this.userList = response.data;
          this.isLoading = false;
        });
    },
  },
});
</script>
