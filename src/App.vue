<template>
  <div id="app">
    <section class="todoapp" v-cloak>
      <header class="header">
        <h1>todos</h1>
        <input class="new-todo" autofocus autocomplete="off" placeholder="What needs to be done?" v-model="newTodo"
          @keyup.enter="addTodo">
      </header>
      <section class="main">
        <input id="toggle-all" class="toggle-all" type="checkbox" v-model="allDone">
        <label for="toggle-all">Mark all as complete</label>
        <ul class="todo-list">
          <li :class="['todo', { completed: todo.completed === true }, { editing: editTodo.id === todo.id }]"
            v-for="(todo, index) in showTodo" :key="index">
            <div class="view">
              <input class="toggle" type="checkbox" v-model="todo.completed">
              <label @dblclick="editingMode(todo)">{{ todo.title }}</label>
              <button class="destroy" @click="removeTodo(todo)"></button>
            </div>
            <input class="edit" type="text" v-todo-focus="todo.id == editTodo.id" v-model="todo.title"
              @blur="editDone(todo)" @keyup.enter="editDone(todo)" @keyup.esc="editCancel(todo)">
          </li>
        </ul>
      </section>
      <footer class="footer">
        <span class="todo-count">
          <strong>{{ todos.length }}</strong> todo
        </span>
        <ul class="filters">
          <li><a href="#/all" :class="{ selected: filter === 'all' }" @click="changeFilter('all')">All</a></li>
          <li><a href="#/active" :class="{ selected: filter === 'active' }" @click="changeFilter('active')">Active</a>
          </li>
          <li><a href="#/completed" :class="{ selected: filter === 'completed' }"
              @click="changeFilter('completed')">Completed</a></li>
        </ul>
        <button class="clear-completed" v-show="todos.some(data => data.completed)" @click="removeCompleted">Clear
          completed</button>
      </footer>
    </section>
  </div>
</template>

<script setup>
import './assets/base.css'
import './assets/index.css'

import { ref, reactive, computed } from "vue";

const filter = ref('all');
const newTodo = ref('');
const beforeEditCache = ref('');

let editTodo = reactive({
  id: null,
  title: null,
});
const todos = reactive([]);

const showTodo = computed(() => {
  return todos.filter(data => {
    if (filter.value === 'all') {
      return true
    } else if (filter.value === 'active') {
      return !data.completed
    } else if (filter.value === 'completed') {
      return data.completed
    }
  })
})

const allDone = computed({
  get: function () {
    return todos.every(data => data.completed);
  },
  set: function (value) {
    todos.forEach(function (todo) {
      todo.completed = value;
    });
  }
})

const addTodo = () => {
  const value = newTodo.value.trim();
  if (!value) {
    return;
  }
  todos.push({ id: todos.length + 1, title: value, completed: false });
  newTodo.value = '';
}

const editingMode = (todo) => {
  beforeEditCache.value = todo.title;
  editTodo.title = todo.title;
  editTodo.id = todo.id;
}

const removeTodo = (todo) => {
  const index = todos.indexOf(todo);
  todos.splice(index, 1);
}

const editDone = (todo) => {
  if (!editTodo.title) {
    return;
  }
  if (todo.title.trim()) {
    /* 这样是不对的！虽然editTodo是用let声明的，可以重新赋值
    但是，这里是将它从一个响应式对象变成了 ull。这就导致了你在后续尝试访问 editTodo.title 和 editTodo.id会显示不存在*/
    // editTodo = null;

    //这才是正确的做法
    editTodo.title = null;
    editTodo.id = null;
    beforeEditCache.value = null;
    todo.title = todo.title.trim();
  } else {
    editCancel(todo);
  }
}

const editCancel = (todo) => {
  //这是不对的
  // editTodo = null;
  editTodo.title = null;
  editTodo.id = null;
  todo.title = beforeEditCache.value;
  beforeEditCache.value = null;
}

const changeFilter = f => filter.value = f;


const removeCompleted = () => {
  //出错了！因为todos不能被重新赋值
  // todos = todos.filter(data => !data.completed);
  todos.splice(0, todos.length, ...todos.filter(data => !data.completed));
}

// start
const vTodoFocus = {
  updated(el, binding, vnode, prevVnode) {
    if (binding.value) {
      el.focus();
    }
  },
}
// end
</script>

