<template>
<div class="background">
  <ElRow class="container">
    <ElCol :span="8" class="container-column todolist-column">
      <h1 class="title">Todos</h1>
      <div shadow="never" class="content">
        <div v-if="todoLists.length == 0" class="state-empty-list">
          <div style="margin-bottom: 1em">Oops, Haven't added a TODO list yet...</div>
          <ElButton type="primary" style="margin-bottom: 1em;">Add a TODO list</ElButton>
        </div>
        <template v-else>
          <ul class="list">
            <li v-for="todo of todoLists" :key="todo.id">
              <span style="margin-right: auto;">{{ todo.title }}</span>
              <ElBadge :value="todo.taskUndone" type="primary" />
            </li>
            <ElDivider content-position="left" style="width: unset; margin: 1em;">That's all</ElDivider>
          </ul>
          <ElButton class="btn-add-list" type="primary">
            <span style="margin-left: 0.5em;">Add List</span>
          </ElButton>
        </template>
      </div>
    </ElCol>
    <div :span="8" class="container-column tasklist-column gutter">
      <h1 class="title">Tasks</h1>
      <div shadow="never" class="content">
          <div v-if="todoLists.length == 0" class="state-no-todo-list">
            Please add a TODO list first.
          </div>
          <div v-else-if="!currentTodoList" class="state-no-list-selected">
            Please select a TODO list first.
          </div>
          <div v-else-if="currentTodoList.length == 0">
            <div class="state-empty-list" style="margin-bottom: 1em">
              Oops, this TODO list is empty...
            </div>
            <ElButton type="primary" style="margin-bottom: 1em;">Add a task</ElButton>
          </div>
          <template v-else>
            <ul class="list">
              <li v-for="todo of currentTodoList" :key="todo.id">{{ todo.title }}</li>
            </ul>
            <ElButton type="primary" class="btn-add-task">
              <span style="margin-left: 0.5em;">Add Task</span>
            </ElButton>
          </template>
      </div>
    </div>
    <ElCol :span="8" class="container-column" v-if="currentTask">
      <h1 class="title">Details</h1>
      <div shadow="never" class="content"></div>
    </ElCol>
  </ElRow>
</div>
</template>
<script setup>
import { ElRow, ElCol, ElButton, ElDivider, ElBadge } from "element-plus";

let todoLists = [
  { id: "0", title: "Life", taskUndone: 3, },
  { id: "1", title: "Study", taskUndone: 5 },
  { id: "2", title: "Wishes fo this year", taskUndone: 10 }
];

let currentTodoList = null;

let currentTask = null;

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

.container-column .content {
  flex: 1 1 300px;
  border: 1px solid var(--el-border-color);
  border-radius: var(--el-border-radius-base);
  background: white;
  overflow: hidden;

  display: flex;
  flex-direction: column;
}


.todolist-column > .content,
.tasklist-column > .content {
    display: flex;
    flex-direction: column;
    height: 100%;
    justify-content: center;
    align-items: center;
}

.todolist-column > .content > .btn-add-list,
.tasklist-column > .content > .btn-add-task {
  width: 100%;
  height: 3em;
  border-radius: 0;
}

.todolist-column > .content > .list,
.tasklist-column > .content > .list {
  flex: 1;
  list-style-type: none;
  overflow: auto;
  width: 100%;
  padding: 0;
}


.todolist-column > .content > .list > li,
.tasklist-column > .content > .list > li {
  display: flex;
  align-items: center;

  height: 2.5em;

  padding: 0 1em;
  margin: 5px 1em;

  cursor: pointer;

  user-select: none;
}

.todolist-column > .content > .list > li:hover:not(.selected),
.tasklist-column > .content > .list > li:hover:not(.selected) {
  background: var(--el-color-info-light-9);
  border-radius: var(--el-border-radius-base);
}


.todolist-column > .content > .list > li.selected,
.tasklist-column > .content > .list > li.selected {
  background: #606266;
  border-radius: var(--el-border-radius-base);
  color: white;
}

.todolist-column > .content > .list > li.selected:hover,
.tasklist-column > .content > .list > li.selected:hover {
  border-radius: var(--el-border-radius-base);
}

.todolist-column > .content > .list > li:active,
.tasklist-column > .content > .list > li:active {
  transform: scale(0.99);
}


.todolist-column .state-empty-list,
.tasklist-column .state-no-todo-list,
.tasklist-column .state-empty-list,
.tasklist-column .state-no-list-selected,
.tasklist-column .state-empty-list {
    font-size: var(--el-font-size-large);
    color: var(--el-text-color-secondary);
    text-align: center;
}

</style>

<style>
body, html {
  margin: 0;
  padding: 0;
  font-family: 'Helvetica Neue', Helvetica, 'PingFang SC', 'Hiragino Sans GB',
  'Microsoft YaHei', '微软雅黑', Arial, sans-serif;
}
</style>