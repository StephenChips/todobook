<template>
  <div class="background">
    <ElRow class="container">
      <ElCol :span="8" class="container-column todolist-column">
        <h1 class="title">TODOs</h1>

        <div v-if="todoLists.length == 0" shadow="never" class="column-panel empty">
          <div style="margin-bottom: 1em;">Oops, Haven't added a TODO list yet...</div>
          <ElButton
            type="primary"
            style="margin-bottom: 1em;"
            @click.stop="addTodoList"
            title="Add a todo list"
          >Add a TODO list</ElButton>
        </div>

        <div v-else shadow="never" class="column-panel">
          <ul class="list" ref="$todoList">
            <li
              v-for="todo of todoLists"
              :key="todo.id"
              :class="isTodoSelected(todo) ? 'selected' : ''"
              @click="openTodoList(todo)"
            >
              <span style="margin-right: auto;">{{ todo.title }}</span>
              <ElBadge
                :value="countUndoneTask(todo)"
                type="primary"
                :hidden="todo.tasks.length == 0"
              />
            </li>
            <ElDivider content-position="left" style="width: unset; margin: 1em;">That's all</ElDivider>
          </ul>
          <ElButton
            class="btn-add-list"
            type="primary"
            @click.stop="addTodoList"
            title="Add a TODO list"
          >
            <span style="margin-left: 0.5em;">Add a TODO list</span>
          </ElButton>
        </div>
      </ElCol>
      <div :span="8" class="container-column tasklist-column gutter">
        <h1 class="title">Tasks</h1>

        <div
          v-if="todoLists.length == 0"
          shadow="never"
          class="column-panel empty"
        >Please add a TODO list first.</div>

        <div
          v-else-if="!currentTodoList"
          shadow="never"
          class="column-panel empty"
        >Please add a TODO list first.</div>

        <div v-else shadow="never" class="column-panel">
          <div class="header">
            <input
              v-model="currentTodoList.title"
              class="input-title"
              placeholder="Name of this Todo"
            />
            <ElButton
              size="large"
              circle
              :icon="IconDelete"
              @click.stop="confirmToDeleteTodoList(currentTodoList)"
              title="Delete this TODO list"
            ></ElButton>
          </div>
          <div
            v-if="currentTodoList.tasks.length == 0"
            class="no-task"
            style="
          display: flex;
          flex-direction: column;
          justify-content: center;
          align-items: center;
          flex: 1;"
          >
            <div
              style="margin-bottom: 1em; color: var(--el-text-color-secondary); font-style: italic;"
            >Oops, this TODO list is empty...</div>
            <ElButton
              type="primary"
              style="margin-bottom: 1em;"
              @click.stop="addTaskToCurrentTodoList"
              title="Add a task"
            >Add a task</ElButton>
          </div>
          <template v-else>
            <ul class="list" ref="$taskList">
              <li
                v-for="task of currentTodoList.tasks"
                :key="task.id"
                @click="currentTask = task"
                :class="{ 'selected': currentTask == task }"
              >
                <ElCheckbox style="margin-right: 1em;" v-model="task.hasDone"></ElCheckbox>
                <span style="margin-right: auto;">{{ task.title }}</span>
                <ElButton
                  size="small"
                  :icon="IconClose"
                  circle
                  @click.stop="confirmToDeleteTask(task)"
                  title="Delete this task"
                ></ElButton>
              </li>
            </ul>
            <ElButton
              type="primary"
              class="btn-add-task"
              @click.stop="addTaskToCurrentTodoList"
              title="Add a new task"
            >
              <span style="margin-left: 0.5em;">Add a task</span>
            </ElButton>
          </template>
        </div>
      </div>
      <ElCol :span="8" class="container-column task-detail-column" v-if="currentTask">
        <h1 class="title">Detail</h1>
        <div shadow="never" class="column-panel">
          <div class="header">
            <input v-model="currentTask.title" class="input-title" placeholder="Name of this Tsk" />
            <ElButton
              circle
              :icon="IconDelete"
              @click.stop="confirmToDeleteTask(currentTask).then(() => currentTask.value = null)"
              title="Delete this task"
            ></ElButton>
            <ElButton
              circle
              :icon="IconClose"
              @click.stop="currentTask.value = null"
              title="Close panel"
            ></ElButton>
          </div>
          <div class="detail" style="display: flex; flex-direction: column; flex: 1;">
            <ElCheckbox size="large" border v-model="currentTask.hasDone">Completed</ElCheckbox>
            <textarea
              style="margin-top: 1rem"
              placeholder="TODO notes ..."
              v-model="currentTask.note"
            ></textarea>
            <ElDatePicker
              style="margin-top: 1rem; width: 100%;"
              v-model="currentTask.dueTime"
              type="datetime"
              placeholder="Due time ..."
            />
          </div>
        </div>
      </ElCol>
    </ElRow>
  </div>
