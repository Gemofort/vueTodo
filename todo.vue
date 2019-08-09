const Field = {
  data: function () {
    return {
      fieldValue: ''
    }
  },
  template: `<div>
  <form action="POST"></form>
    <h3>Add task</h3>
    <input v-model="fieldValue" type="text" id="fieldInput">
    <button type="submit" @click="onFormSubmit($event)">submit</button>
  </div>`,
  methods: {
    onFormSubmit: function (e) {
      if (this.fieldValue) {
        this.$emit('onStatusChange', this.fieldValue);
      }
      document.getElementById("fieldInput").value = ''
    },
  },
}

const Task = {
  props: {
    task: Object
  },

  name: 'task-item',

  methods: {
    switchDelete: function () {
      this.$emit('onRemoveItem', this.task)
    },
    mark: function() {
      this.$emit('onDone', this.task)
    }
  },

  mounted: function () {
    console.log('Component mounted');
  },

  template: `
  <div class="task" v-on:click="mark">
    <div>
      <span> {{task.name}} </span> 
      <span v-if="task.isDone">
        <i class="fas fa-check"></i>
      </span>
    </div>
    <div>
      <i class="fas fa-times deleteButton" v-on:click="switchDelete"></i>
    </div>
  </div>
  `,
}

Vue.component('task-list', {
  name: 'task-list',
  props: {
    tasks: Array
  },
  components: {
    taskListItem: Task,
    formCustom: Field
  },
  template: `<div>
    <div v-for="task in tasks" class="taskListInner">
        <task-list-item :task="task" @onRemoveItem="remove($event)" @onDone="markDone($event)"></task-list-item>
    </div>
  </div>`,
  methods: {
    markDone: function(task) {
      task.isDone = !task.isDone;
    },
    remove: function(task){
      this.$emit('onRemove', task);
    }
  }
})

const app = new Vue({
  el: '#app',
  data: function () {
    return {
      tasks: [
        {
          name: 'Task1',
          isDone: true
        },
        {
          name: 'Task2',
          isDone: false
        },
        {
          name: 'Task3',
          isDone: false
        }
      ],
      lookFor: 'all',
      pattern: ''
    }
  },
  computed: {
    filteredList() {
      return this.tasks.filter(task => {
        return task.name.toLowerCase().includes(this.pattern.toLowerCase());
      });
    }
  },
  methods: {
    all: function(){
      this.lookFor = 'all'
    },
    done: function(){
      this.lookFor = 'done'
    },
    notDone: function() {
      this.lookFor = 'notDone'
    },
    notify: function (task) {
      this.tasks.push({
        name: task,
        isDone: false
      });
    },
    remove: function (task) {
      this.tasks.splice(this.tasks.indexOf(task), 1);
    }
  },
  components: {
    formCustom: Field
  },
  template: `
  <div>
    <header>
      <h1>Search for: {{pattern}}</h1>
      <input v-model="pattern" type="text" class="input"></input>
    </header>

    <div class="buttons">
      <button class="button" @click="all">All</button>
      <button class="button" @click="done">Done</button>
      <button class="button" @click="notDone">In progress</button>
    </div>

    <div class="tasks">
      <form-custom @onStatusChange="notify($event)" class="inputForm"></form-custom>
      <task-list class="taskList" @onRemove="remove($event)" 
        v-if="lookFor === 'all' " :tasks="filteredList"
      ></task-list>
      <task-list class="taskList" @onRemove="remove($event)" 
        v-else-if="lookFor === 'done' " 
        :tasks="filteredList.filter(elem => elem.isDone)"
      ></task-list>
      <task-list class="taskList" @onRemove="remove($event)" 
        v-else-if="lookFor === 'notDone' " 
        :tasks="filteredList.filter(elem => !elem.isDone)"
      ></task-list>
    </div>

  </div>
  `
})