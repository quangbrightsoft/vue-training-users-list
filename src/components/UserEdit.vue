<template>
  <div>
    <h1>Create/Edit User</h1>
    <v-snackbar :value="!!message" :multi-line="multiLine">
      {{ message }}
      <v-btn color="red" text @click="message = null">
        Close
      </v-btn>
    </v-snackbar>
    <form id="demo">
      <v-text-field
        :rules="[rules.required, rules.email]"
        v-model="currentUser.email"
        label="Email"
      ></v-text-field>
      <v-text-field
        v-model="currentUser.fullName"
        label="Full name"
      ></v-text-field>
      <v-text-field
        v-model="currentUser.personalId"
        label="Personal Id"
      ></v-text-field>
      Roles:
      <br />
      <div :key="role.name" v-for="role in roleItems">
        <input
          type="checkbox"
          :id="role.name + 'checkbox'"
          :value="role.value"
          v-model="currentUser.roles"
        />
        <label :for="role.name + 'checkbox'">{{ role.name }}</label>
      </div>
      <v-text-field
        v-if="currentUser.roles.includes('Patient')"
        v-model="currentUser.ssn"
        label="ssn"
      ></v-text-field>
      <v-text-field
        v-if="currentUser.roles.includes('Patient')"
        v-model="currentUser.medicalNote"
        label="medicalNote"
      ></v-text-field>
      <div v-if="currentUser.roles.includes('Doctor')">
        <p>Skills</p>

        <div :key="skill.name" v-for="skill in skillOptions">
          <input
            type="checkbox"
            :id="skill.name + 'checkbox'"
            :value="skill.value"
            v-model="currentUser.skills"
          />
          <label :for="skill.name + 'checkbox'">{{ skill.name }}</label>
        </div>
        <v-menu
          ref="menu"
          v-model="startTimePicker"
          :close-on-content-click="false"
          :nudge-right="40"
        >
          <template v-slot:activator="{ on }">
            <v-text-field
              v-model="currentUser.workTime.startTime"
              label="start time"
              readonly
              v-on="on"
            ></v-text-field>
          </template>
          <v-time-picker
            v-if="startTimePicker"
            v-model="currentUser.workTime.startTime"
            full-width
          ></v-time-picker>
        </v-menu>
         <v-menu
          ref="menu"
          v-model="endTimePicker"
          :close-on-content-click="false"
          :nudge-right="40"
        >
          <template v-slot:activator="{ on }">
            <v-text-field
              v-model="currentUser.workTime.endTime"
              label="start time"
              readonly
              v-on="on"
            ></v-text-field>
          </template>
          <v-time-picker
            v-if="endTimePicker"
            v-model="currentUser.workTime.endTime"
            full-width
          ></v-time-picker>
        </v-menu>
      </div>

      <v-btn color="primary" @click="save()">Save</v-btn>
      <v-btn color="grey" @click="back()">Cancel</v-btn>
    </form>
  </div>
</template>

