<template>
  <div>
    <b-form-row>
      <b-col cols="10">
        <b-form-input v-model="query" type="text" />
      </b-col>
      <b-col cols="2">
        <b-button block @click="searchTodo">検索</b-button>
      </b-col>
    </b-form-row>
    <b-button class="my-2" variant="primary" block @click="openRegisterModal"
      >TODOを追加</b-button
    >

    <b-card
      v-for="todo in todoList"
      :key="todo.objectID"
      class="my-2"
      :title="todo.title"
    >
      <p>{{ todo.description }}</p>
    </b-card>

    <b-modal
      v-model="registerModalIsVisible"
      @ok="registerTodo"
      @calcel="clearInput"
    >
      <b-form-group label="Title" label-for="title">
        <b-form-input id="title" v-model="todoInput.title" type="text" />
      </b-form-group>
      <b-form-group label="Description" label-for="description">
        <b-form-input
          id="description"
          v-model="todoInput.description"
          type="text"
        />
      </b-form-group>
    </b-modal>
  </div>
</template>

<script>
import * as algoliasearch from 'algoliasearch'
import config from '~/algolia.config.js'

const client = algoliasearch(config.appId, config.apiKey)
const index = client.initIndex('todo')

export default {
  async asyncData() {
    const searchResult = await index.search('')
    return {
      todoList: searchResult.hits,
    }
  },
  data() {
    return {
      todoInput: {
        title: '',
        description: '',
        done: false,
      },
      registerModalIsVisible: false,
      query: '',
      todoList: [],
    }
  },
  methods: {
    clearInput() {
      this.todoInput.title = ''
      this.todoInput.description = ''
      this.todoInput.done = false
    },
    openRegisterModal() {
      this.registerModalIsVisible = true
    },
    async registerTodo() {
      await index.saveObject(this.todoInput, {
        autoGenerateObjectIDIfNotExist: true,
      })
      this.clearInput()
    },
    async searchTodo() {
      const searchResult = await index.search(this.query)
      this.todoList = searchResult.hits
    },
  },
}
</script>
