<template>
<q-card-section>
  <q-card class="q-pa-md" style="max-width: 600px; margin: auto;">
  <div class="text-h6 text-center">Task Manager</div>

  <div class="row items-center q-gutter-sm">
    <q-input
      v-model="newTask"
      label="New Task"
      outlined
      @keyup.enter="addTask"
      class="q-flex"
      style="flex: 2"
    />
    <q-btn
      label="Add Task"
      color="primary"
      @click="addTask"
    />
  </div>

  <br>
        <q-separator />
  </br>
      <q-card-section>
        <div v-if="loading" class="text-center q-mt-md">Loading tasks...</div>
        <q-list v-else>
          <q-item v-for="task in tasks" :key="task.id" clickable>
            <q-item-section>
              <div v-if="editId !== task.id">{{ task.title }}</div>
              <q-input
                v-else
                v-model="editTitle"
                dense
                outlined
                @keyup.enter="updateTask(task.id)"
              />
            </q-item-section>
            <q-item-section side>
              <q-btn
                dense
                flat
                icon="edit"
                color="orange"
                @click="startEdit(task)"
                v-if="editId !== task.id"
              />
              <q-btn
                dense
                flat
                icon="check"
                color="green"
                @click="updateTask(task.id)"
                v-if="editId === task.id"
              />
              <q-btn dense flat icon="delete" color="red" @click="deleteTask(task.id)" />
            </q-item-section>
          </q-item>
        </q-list>
      </q-card-section>
</q-card>
</q-card-section>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      tasks: [],
      newTask: "",
      editId: null,
      editTitle: "",
      loading: false,
    };
  },
  methods: {
    // Get
    async fetchTasks() {
      this.loading = true;
      try {
        const res = await axios.get("https://jsonplaceholder.typicode.com/todos?_limit=10");
        this.tasks = res.data;
      } catch (error) {
        console.error("Error fetching tasks:", error);
      } finally {
        this.loading = false;
      }
    },

    // Add
    async addTask() {
      if (!this.newTask.trim()) return;
      try {
        const res = await axios.post("https://jsonplaceholder.typicode.com/todos", {
          title: this.newTask,
          completed: false,
        });
        this.tasks.unshift(res.data);
        this.newTask = "";
      } catch (error) {
        console.error("Error adding task:", error);
      }
    },

    // Edit
    startEdit(task) {
      this.editId = task.id;
      this.editTitle = task.title;
    },

    // Update task
    async updateTask(id) {
      if (!this.editTitle.trim()) return;
      try {
        const res = await axios.put(`https://jsonplaceholder.typicode.com/todos/${id}`, {
          title: this.editTitle,
        });
        const index = this.tasks.findIndex((t) => t.id === id);
        this.tasks[index] = res.data;
        this.editId = null;
        this.editTitle = "";
      } catch (error) {
        console.error("Error updating task:", error);
      }
    },

    // Delete task
    async deleteTask(id) {
      try {
        await axios.delete(`https://jsonplaceholder.typicode.com/todos/${id}`);
        this.tasks = this.tasks.filter((t) => t.id !== id);
      } catch (error) {
        console.error("Error deleting task:", error);
      }
    },
  },
  mounted() {
    this.fetchTasks();
  },
};
</script>

<style>
.q-page {
  display: flex;
  justify-content: center;
  align-items: flex-start;
  min-height: 100vh;
  padding-top: 50px;
}
</style>