<script>
import Vue from "vue";
import axios from "axios";
import router from "../router";
export default Vue.extend({
  name: "UserEdit",

  data() {
    return {
      startTimePicker: false,
      endTimePicker: false,
      message: "",
      multiLine: true,
      isLoading: false,
      userList: { items: [] },
      singleSelect: false,
      options: {},
      roleItems: [
        { name: "Admin", value: "Admin", checked: false },
        { name: "Power user", value: "PowerUser", checked: false },
        { name: "Patient", value: "Patient", checked: false },
        { name: "Doctor", value: "Doctor", checked: false },
        { name: "BMA", value: "BmaSkill", checked: false },
      ],
      skillOptions: [
        { name: "EB", value: "EB", checked: false },
        { name: "ES", value: "ES", checked: false },
        { name: "BP", value: "BP", checked: false },
        { name: "HO", value: "HO", checked: false },
        { name: "SP", value: "SP", checked: false },
      ],
      currentUser: {
        createdAt: null,
        deletedAt: null,
        email: "",
        fullName: "",
        id: 0,
        isDeleted: false,
        isDisabled: false,
        leaves: [],
        medicalNote: "",
        personalId: "",
        roles: [],
        sex: "",
        skillIds: [],
        skills: [],
        ssn: "",
        updatedAt: null,
        userName: null,
        workTime: {
          startTime: "08:00:00",
          endTime: "08:00:00",
          weekDays: [1, 2, 3, 4, 5],
        },
      },
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
      rules: {
        required: (value) => !!value || "Required.",
        counter: (value) => value.length <= 20 || "Max 20 characters",
        email: (value) => {
          const pattern = /^(([^<>()[\]\\.,;:\s@"]+(\.[^<>()[\]\\.,;:\s@"]+)*)|(".+"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,}))$/;
          return pattern.test(value) || "Invalid e-mail.";
        },
      },
    };
  },

  mounted() {
    this.getCurrentUser();
  },
  methods: {
    back() {
      router.go(-1);
    },
    getCurrentUser() {
      let params = {
        id: this.$route.params.id,
      };
      if (!parseInt(params.id)) return;
      const headers = {
        "Content-Type": "application/json",
        Authorization:
          "Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJqdGkiOiIyNGE1NzY5Mi00MDA5LTQ2NDYtODRkOS0yZDMzMWI4MTgwYmUiLCJzdWIiOiJkZXZlbG9wZXIuYnJpZ2h0c29mdEBnbWFpbC5jb20iLCJodHRwOi8vc2NoZW1hcy5taWNyb3NvZnQuY29tL3dzLzIwMDgvMDYvaWRlbnRpdHkvY2xhaW1zL3JvbGUiOiJBZG1pbiIsImV4cCI6MTYyMTYwNTQ2MSwiaXNzIjoiQnJpZ2h0c29mdC5FSGVhcnRCb29raW5nIiwiYXVkIjoiQnJpZ2h0c29mdC5FSGVhcnRCb29raW5nIn0.9ix0zQ2lks9zcHvXHHgJGFtNz91FCO7ZXFBzZtr30Fw",
      };
      let userUrl = `https://localhost:5001/api/user/get`;
      axios
        .get(userUrl, {
          headers: headers,
          params: params,
        })
        .then((response) => {
          this.currentUser = response.data;
          if (!this.currentUser.skills) {
            this.currentUser.skills = [];
          }
        });
    },
    save() {
      let params = {
        id: this.$route.params.id,
      };
      if (!parseInt(params.id)) {
        this.createUser();
      } else {
        this.editUser();
      }
    },
    createUser() {
      const headers = {
        "Content-Type": "application/json",
        Authorization:
          "Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJqdGkiOiIyNGE1NzY5Mi00MDA5LTQ2NDYtODRkOS0yZDMzMWI4MTgwYmUiLCJzdWIiOiJkZXZlbG9wZXIuYnJpZ2h0c29mdEBnbWFpbC5jb20iLCJodHRwOi8vc2NoZW1hcy5taWNyb3NvZnQuY29tL3dzLzIwMDgvMDYvaWRlbnRpdHkvY2xhaW1zL3JvbGUiOiJBZG1pbiIsImV4cCI6MTYyMTYwNTQ2MSwiaXNzIjoiQnJpZ2h0c29mdC5FSGVhcnRCb29raW5nIiwiYXVkIjoiQnJpZ2h0c29mdC5FSGVhcnRCb29raW5nIn0.9ix0zQ2lks9zcHvXHHgJGFtNz91FCO7ZXFBzZtr30Fw",
      };
      let userCreateUrl = `https://localhost:5001/api/user/create`;
      axios
        .post(userCreateUrl, this.currentUser, {
          headers: headers,
        })
        .then(() => {
          this.back();
        })
        .catch((err) => {
          this.message = err.response.data;
        });
    },
    editUser() {
      const headers = {
        "Content-Type": "application/json",
        Authorization:
          "Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJqdGkiOiIyNGE1NzY5Mi00MDA5LTQ2NDYtODRkOS0yZDMzMWI4MTgwYmUiLCJzdWIiOiJkZXZlbG9wZXIuYnJpZ2h0c29mdEBnbWFpbC5jb20iLCJodHRwOi8vc2NoZW1hcy5taWNyb3NvZnQuY29tL3dzLzIwMDgvMDYvaWRlbnRpdHkvY2xhaW1zL3JvbGUiOiJBZG1pbiIsImV4cCI6MTYyMTYwNTQ2MSwiaXNzIjoiQnJpZ2h0c29mdC5FSGVhcnRCb29raW5nIiwiYXVkIjoiQnJpZ2h0c29mdC5FSGVhcnRCb29raW5nIn0.9ix0zQ2lks9zcHvXHHgJGFtNz91FCO7ZXFBzZtr30Fw",
      };
      let userUpdateUrl = `https://localhost:5001/api/user/update`;
      axios
        .put(userUpdateUrl, this.currentUser, {
          headers: headers,
          params: { id: this.currentUser.id.toString() },
        })
        .then(() => {
          this.back();
        })
        .catch((err) => {
          this.message = err.response.data;
        });
    },
  },
});
</script>
