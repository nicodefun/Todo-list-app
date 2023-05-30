<script setup>
import { ref, computed, onMounted, watch, nextTick } from "vue";
import lightHeader from "./images/bg-desktop-light.jpg";
import darkHeader from "./images/bg-desktop-dark.jpg";
import moon from "./images/icon-moon.svg";
import sun from "./images/icon-sun.svg";

const todoList = ref([]);
const todoInput = ref(null);
const mode = ref(true);
const state = ref("active");
const filteredArray = ref([]);
const isClicked = ref(false);
const numOfItems = ref(null);

//mode
const headerSrc = computed(() => {
  return mode.value ? lightHeader : darkHeader;
});
const iconImg = computed(() => {
  return mode.value ? moon : sun;
});

//todos
const todoRender = () => {
  if (!isClicked.value) {
    numOfItems.value = todoList.value.length;
    return todoList.value.sort((a, b) => b.createdAt - a.createdAt);
  }
  numOfItems.value = filteredArray.value.length;
  return filteredArray.value.sort((a, b) => b.createdAt - a.createdAt);
};

const todoListAsc = computed(() => {
  return todoRender();
});


//methods
const addTodo = () => {
  if (todoInput.value.trim() === "" && todoInput.value === null) {
    return;
  } else {
    todoList.value.push({
      content: todoInput.value,
      state: state.value,
      createdAt: new Date().getTime(),
    });
    todoInput.value = "";
  }
};
const remove = (todo) => {
  isClicked.value = false;
  todoList.value = todoList.value.filter((t) => t != todo);
};
const checked = (todo) => {
  if (todo.state === "active") {
    todo.state = "completed";
    return;
  }
  todo.state = "active";
};
const switchMode = () => {
  mode.value = !mode.value;
};

const clear = () => {
  isClicked.value = false;
  todoList.value = [];
};

const filterClicked = (e) => {
  isClicked.value = true;
  if (e.target.value === "All") {
    filteredArray.value = todoList.value;
  } else if (e.target.value === "Active") {
    filteredArray.value = todoList.value.filter((t) => t.state === "active");
  } else if (e.target.value === "Completed") {
    filteredArray.value = todoList.value.filter((t) => t.state === "completed");
  }
};

onMounted(() => {
  isClicked.value = false;
  todoList.value = JSON.parse(localStorage.getItem("todos") || []);
});

//watches
watch(
  todoList,
  (newVal) => {
    localStorage.setItem("todos", JSON.stringify(newVal));
  },
  { deep: true }
);
</script>

<template>
  <header>
    <img class="header-img" :src="headerSrc" alt="light-mode" />
  </header>
  <main :class="mode ? 'light-mode' : 'dark-mode'">
    <div class="todo-container">
      <div class="container-top">
        <h1>TODO</h1>
        <div @click="switchMode">
          <img :src="iconImg" alt="moon" />
        </div>
      </div>
      <form
        @submit.prevent="addTodo"
        class="container-input"
        :class="mode ? '' : 'dark-mode'"
      >
        <input
          v-model="todoInput"
          type="text"
          autofocus
          placeholder="e.g. add tods..."
        />
      </form>
      <section
        class="container-show-todos"
        :class="mode ? 'light-mode' : 'dark-mode'"
      >
        <ul>
          <li class="todo" v-for="todo in todoListAsc" :key="todo.content">
            <input
              class="input-radio"
              type="radio"
              :id="todo.content"
              :name="todo.content"
              :value="todo.content"
              @click="checked(todo)"
              :checked="todo.state === 'completed'"
            />
            <label :for="todo.content">{{ todo.content }}</label>
            <span class="flex-1"></span>
            <input
              class="input-click"
              type="submit"
              value="X"
              @click="remove(todo)"
            />
          </li>
        </ul>
        <div class="flex-1"></div>
        <div class="container-bottom">
          <p>{{ numOfItems }} items left</p>
          <ul>
            <input
              class="input-click"
              type="submit"
              value="All"
              @click="filterClicked"
              :class="isClicked ?'': 'all'"
            />
            <input
              class="input-click"
              type="submit"
              value="Active"
              @click="filterClicked"
            />
            <input
              class="input-click"
              type="submit"
              value="Completed"
              @click="filterClicked"
            />
          </ul>
          <input type="submit" class="btn-click" @click="clear" value="clear" />
        </div>
      </section>
    </div>
  </main>
