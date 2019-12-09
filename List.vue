<template>
  <div class="col-md-3">
    <div class="card">
      <div class="card-header">
        <h5 class="card-title">
          <a data-toggle="collapse" :href="'#' + this.index">
            {{ list.name }}
          </a>
          <button type="button" class="btn btn-light right" @click="$emit('delete-list')" title = "Delete List">
            <i class="fa fa-minus-circle"></i>
          </button>
          <button type="button" class="btn btn-light right" @click="editName(list)" title = "Edit List Name">
            <i class="fa fa-edit" aria-hidden="true"></i>
          </button>
        </h5>
      </div>
      <div class="card-block collapse show" :id="this.index">
        <template v-for="task in tasks">
          <task v-show="render(task)" :users="users" :task="task" :index="index" :length="length" :key="task.key"
          @delete-task="deleteTask(task)" @move-task-left="moveTaskLeft(task)" @move-task-right="moveTaskRight(task)"></task>
        </template>
        <hr>
        <button type="button" class="btn btn-default btn-light" @click="createTask" title = "Add Task">
          <i class="fa fa-plus" style="color: #0275d8" aria-hidden="true"></i>
        </button>
        <button v-if="index < length-1" class="btn btn-light right" @click="$emit('move-list-right')" title = "Move List Right">
          &raquo;</button>
        <button v-if="index > 0" class="btn btn-light right" @click="$emit('move-list-left')" title = "Move List Left">
          &laquo;</button>
      </div>
    </div>
  </div>
</template>
<script>
module.exports = {
  props: ['list', 'index', 'length', 'tasks', 'users', 'checkedcategories', 'checkeddates'],
  data: function() {
    return {
    }
  },
  methods: {
    editName(list){
      var newName = prompt("What would you like to rename list " + list.name + "? Beware that the list should be empty or your cards will disappear!", list.name)
      if (newName != null && newName != ""){
        this.$emit("edit-list-name", [list, newName])
      }
    },
    createTask(){
      var name = prompt("What is the name of the new task?", "New Task")
      var description = prompt("Describe the task " + name)
      var category = prompt("What category is this task? A category MUST be added")
      // gets and formats the date strings to look nice
      var creationdate = new Date()
      if (creationdate.getMonth() < 10){
        creationdate = "0" + creationdate.toLocaleString().substring(0, 9)
      }
      else creationdate = creationdate.toLocaleString().substring(0, 10)
        var deadline = new Date()
      deadline.setDate(deadline.getDate() + 7)
      var deadlinestring  = deadline.toLocaleString().substring(0, 10)
      if (deadline.getMonth() < 10){
        deadlinestring = "0" + deadline.toLocaleString().substring(0, 9)
      }
      var myList = this.list.name
      //makes an array of all categories to check the existence of the category 
      var categories = db.ref('categories')
      var catArray = []
      categories.on('value', function(snapshot){
        snapshot.forEach(function(cat){
          var cat = cat.val()
          catArray.push(cat)
        })
      })
      // checks if the category's name is valid. if it is, run the query, get the key, and push to firebase
      var nameCheck = catArray.filter(cat => cat.name == category)
      var categoryKey
      if (nameCheck.length == 0 ||  category == null || category == undefined){
        alert("Invalid category! Tasks must have categories")
      }
      else{
        var query = db.ref('categories').orderByChild('name').equalTo(category)
        query.once('value', function(snapshot){
          snapshot.forEach(function(itemSnapshot) {
           categoryKey = Object.keys(snapshot.val())[0]
         })
        })
        if (name != null && name != ''){
          this.$emit('create-task',
          {
            name: name,
            description: description,
            creationdate: creationdate,
            deadline: deadlinestring,
            list: myList,
            comments: [],
            todos: [],
            images: [],
            users: [],
            category: categoryKey
          })
        }
      }
    },
    deleteTask(task){
      this.$emit('delete-task', task)
    },
    moveTaskLeft(task){
      this.$emit('move-task-left', task)
    },
    moveTaskRight(task){
      this.$emit('move-task-right', task)
    },
    render(task){
      // function that checks if the task should be displayed in each list. Can't be computed b/c Firebase changes don't update computed values AFAIK
      var list = db.ref('allTasks/'+task['.key']+'/list')
      var listname
      list.on('value', function(snapshot){
        listname = snapshot.val()
      })
      var category = db.ref('allTasks/'+task['.key']+'/category')
      var cat
      category.on('value', function(snapshot){
        cat = snapshot.val()
      })
      return (listname == this.list.name) && (this.checkedcategories.includes(cat))
    }
  },
  components: {
    'task': httpVueLoader('Task.vue')
  }
}
</script>

<style scoped>
.card{
  margin: 1em;
}
.right{
  float: right;
}
button{
  margin: .05em;
}
</style>
