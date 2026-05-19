<template>
  <div class="min-h-screen bg-gray-100 flex items-center justify-center p-6">
    <div class="w-full max-w-2xl bg-white shadow-xl rounded-lg p-6">
      <h1 class="text-3xl font-bold mb-6 text-gray-800">Task Manager</h1>

      <div v-if="!token" class="space-y-4">
        <input v-model="email" placeholder="Email"
               class="w-full p-3 border rounded-lg focus:ring focus:border-blue-400"/>

        <input v-model="password" type="password" placeholder="Password"
               class="w-full p-3 border rounded-lg focus:ring focus:border-blue-400"/>

        <button @click="login"
                class="w-full bg-blue-600 hover:bg-blue-700 text-white py-3 rounded-lg">
          Login
        </button>
      </div>

      <div v-else>
        <button @click="logout"
                class="bg-red-500 hover:bg-red-600 text-white py-2 px-4 rounded mb-4">
          Logout
        </button>

        <h2 class="text-xl font-semibold mb-4">Create New Task</h2>
        <div class="space-y-4">
          <input v-model="title" placeholder="Task Title"
                 class="w-full p-3 border rounded-lg focus:ring"/>

          <textarea v-model="description" placeholder="Description"
                    class="w-full p-3 border rounded-lg focus:ring"></textarea>

          <select v-model="status"
                  class="w-full p-3 border rounded-lg focus:ring">
            <option disabled value="">Select Status</option>
            <option value="todo">Todo</option>
            <option value="progress">In Progress</option>
            <option value="done">Done</option>
          </select>

          <input v-model="due_date" type="date"
                 class="w-full p-3 border rounded-lg focus:ring"/>

          <button @click="createTask"
                  class="w-full bg-green-600 hover:bg-green-700 text-white py-3 rounded-lg">
            Add Task
          </button>
        </div>

        <h2 class="text-xl font-semibold mt-8 mb-3">Task List</h2>

        <ul class="space-y-3">
          <li v-for="t in tasks" :key="t.id"
              class="border p-4 rounded-lg bg-gray-50 flex justify-between items-center">
            <div>
              <div class="font-bold text-lg">{{ t.title }}</div>
              <div class="text-sm text-gray-600">{{ t.description }}</div>
              <div class="text-xs text-blue-600">Status: {{ t.status }}</div>
              <div class="text-xs text-gray-500">Due: {{ t.due_date }}</div>
            </div>

            <div class="flex space-x-3">
              <button @click="openEdit(t)"
                      class="text-blue-500 hover:underline">
                Edit
              </button>

              <button @click="deleteTask(t.id)"
                      class="text-red-500 hover:underline">
                Delete
              </button>
            </div>
          </li>
        </ul>
      </div>

      <div v-if="showEdit" class="fixed inset-0 bg-black bg-opacity-40 flex items-center justify-center">
        <div class="bg-white p-6 rounded-lg w-full max-w-md">

          <h2 class="text-xl font-bold mb-4">Edit Task</h2>

          <input v-model="editForm.title" placeholder="Title"
                 class="w-full p-3 border rounded-lg mb-3"/>

          <textarea v-model="editForm.description"
                    class="w-full p-3 border rounded-lg mb-3"></textarea>

          <select v-model="editForm.status"
                  class="w-full p-3 border rounded-lg mb-3">
            <option value="todo">Todo</option>
            <option value="progress">Progress</option>
            <option value="done">Done</option>
          </select>

          <input v-model="editForm.due_date" type="date"
                 class="w-full p-3 border rounded-lg mb-4"/>

          <div class="flex justify-end space-x-4">
            <button @click="showEdit = false"
                    class="px-4 py-2 bg-gray-300 rounded">
              Cancel
            </button>

            <button @click="updateTask"
                    class="px-4 py-2 bg-blue-600 text-white rounded">
              Update
            </button>
          </div>

        </div>
      </div>

    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      email: "",
      password: "",
      token: localStorage.getItem("token"),

      title: "",
      description: "",
      status: "",
      due_date: "",

      tasks: [],

      showEdit: false,
      editForm: {
        id: null,
        title: "",
        description: "",
        status: "",
        due_date: ""
      }
    };
  },

  methods: {
    async login() {
      const res = await fetch("http://localhost:8080/login", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ email: this.email, password: this.password }),
      });

      const data = await res.json();
      this.token = data.token;
      localStorage.setItem("token", data.token);

      this.getTasks();
    },

    logout() {
      this.token = null;
      localStorage.removeItem("token");
    },

    async getTasks() {
      const res = await fetch("http://localhost:8080/tasks/", {
        headers: {
          Authorization: "Bearer " + this.token,
        },
      });
      this.tasks = await res.json();
    },

    async createTask() {
      await fetch("http://localhost:8080/tasks/", {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
          Authorization: "Bearer " + this.token,
        },
        body: JSON.stringify({
          title: this.title,
          description: this.description,
          status: this.status,
          due_date: this.due_date,
        }),
      });

      this.getTasks();
    },

    openEdit(task) {
      this.showEdit = true;
      this.editForm = { ...task }; 
    },

    // UPDATE TASK (PUT /tasks/:id)
    async updateTask() {
      await fetch(`http://localhost:8080/tasks/${this.editForm.id}`, {
        method: "PUT",
        headers: {
          "Content-Type": "application/json",
          Authorization: "Bearer " + this.token,
        },
        body: JSON.stringify(this.editForm),
      });

      this.showEdit = false;
      this.getTasks();
    },

    async deleteTask(id) {
      await fetch(`http://localhost:8080/tasks/${id}`, {
        method: "DELETE",
        headers: { Authorization: "Bearer " + this.token },
      });

      this.getTasks();
    }
  },

  mounted() {
    if (this.token) this.getTasks();
  },
};
</script>