</template>

<style scoped>
header {
  max-width: 100vw;
  position: absolute;
  height: 30vh;
  top: 0;
}
header img {
  width: 100%;
  object-fit: cover;
  transition: all 1s ease-in-out;
}
main {
  height: 100vh;
  display: flex;
  justify-content: center;
  align-items: flex-start;
  transition: all 0.1s ease-in;
}
main.light-mode {
  background-color: var(--lt-light-grey);
}
.dark-mode {
  background-color: var(--dt-very-dark-blue);
}
.todo-container {
  width: 40vw;
  z-index: 10;
  display: flex;
  margin-top: 5rem;
  flex-direction: column;
}
h1 {
  color: white;
}
.input-radio:checked {
  border: none;
  background: linear-gradient(75deg, hsl(192, 100%, 67%), hsl(280, 87%, 65%));
}
.input-radio:checked:after {
  padding-left: 3px;
  content: " ✓";
  font-size: 15px;
  font-weight: 400;
  color: #fff;
  position: absolute;
}

/* container-top */
.container-top {
  display: flex;
  justify-content: space-between;
}
input:focus {
  outline: none !important;
}
.container-top h1 {
  letter-spacing: 0.5rem;
  font-weight: 600;
  margin-bottom: 2rem;
}
::placeholder {
  color: var(--lt-light-grey);
}
section {
  background-color: white;
  border-radius: 0.15rem;
  height: 100%;
  box-shadow: rgba(0, 0, 0, 0.16) 0px 10px 36px 0px,
    rgba(0, 0, 0, 0.06) 0px 0px 0px 1px;
}

.container-input {
  height: 3rem;
  margin-bottom: 1rem;
  background-color: white;
}
.container-input.dark-mode,
.container-show-todos.dark-mode {
  background-color: var(--dt-ery-dark-desaturated-blue);
  color: white;
}
.container-input input {
  background-color: transparent;
  padding: 0rem 1rem;
  display: block;
  width: 100%;
  appearance: none;
  border: none;
  border-radius: 0.15rem;
  height: 100%;
  font-size: 1.2rem;
  color: var(--lt-dark-grayish-blue);
}
.container-show-todos {
  display: flex;
  flex-direction: column;
  background-color: white;
}

.flex-1 {
  flex: 1 1 0%;
}
.container-bottom {
  display: flex;
  justify-content: space-between;
}
.input-click,
.btn-click {
  border: none;
  appearance: none;
  text-decoration: none;
  cursor: pointer;
}
input[type="submit"],
.btn-click {
  background-color: white;
  font-size: 0.8rem;
  color: var(--lt-dark-grayish-blue);
  font-family: inherit;
  background-color: transparent;
}

input.all{
  color:var(--main-bright-blue);
}

input[type="submit"]:hover,
.btn-click:hover {
  color: var(--dt-very-dark-blue);
}
ul {
  list-style: none;
}
li {
  background-color: transparent;
  height: 3rem;
  border-bottom: 1px solid var(--lt-light-grey);
  padding: 1rem;
  display: flex;
  font-weight: 500;
  color: var(--lt-dark-grayish-blue);
  align-items: center;
  font-size: 1.1rem;
}
.input-radio {
  appearance: none;
  border: none;
  width: 1.2rem;
  height: 1.2rem;
  border-radius: 999px;
  border: 1px solid var(--lt-light-grey);
  margin-right: 0.8rem;
}
/* container-bottom */
.container-bottom {
  padding: 1rem;
}
.container-bottom p {
  color: var(--lt-dark-grayish-blue);
  font-size: 0.8rem;
}
.container-bottom ul {
  width: 40%;
  display: flex;
  justify-content: space-around;
}
</style>
