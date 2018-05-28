<template>
  <div class="card">
    <div class="card-header">
      <h5 class="card-title">{{ task.name }}
        <button type="button" class="btn btn-default right" @click="$emit('delete-task')">
          <i class="fa fa-minus-circle"></i>
        </button>
        <button type="button" class="btn btn-default right" @click="editName(task)">
          <i class="fa fa-edit" aria-hidden="true"></i>
        </button>
      </h5>
    </div>
    <div class="card-body">
      <h6 class="card-text">Task Details:</h6>
      <p class="card-text">Created: {{ task.creationdate }}</p>
      <button class="btn btn-primary" @click="openModal">Expand</button>
    </div>
    <div class="card-footer">
      <button class="btn btn-success right" v-if="this.index < this.length-1" @click="$emit('move-task-right', task)">&raquo;</button>
      <button class="btn btn-danger right" v-if="this.index > 0" @click="$emit('move-task-left', task)">&laquo;</button>
    </div>
    <modal :task="task" :users="users" :show="showModal" @close="closeModal"></modal>
  </div>
</template>
<script>
module.exports = {
  props: ['task', 'index', 'length', 'users'],
  data: function() {
    return {
      showModal: false
    }
  },
  methods: {
    editName(task){
      var newName = prompt("What would you like to rename task " + task.name + "?", task.name)
      if (newName != null && newName != ""){
        this.$root.$firebaseRefs.allTasks.child(task['.key']).child('name').set(newName)
      }
    },
    openModal() {
      this.showModal = true
    },
    closeModal() {
      this.showModal = false
    }
  },
  components: {
    'modal': httpVueLoader('Modal.vue')
  }
}
</script>

<style scoped>
.right{
  float: right;
}
button{
  margin: .05em;
}
.card{
  margin: 1.2em;
}
.card-block{
  margin: 0;
}
</style>
