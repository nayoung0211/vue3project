<template>
<div>
<router-view/>
<div class="container">
    <h2>To-Do List</h2>
    <input
    class="form-control"
    type="text" 
    v-model="searchText"
    placeholder="Search"
    @keyup.enter="searchTodo"
    >
    <hr />
    <TodoSimpleForm @add-todo="addTodo" />
    <div style="color: red">{{ error }}</div>
    
    <div v-if="!todos.length">
    There is nothing to display
    </div>
    <TodoList 
    :todos="todos" 
    @toggle-todo="toggleTodo"
    @delete-todo="deleteTodo"
    />
    <hr />
    <nav aria-label="Page navigation example">
    <ul class="pagination">
        <li v-if="currentPage !== 1" class="page-item">
        <a style="cursor: pointer" class="page-link" @click="getTodos(currentPage - 1)">
            Previous
        </a>
        </li>

        <li
        v-for="page in numberOfPages"
        :key="page"
        class="page-item"
        :class="{ active: currentPage === page }"
        >
        <a style="cursor: pointer" class="page-link" @click="getTodos(page)">{{page}}</a>
        </li>

        <li v-if="currentPage < numberOfPages" class="page-item">
        <a style="cursor: pointer" class="page-link" @click="getTodos(currentPage + 1)">Next</a>
        </li>
    </ul>
    </nav>
</div>
</div>
</template>

<script>
import { ref, computed, watch } from 'vue';
import TodoSimpleForm from '@/components/TodoSimpleForm.vue';
import TodoList from '@/components/TodoList.vue';
import axios from 'axios';

export default {
components: {
TodoSimpleForm,
TodoList,
},
setup() {
const todos = ref([]);
const error = ref('');
const numberOfTodos = ref(0);
const limit = 5;
const currentPage = ref(1);
const searchText = ref('');

const numberOfPages = computed(() => {
    return Math.max(Math.ceil((numberOfTodos.value || 0) / limit), 1);
});

const getTodos = async (page = currentPage.value) => {
    currentPage.value = Number(page); // 숫자로 변환
    error.value = '';
    try {
    // 전체 개수 계산
    const allRes = await axios.get(`http://localhost:3000/todos?subject_like=${searchText.value}`);
    numberOfTodos.value = allRes.data.length;

    // 현재 페이지 데이터 가져오기
    const res = await axios.get(
        `http://localhost:3000/todos?_sort=id&_order=desc&subject_like=${searchText.value}&_page=${currentPage.value}&_limit=${limit}`
    );
    todos.value = res.data;
    } catch (err) {
    console.log(err);
    error.value = 'Something went wrong.';
    }
};

getTodos();

const addTodo = async (todo) => {
    error.value = '';
    try {
    await axios.post('http://localhost:3000/todos', {
        subject: todo.subject,
        completed: todo.completed,
    });
    getTodos(1);
    } catch (err) {
    console.log(err);
    error.value = 'Something went wrong.';
    }
};

const deleteTodo = async (index) => {
    error.value = '';
    const id = todos.value[index].id;
    try {
    await axios.delete('http://localhost:3000/todos/' + id);
    getTodos(1);
    } catch (err) {
    console.log(err);
    error.value = 'Something went wrong.';
    }
};  

const toggleTodo = async (index,checked) => {
    error.value = '';
    const id = todos.value[index].id;
    try {
    await axios.patch('http://localhost:3000/todos/' + id, {
        completed: checked
    });
    todos.value[index].completed = checked;
    } catch (err) {
    console.log(err);
    error.value = 'Something went wrong.';
    }
};

let timeout = null;
const searchTodo = () => {
    clearTimeout(timeout);
    getTodos(1);
};

watch(searchText, () => {
    clearTimeout(timeout);
    timeout = setTimeout(() => {
    getTodos(1);
    }, 2000);     
});

return {
    searchTodo,
    todos,
    addTodo,
    deleteTodo,
    toggleTodo,
    searchText,
    error,
    numberOfPages,
    currentPage,
    getTodos,
};
}
}
</script>

<style>
.todo {
color: gray;
text-decoration: line-through;
}
</style>
