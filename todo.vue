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
      this.$emit('onRemove', this.task)
    },
    mark: function() {
      this.$emit('onDone', this.task)
    }
  },

  mounted: function () {
    console.log('Component mounted');
  },

  template: `
  <div>
    <div v-on:click="mark">
      Task: <span> {{task.name}} </span> <span v-if="task.isDone">(Done!)</span>
    </div>
    <button v-on:click="switchDelete">delete</button>
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
    <div v-for="task in tasks">
        <task-list-item :task="task" @onRemove="remove($event)" @onDone="markDone($event)"></task-list-item>
    </div>
  </div>`,
  methods: {
    notify: function (task) {
      this.tasks.push({
        name: task,
        isDone: false
      });
    },
    markDone: function(task) {
      task.isDone = !task.isDone
    },
    remove: function (task) {
      this.tasks.splice(this.tasks.indexOf(task), 1);
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
    }
  },
  components: {
    formCustom: Field
  },
  template: `
  <div>
    <button @click="all">All tasks</button>
    <button @click="done">Done tasks</button>
    <button @click="notDone">In progress tasks</button>
    <h1>Search for: {{pattern}}</h1>
    <input v-model="pattern" type="text"></input>
    <form-custom @onStatusChange="notify($event)"></form-custom>
    <task-list v-if="lookFor === 'all' " :tasks="filteredList"></task-list>
    <task-list v-else-if="lookFor === 'done' " :tasks="filteredList.filter(elem => elem.isDone)"></task-list>
    <task-list v-else-if="lookFor === 'notDone' " :tasks="filteredList.filter(elem => !elem.isDone)"></task-list>
  </div>
  `
})