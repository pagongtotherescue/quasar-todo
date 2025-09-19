<template>
  <q-card-section>
    <q-card class="q-pa-md" style="max-width: 600px; margin: auto;">
      <div class="text-h6 text-center">Task Manager</div>

      <!-- Add new task -->
      <div class="row items-center q-gutter-sm">
        <q-input
          v-model="newTask"
          label="New Task"
          outlined
          @keyup.enter="addTask"
          class="q-flex"
          style="flex: 2"
        />
        <q-btn label="Add Task" color="primary" @click="addTask" />
      </div>

      <br />
      <q-separator />
      <br />

      <!-- Task list -->
      <q-card-section>
        <div v-if="loading" class="text-center q-mt-md">Loading tasks...</div>
        <q-list v-else>
          <q-item v-for="task in tasks" :key="task.id" clickable>
            <q-item-section>
              <!-- Normal view -->
              <div v-if="!task.editing">{{ task.title }}</div>

              <!-- Edit mode -->
              <q-input
                v-else
                v-model="task.tempTitle"
                dense
                outlined
                @keyup.enter="updateTask(task)"
              />
            </q-item-section>

            <q-item-section side>
              <!-- Edit button -->
              <q-btn
                dense
                flat
                icon="edit"
                color="orange"
                @click="startEdit(task)"
                v-if="!task.editing"
              />

              <!-- Save button -->
              <q-btn
                dense
                flat
                icon="check"
                color="green"
                @click="updateTask(task)"
                v-if="task.editing"
              />

              <!-- Delete button -->
              <q-btn
                dense
                flat
                icon="delete"
                color="red"
                @click="deleteTask(task)"
              />
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
      loading: false,
      localId: 1000, // counter for locally created tasks
    };
  },
  methods: {
    // Get
    async fetchTasks() {
      this.loading = true;
      try {
        const res = await axios.get(
          "https://jsonplaceholder.typicode.com/todos?_limit=10"
        );
        // Add edit fields to each task
        this.tasks = res.data.map((t) => ({
          ...t,
          editing: false,
          tempTitle: t.title,
        }));
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
        const res = await axios.post(
          "https://jsonplaceholder.typicode.com/todos",
          {
            title: this.newTask,
            completed: false,
          }
        );
        // Add to local list with a unique local ID
        const newTask = {
          ...res.data,
          id: this.localId++, // unique local ID
          editing: false,
          tempTitle: res.data.title,
        };
        this.tasks.unshift(newTask);
        this.newTask = "";
      } catch (error) {
        console.error("Error adding task:", error);
      }
    },

    // Edit mode
    startEdit(task) {
      task.editing = true;
      task.tempTitle = task.title;
    },

    // Update task
    async updateTask(task) {
      if (!task.tempTitle.trim()) return;
      try {
        if (task.id <= 200) {
          // API task → update via server
          await axios.put(
            `https://jsonplaceholder.typicode.com/todos/${task.id}`,
            {
              ...task,
              title: task.tempTitle,
            }
          );
        }
        // Always update locally
        task.title = task.tempTitle;
        task.editing = false;
      } catch (error) {
        console.error("Error updating task:", error);
      }
    },

    // Delete task
    async deleteTask(task) {
      try {
        if (task.id <= 200) {
          // API task → delete via server
          await axios.delete(
            `https://jsonplaceholder.typicode.com/todos/${task.id}`
          );
        }
        // Always update locally
        this.tasks = this.tasks.filter((t) => t.id !== task.id);
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
