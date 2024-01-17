
<template>
  <div id="app">
    <h1 style="margin-left: 50px;" v-if="users.length">List of users</h1>
<h1 v-else>User list is empty</h1>
    <div class="search">
      <input
        type="text"
        v-model="search"
        placeholder="Search..."
      />
    </div>
    <div class="users">
      <ul>
        <li v-for="user in filteredUsers" :key="user.id">
          <img :src="user.avatar" alt="Аватар пользователя" />
          <span class="name" @click="showDetails(user)"
            >{{ user.first_name }} {{ user.last_name }}</span
          >
          <span class="email">{{ user.email }}</span>
          <button class="delete" @click="deleteUser(user.id)">Delete</button>
        </li>
      </ul>
    </div>
    <div class="form">
      <h2>Add a new user</h2>
      <form @submit.prevent="addUser">
        <div class="input-group">
          <label for="name">Name</label>
          <input type="text" id="name" v-model="name" />
          <span class="error" v-if="errors.name">{{ errors.name }}</span>
        </div>
        <div class="input-group">
          <label for="email">Email</label>
          <input type="email" id="email" v-model="email" />
          <span class="error" v-if="errors.email">{{ errors.email }}</span>
        </div>
        <button type="submit">Add</button>
      </form>
    </div>
    <div class="modal" v-if="selectedUser" @click="closeDetails">
  <div class="modal-content" @click.stop>
    <div class="modal-header">
      <span class="close" @click="closeDetails">×</span>
        <h3 style="margin: 0;">User details</h3>
        </div>
        <img :src="selectedUser.avatar" alt="Аватар пользователя" />
        <p>
          <strong>Name:</strong>
          <input type="text" v-model="selectedUser.first_name" />
        </p>
        <p>
          <strong>Surname:</strong>
          <input type="text" v-model="selectedUser.last_name" />
        </p>
        <p>
          <strong>Email:</strong>
          <input type="email" v-model="selectedUser.email" />
        </p>
        <p>
          <strong>Number:</strong>
          <input type="tel" v-model="selectedUser.phone" />
        </p>
        <p>
          <strong>Address:</strong>
          <input type="text" v-model="selectedUser.address" />
        </p>
        <button @click="updateUser">Save changes</button>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";

export default {
  name: "App",
  data() {
    return {
      users: [],
      search: "",
      name: "",
      email: "",
      errors: {},
      selectedUser: null,
    };
  },
  computed: {
    filteredUsers() {
      return this.users.filter((user) => {
        return (
          user.first_name.toLowerCase().includes(this.search.toLowerCase()) ||
          user.last_name.toLowerCase().includes(this.search.toLowerCase()) ||
          user.email.toLowerCase().includes(this.search.toLowerCase())
        );
      });
    },
  },
  methods: {
    async fetchUsers() {
      try {
        const response = await axios.get("https://reqres.in/api/users");
        this.users = response.data.data;
      } catch (error) {
        console.error(error);
      }
    },
    async addUser() {
      this.errors = {};
      if (!this.name) {
        this.errors.name = "Name field cannot be empty";
      }
      if (!this.email) {
        this.errors.email = "Email field cannot be empty";
      } else if (!this.validateEmail(this.email)) {
        this.errors.email =
          "Email field must be a valid email address";
      }
      if (Object.keys(this.errors).length > 0) {
        return;
      }
      const newUser = {
        first_name: this.name.split(" ")[0],
        last_name: this.name.split(" ")[1] || "",
        email: this.email,
      };
      try {
        const response = await axios.post(
          "https://reqres.in/api/users",
          newUser
        );
        newUser.id = response.data.id;
        this.users.unshift(newUser);
        this.name = "";
        this.email = "";
      } catch (error) {
        console.error(error);
      }
    },
    async deleteUser(id) {
      try {
        await axios.delete(`https://reqres.in/api/users/${id}, updatedUser`);
        const index = this.users.findIndex((user) => user.id === id);
        this.users.splice(index, 1);
      } catch (error) {
        console.error(error);
      }
    },
    showDetails(user) {
      this.selectedUser = user;
    },
    closeDetails() {
      this.selectedUser = null;
    },
    async updateUser() {
      const updatedUser = {
        first_name: this.selectedUser.first_name,
        last_name: this.selectedUser.last_name,
        email: this.selectedUser.email,
        phone: this.selectedUser.phone,
        address: this.selectedUser.address,
        avatar: this.selectedUser.avatar,
      };
      try {
        await axios.put(
          `https://reqres.in/api/users/${this.selectedUser.id}`,
          updatedUser
        );
        const index = this.users.findIndex(
          (user) => user.id === this.selectedUser.id
        );
        this.users.splice(index, 1, updatedUser);
        this.closeDetails();
      } catch (error) {
        console.error(error);
      }
    },

    validateEmail(email) {
      const regex = /^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$/;
      return regex.test(email);
    },
  },
  mounted() {
    this.fetchUsers();
  },
};
</script>