</template>
<script setup>
import { ElRow, ElCol, ElButton, ElDivider, ElBadge, ElCheckbox, ElMessageBox, ElDatePicker } from "element-plus";
import { Delete as IconDelete, Close as IconClose } from "@element-plus/icons-vue";
import { nextTick, onMounted, ref } from "vue";

let todoListId = 0;
let todoLists = ref([]); // should be ref here.
let currentTodoList = ref(null);
let currentTask = ref(null);

// The DOM references
let $taskList = ref(null);
let $todoList = ref(null);

onMounted(() => {
  const storage = localStorage.getItem("todobook__todolist");
  if (storage) {
    todoLists.value = JSON.parse(storage, function (key, value) {
      if (key == "dueTime" && value !== null) return new Date(value);
      else return value;
    })
  }

  if (todoLists.value.length == 0) {
    todoListId = 0;
  } else {
    todoListId = Math.max(...todoLists.value.map(todo => todo.id)) + 1;
  }
});

function addTodoList() {
  todoLists.value.push({
    id: ++todoListId,
    title: "New TODO",
    tasks: [],
  });

  currentTodoList.value = todoLists.value[todoLists.value.length - 1];
  currentTask.value = null;
  nextTick(() => {
    $todoList.value.scrollTop = $todoList.value.scrollHeight;
  })
}

function openTodoList(todo) {
  currentTodoList.value = todo;
  currentTask.value = null;
}

function isTodoSelected(todo) {
  return currentTodoList.value && currentTodoList.value.id == todo.id;
}

function addTaskToCurrentTodoList() {
  currentTodoList.value.tasks.push({
    title: "New task",
    hasDone: false,
    note: "",
    dueTime: null,
  });

  currentTask.value = currentTodoList.value.tasks[currentTodoList.value.tasks.length - 1];
  nextTick(() => {
    $taskList.value.scrollTop = $taskList.value.scrollHeight;
  });
}

function countUndoneTask(todo) {
  let count = 0;
  for (const task of todo.tasks) {
    if (!task.hasDone) count++;
  }
  return count;
}

/**
 * Note that the parameter has been unwrapped.
 * @param {*} task 
 */
function confirmToDeleteTask(task) {
  return ElMessageBox.confirm(
    `Task "${task.title}" will be deleted permanently.`,
    "Delete Task"
  ).then(() => {
    /** 
     * If the task we are going to delete is shown on the detail panel (a.k.a current task),
     * We have to let the task next to it to be the new current task.
     */
    if (currentTask.value == task) {
      const tasklistSize = currentTodoList.value.tasks.length;
      if (tasklistSize <= 1) {
        currentTask.value = null;
      } else {
        const indexOfTask = currentTodoList.value.tasks.findIndex(t => t == task);
        if (indexOfTask == tasklistSize - 1) {
          currentTask.value = currentTodoList.value.tasks[indexOfTask - 1];
        } else {
          currentTask.value = currentTodoList.value.tasks[indexOfTask + 1]
        }
      }
    }

    currentTodoList.value.tasks = currentTodoList.value.tasks.filter((t) => t != task);
  });
}

function confirmToDeleteTodoList(todoList) {
  ElMessageBox.confirm(
    `TODO list ${todoList.title} will be deleted permanently.`,
    "Delete Todo"
  ).then(() => {
    /** 
     * If the task we are going to delete is shown on the detail panel (a.k.a current task),
     * We have to let the task next to it to be the new current task.
     */
    if (currentTodoList.value == todoList) {
      const tasklistSize = todoLists.value.length;
      if (tasklistSize <= 1) {
        currentTodoList.value = null;
      } else {
        const indexOfTodoList = todoLists.value.findIndex(t => t == todoList);
        if (indexOfTodoList == tasklistSize - 1) {
          currentTodoList.value = todoLists.value[indexOfTodoList - 1];
        } else {
          currentTodoList.value = todoLists.value[indexOfTodoList + 1];
        }
      }
    }

    todoLists.value = todoLists.value.filter((todo) => todo != todoList);
    currentTask.value = null;
  });
}

