<script setup>
import { ref, onMounted } from "vue"
import draggable from "vuedraggable"
import { getTasks, updateTask } from "./services/api"

const todo = ref([])
const inprogress = ref([])
const done = ref([])

const newTask = ref("")

onMounted(async () => {
  const res = await getTasks()
  const tasks = res.data

  todo.value = tasks.filter(t => t.status === "todo")
  inprogress.value = tasks.filter(t => t.status === "inprogress")
  done.value = tasks.filter(t => t.status === "done")
})

const onDragEnd = async (event) => {
  const task = event.item.__draggable_context.element

  if (event.to.parentElement.innerText.includes("To Do")) {
    await updateTask(task._id, "todo")
  }

  if (event.to.parentElement.innerText.includes("In Progress")) {
    await updateTask(task._id, "inprogress")
  }

  if (event.to.parentElement.innerText.includes("Done")) {
    await updateTask(task._id, "done")
  }
}

const addTask = async () => {
  if (!newTask.value) return

  const res = await fetch("http://localhost:5000/api/tasks", {
    method: "POST",
    headers: {
      "Content-Type": "application/json"
    },
    body: JSON.stringify({
      title: newTask.value,
      description: "",
      status: "todo"
    })
  })

  const task = await res.json()
  todo.value.push(task)

  newTask.value = ""
}
</script>

<template>

<div class="addTask">
  <input v-model="newTask" placeholder="New task..." />
  <button @click="addTask">Add Task</button>
</div>

  <div class="board">

    <!-- TODO -->
    <div class="column">
      <h2>To Do ({{ todo.length }})</h2>

      <draggable
        v-model="todo"
        group="tasks"
        item-key="_id"
        @end="onDragEnd"
      >
        <template #item="{ element }">
          <div class="task">
            {{ element.title }}
          </div>
        </template>
      </draggable>

    </div>

    <!-- IN PROGRESS -->
    <div class="column">
      <h2>In Progress ({{ inprogress.length }})</h2>

      <draggable
        v-model="inprogress"
        group="tasks"
        item-key="_id"
        @end="onDragEnd"
      >
        <template #item="{ element }">
          <div class="task">
            {{ element.title }}
          </div>
        </template>
      </draggable>

    </div>

    <!-- DONE -->
    <div class="column">
      <h2>Done ({{ done.length }})</h2>

      <draggable
        v-model="done"
        group="tasks"
        item-key="_id"
        @end="onDragEnd"
      >
        <template #item="{ element }">
          <div class="task">
            {{ element.title }}
          </div>
        </template>
      </draggable>

    </div>

  </div>
</template>

<style>
body{
  margin:0;
  font-family:Arial;
  background:#f4f5f7;
}

.addTask{
  padding:20px;
  display:flex;
  gap:10px;
}

.addTask input{
  padding:8px;
  width:200px;
}

.addTask button{
  padding:8px 12px;
  cursor:pointer;
}

.board{
  display:flex;
  gap:20px;
  padding:20px;
}

.column{
  background:#ebecf0;
  padding:15px;
  width:300px;
  border-radius:8px;
  min-height:300px;
}

.task{
  background:white;
  padding:12px;
  margin-top:10px;
  border-radius:6px;
  box-shadow:0 2px 4px rgba(0,0,0,0.1);
  cursor:grab;
}
</style>