<style>
.search {
  display: flex;
  justify-content: center;
  margin: 20px 0;
}

.search input {
  width: 100%;
  max-width: 600px;
  padding: 10px 20px;
  border: 1px solid #ccc;
  border-radius: 5px;
  font-size: 16px;
  transition: border-color 0.3s ease;
}

.search input:focus {
  border-color: #007BFF;
  outline: none;
}


.users {
  display: flex;
  flex-wrap: wrap;
  list-style: none;
  padding: 0;
}

.users li {
  display: flex;
  align-items: center;
  margin: 10px;
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 5px;
}

.users img {
  width: 50px;
  height: 50px;
  border-radius: 50%;
  margin-right: 10px;
}

.users .name {
  font-weight: bold;
  cursor: pointer;
}
.users .name:hover {
color: #005b9e;
}

.users .email {
  margin-left: auto;
  margin-right: 10px;
}

.users .delete {
  background-color: #f44336;
  color: white;
  border: none;
  border-radius: 5px;
  padding: 5px 10px;
  cursor: pointer;
  
}
.users .delete:hover {
  background-color: #9b2020;
}
.form {
  margin: 10px;
}

.form .input-group {
  margin: 10px;
}

.form label {
  display: block;
  font-weight: bold;
}

.form input {
  display: block;
  width: 300px;
  padding: 5px;
  border: 1px solid #ccc;
  border-radius: 5px;
}

.form button {
  display: block;
  margin: 10px;
  padding: 10px;
  background-color: #007bd2;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

.form .error {
  color: red;
  font-style: italic;
}

.modal {
  display: flex;
  align-items: center;
  justify-content: center;
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
}

.modal-content {
  width: 400px;
  padding: 20px;
  background-color: white;
  border-radius: 10px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
}

.modal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-direction: row-reverse;
}

.close {
  font-size: 20px;
  font-weight: bold;
  cursor: pointer;
  color: #333;
  transition: color 0.3s ease-in-out;
}

.close:hover {
  color: #ff0000; /* Измените на желаемый цвет при наведении */
}

h3 {
  font-size: 18px;
  margin-left: 10px;
}

img {
  width: 50%;
  max-height: 200px;
  object-fit: cover;
  border-radius: 5px;
  margin-top: 10px;
}

p {
  margin-top: 10px;
}

strong {
  font-weight: bold;
}

input {
  width: 100%;
  padding: 8px;
  margin-top: 5px;
  border: 1px solid #ddd;
  border-radius: 5px;
  box-sizing: border-box;
}

button {
  background-color: #007bd2;
  color: white;
  padding: 10px 15px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  font-size: 16px;
  margin-top: 10px;
  transition: background-color 0.2s ease-in-out;
}

button:hover {
  background-color: #005b9e; /* Измените на желаемый цвет при наведении */
}
</style>