window.addEventListener("beforeunload", () => {
  const serialized = JSON.stringify(todoLists.value);
  localStorage.setItem("todobook__todolist", serialized);
});

</script>
<style scoped>
.background {
  height: 100vh;
  width: 100vw;

  display: flex;
  justify-content: center;
  align-items: center;

  background: var(--el-color-primary-light-9);
}

.container {
  height: calc(100% - 6rem);
  flex: 1;
  max-width: calc(1920px * 0.9);
  margin: 3rem;
}

.container-column {
  display: flex; /* Override ElementPlus's rule */
  flex-direction: column;
  flex: 1;
}

.container-column.gutter {
  margin: 0 1rem;
}

.container-column .title {
  flex: 0;
  margin: 0;
  margin-bottom: 0.25em;
  margin-left: 3px;

  color: var(--el-text-color-primary);
}

.container-column .column-panel {
  display: flex;
  flex-direction: column;

  flex: 1 1 300px;
  border: 1px solid var(--el-border-color);
  border-radius: var(--el-border-radius-base);
  background: white;
  overflow: hidden;
}

.todolist-column > .column-panel.empty,
.tasklist-column > .column-panel.empty {
  display: flex;
  flex-direction: column;
  height: 100%;
  width: 100%;
  justify-content: center;
  align-items: center;

  color: var(--el-text-color-secondary);

  font-style: italic;
}

.tasklist-column .input-title,
.task-detail-column .input-title {
  border: none;
  font-size: var(--el-font-size-large);
  flex: 1;
  outline: none;
  color: inherit;
  width: 0;
}

.tasklist-column .input-title {
  font-weight: bold;
}

.tasklist-column > .column-panel > .header,
.task-detail-column > .column-panel > .header {
  display: flex;
  padding: 1em;
  padding-top: calc(1em + 4px);
  border-bottom: 1px dashed var(--el-border-color);
}

.todolist-column > .column-panel > .btn-add-list,
.tasklist-column > .column-panel .btn-add-task {
  width: 100%;
  height: 3em;
  border-radius: 0;
}

.todolist-column > .column-panel > .list,
.tasklist-column > .column-panel > .list {
  flex: 1;
  list-style-type: none;
  overflow: auto;
  width: 100%;
  padding: 0;
}

.todolist-column > .column-panel > .list > li,
.tasklist-column > .column-panel > .list > li {
  display: flex;
  align-items: center;

  height: 2.5em;

  padding: 0 1em;
  margin: 5px 1em;

  cursor: pointer;

  user-select: none;
}

.todolist-column > .column-panel > .list > li:hover:not(.selected),
.tasklist-column > .column-panel > .list > li:hover:not(.selected) {
  background: var(--el-color-info-light-9);
  border-radius: var(--el-border-radius-base);
}

.todolist-column > .column-panel > .list > li.selected:hover,
.tasklist-column > .column-panel > .list > li.selected:hover {
  border-radius: var(--el-border-radius-base);
}

.todolist-column > .column-panel > .list > li:active {
  transform: scale(0.99);
}

.todolist-column > .column-panel > .list > li.selected,
.tasklist-column > .column-panel > .list > li.selected {
  background: var(--el-color-primary-light-9);
  border-radius: var(--el-border-radius-base);
}

.task-detail-column > .column-panel .detail {
  margin: 1em 1.5em;
}

.task-detail-column > .column-panel .detail textarea {
  resize: none;
  flex: 1;
  outline: none;
  border: 1px solid var(--el-border-color-light);
  border-radius: var(--el-border-radius-base);
  font-family: unset;
  color: unset;
  font-size: var(--el-font-size-medium);
  box-sizing: border-box;
  padding: 0.75em;
}
</style>

<style>
body,
html {
  margin: 0;
  padding: 0;
  font-family: "Helvetica Neue", Helvetica, "PingFang SC", "Hiragino Sans GB",
    "Microsoft YaHei", "微软雅黑", Arial, sans-serif;
  color: var(--el-text-color-primary);
}

::-webkit-scrollbar {
  width: 6px;
}

::-webkit-scrollbar-thumb {
  background-color: #0003;
  border-radius: 6px;
}
</style>