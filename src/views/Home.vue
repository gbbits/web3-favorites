<template>
  <v-layout row wrap>
    <v-flex text-xs-center>
      <!-- header -->
      <h1 class="primary--text display-3 font-weight-medium my-3">TODOS</h1>
      <v-card>
        <v-list class="pa-0">
          <v-list-tile>
            <v-list-tile-action>
              <v-checkbox
                :input-value="allChecked"
                @change="toggleAll(!allChecked)"
                color="primary"
                v-if="todos.length > 0"
              ></v-checkbox>
              <v-icon
                color="primary"
                v-else
              >check</v-icon>
            </v-list-tile-action>
            <v-text-field
              :label="'New todo input'"
              @keydown.enter="addTodo"
              autofocus
              browser-autocomplete="off"
              clearable
              color="primary"
              flat
              hide-details
              maxlength="1023"
              placeholder="What needs to be done?"
              solo
              v-model="newTodo"
            ></v-text-field>
          </v-list-tile>
        </v-list>
      </v-card>
      <!-- main -->
      <v-card class="mt-3" v-show="todos.length">
        <v-progress-linear class="my-0" v-model="progressPercentage"/>
        <v-card-actions class="px-3" v-show="todos.length">
          <span class="primary--text">
            {{ remaining }} {{ remaining | pluralize('item') }} left
          </span>
          <v-spacer></v-spacer>
          <v-btn-toggle
            class="elevation-0"
            mandatory
            v-model="visibility"
            v-show="todos.length"
          >
            <v-btn
              :key="key"
              :to="key"
              :value="key"
              class="mx-0"
              color="primary"
              flat
              small
              v-for="(val, key) in filters"
            >
              {{ key | capitalize }}
            </v-btn>
          </v-btn-toggle>
        </v-card-actions>
        <v-list class="pa-0">
          <template v-for="todo in filteredTodos">
            <v-divider :key="`${todo.uid}-divider`"></v-divider>
            <TodoItem
              :key="todo.uid"
              :todo="todo"
            />
          </template>
        </v-list>
      </v-card>
      <v-btn
        @click="clearCompleted"
        block
        class="mt-3"
        color="primary"
        depressed
        round
        v-show="todos.length > remaining"
      >
        Clear completed
      </v-btn>
      <!-- footer -->
      <footer-info></footer-info>
    </v-flex>
  </v-layout>
</template>

<script>
import { mapActions } from 'vuex'
import TodoItem from '@/components/TodoItem.vue'
import FooterInfo from '@/components/FooterInfo.vue'

const filters = {
  all: todos => todos,
  active: todos => todos.filter(todo => !todo.done),
  completed: todos => todos.filter(todo => todo.done)
}

export default {
  props: ['filter'],
  components: {
    TodoItem,
    FooterInfo
  },
  data () {
    return {
      newTodo: '',
      filters: filters,
      visibility: this.filter,
      config: null
    }
  },
  computed: {
    todos () {
      return this.$store.state.todos
    },
    allChecked () {
      return this.todos.every(todo => todo.done)
    },
    filteredTodos () {
      return filters[this.visibility](this.todos)
    },
    remaining () {
      return this.todos.filter(todo => !todo.done).length
    },
    progressPercentage () {
      const len = this.todos.length
      return ((len - this.remaining) * 100) / len
    }
  },
  created() {
    // console.log(this.config)
  },
  mounted() {
    this.getConfig()
  },
  methods: {
    ...mapActions([
      'toggleAll',
      'clearCompleted'
    ]),
    addTodo () {
      const text = this.newTodo.trim()
      if (text) {
        this.$store.dispatch('addTodo', text)
      }
      this.newTodo = ''
    },
    getConfig() {
      this.axios
        .get('./config.json')
        .then((res) => {
          let configObj = {}
          let Arr = []
          const { data } = res
          data.config.forEach((item, index) => {
            let navObj = {}
            item.options.forEach((i) => {
              configObj[i.key] = i.value
              // if (i.key.indexOf('linkName') != -1) {
              //   if (i.value) {
              //     navObj.linkName = i.value;
              //   }
              // }
              // if (i.key.indexOf('linkHref') != -1) {
              //   const isHttp = this.testHttp(i.value)
              //   if (i.value && !isHttp) {
              //     navObj.linkHref = 'http://' + i.value
              //   } else {
              //     navObj.linkHref = i.value
              //   }
              // }
            })
            Arr[index] = navObj
          })
          if (configObj.webname) {
            document.title = configObj.webname
          }
          this.config = configObj

          let num = window.localStorage.getItem('addNum') || 0
          console.log(num)
          if(num == 0){
            for(var key in configObj){
            if(key != 'webname'){
              // console.log(this.$store.state.todos)
              if(configObj[key]){
                this.newTodo = configObj[key]
                this.addTodo()
              }
            }
          }
          window.localStorage.setItem('addNum', 1)
          }


          this.mediaList = Arr.filter((item) => {
            if (item.linkHref) {
              return item
            }
          })
          
        })
        .catch((error) => {
          console.log(error)
        })
    }
  },
  filters: {
    pluralize: (n, w) => n === 1 ? w : (w + 's'),
    capitalize: s => s.charAt(0).toUpperCase() + s.slice(1)
  }
}
</script>

<style lang="stylus">
h1
  opacity: 0.3
.v-text-field .v-input__slot
  padding: 0 !important
</style>
