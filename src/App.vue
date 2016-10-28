<style>
body {
  font-family: Helvetica, sans-serif;
}
</style>

<template>
<div>
  <p>UID: {{uid || 'None'}}</p>
  <div v-if="user">
    <input v-model.trim="newTodoText" @keyup.enter="addTodo" placeholder="Add new todo">
    <ul>
      <li v-for="todo in todos">
        <div v-if="todo.owner === user.uid">
          <input :value="todo.text" @input="updateTodoText(todo, $event.target.value)">
          <button @click="removeTodo(todo)">X</button>
        </div>
        <div v-else>
          {{todo.text}}
        </div>
      </li>
    </ul>
  </div>
  <div v-else></div>
</div>
</template>

<script>
import firebase from './firebase'

const auth = firebase.auth()
const db = firebase.database()

const todosRef = db.ref('todos')

export default {
  data() {
    return {
      user: null,
      newTodoText: '',
    }
  },
  computed: {
    uid() {
      if (!this.user) {
        return null
      }
      return this.user.uid
    }
  },
  firebase: {
    todos: todosRef
  },
  mounted() {
    auth.onAuthStateChanged(user => {
      if (!user) {
        auth.signInAnonymously().catch(err => console.error(err))
      }
      this.user = user
    })
  },
  methods: {
    addTodo() {
      if (!this.newTodoText || !this.user) {
        return
      }
      todosRef.push({
        text: this.newTodoText,
        owner: this.user.uid,
      }).then(_ => {
        this.newTodoText = ''
      })
    },
    updateTodoText(todo, newText) {
      todosRef.child(todo['.key']).child('text').set(newText)
    },
    removeTodo(todo) {
      todosRef.child(todo['.key']).remove()
    }
  }
}
</script>
