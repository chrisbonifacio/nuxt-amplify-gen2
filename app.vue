// vue.app
<script setup lang="ts">
import { Authenticator } from '@aws-amplify/ui-vue';
import '@aws-amplify/ui-vue/styles.css';
import { onMounted, ref } from 'vue';
import type { Schema } from '@/amplify/data/resource';

// create a reactive reference to the array of todos
const todos = ref<Schema['Todo'][]>([]);

async function listTodos() {
  try {
    // fetch all todos
    const { data } = await useNuxtApp().$Amplify.GraphQL.client.models.Todo.list();
    todos.value = data;

    // subscribe to todos
    subscribeToTodo();
  } catch (error) {
    console.error('Error fetching todos', error);
  }
}

async function createTodo() {
  try {
    // create a new Todo
    const title = window.prompt('Enter the title:');
    
    // if title is empty or user cancels, do nothing
    if (!title) return;

    const { errors, data: newTodo } = await useNuxtApp().$Amplify.GraphQL.client.models.Todo.create({
      content: title,
      done: false,
      priority: 'medium'
    });

    console.log(errors, newTodo);

    // after creating the todo, refresh the list
    await listTodos();
  } catch (error) {
    console.error('Error creating todo', error);
  }
}

async function getUser() {
  try {
    const response = await fetch('http://localhost:3000/api/current-user')

    if (response.ok) {
      const user = await response.json()

      console.log({user})
    }

  } catch (error) {
   console.log(error) 
  }

}

function subscribeToTodo() {
  const sub = useNuxtApp().$Amplify.GraphQL.client.models.Todo.observeQuery().subscribe(({ items }) => {
    todos.value = items;
  });

  return sub;
}

// fetch todos when the component is mounted
 onMounted(() => {
  getUser()
  listTodos();
});

</script>


<template>
  <Authenticator>
    <template v-slot="{ user, signOut }">
      <h1>Hello, Amplify 👋</h1>
      <button @click="createTodo"> Create</button>
      <ul>
        <li v-for="todo in todos" :key="todo.id">{{ todo.content }}</li>
      </ul>
      <button @click="signOut">Sign Out</button>
    </template>
  </Authenticator>
</template>