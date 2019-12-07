<template lang="html">
  <div class="modal" v-show="show">
    <div class="modal-dialog">
      <div class="modal-content">
        <div class="modal-header">
          <h4>{{ task.name }}</h4>
          <button type="button" class="close" @click="closeModal">
            <span aria-hidden="true">&times</span>
          </button>
        </div>
        <div class="modal-body">
          <div class="row-fluid">
            <div class="col-xs-12">
              <p @click="editDescription">Description: {{  task.description  }}</p>
              <p>Creation Date:  {{  task.creationdate  }}</p>
              <p @click="editDeadline">Deadline:  {{  task.deadline  }}</p>
              <p @click="switchCategory" :style="{ color: categoryColor() }">Category: {{ categoryName() }}</p>
              <p>List:  {{  task.list  }}</p>
              <small class="text-muted">Click on an item to edit it.</small>
            </div>
            <hr>
          </div>
          <div class="row-fluid">
            <div class="col-xs-3">
              <p>Images: </p>
              <ul>
                <li v-for="image in task.images" @click="removeImage(image)">
                  <img :src="image"/>
                </li>
              </div>
            </ul>
            <div class="col-xs-3">
              <p>Comments: </p>
              <ul>
                <li v-for="comment in task.comments" @click="removeComment(comment)">
                  {{  comment  }}
                </li>
              </ul>
            </div>
            <div class="col-xs-3">
              <p>Hashtags: </p>
              <ul>
                <li v-for="todo in task.todos" @click="removeTodo(todo)">
                  {{ '#' + todo  }}
                </li>
              </ul>
            </div>
            <div class="col-xs-3">
              <p>Users: </p>
              <ul>
                <li v-for="user in task.users" @click="removeUser(user)">
                  {{  user.name  }}
                </li>
              </ul>
            </div>
            <small class="text-muted">Click on an item to remove it.</small>
          </div>
        </div>
        <div class="modal-footer">
          <button class="btn btn-primary" @click="addComment">Add Comment</button>
          <button class="btn btn-primary" @click="addTodo">Add Hashtag</button>
          <button class="btn btn-primary" @click="addUser">Invite User</button>
          <label class="btn btn-primary" for="im-uploader">
            Add Image
            <input id="im-uploader" type="file" @change="fileUpload($event)" class="custom-file-input" hidden>
          </label>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
module.exports = {
  props: ['task', 'show', 'users'],
  data: function(){
    return {
    }
  },
  methods: {
    categoryName(){
      var categoryKey = this.task.category
      var query = db.ref('categories').child(categoryKey).child('name')
      var categoryName
      query.once('value', function(snapshot){
        categoryName = snapshot.val()
      })
      return categoryName
    },
    categoryColor(){
      var categoryKey = this.task.category
      var query = db.ref('categories').child(categoryKey).child('color')
      var categoryColor
      query.once('value', function(snapshot){
        categoryColor = snapshot.val()
      })
      return categoryColor
    },
    switchCategory(){
      var category = prompt("What category is this task? Press enter without hitting anything to leave it as is", this.task.category.name)
      var categories = db.ref('categories')
      var catArray = []
      //makes an array of all categories to check the existence of the category 
      categories.on('value', function(snapshot){
        snapshot.forEach(function(cat){
          var cat = cat.val()
          catArray.push(cat)
        })
      })
      // checks if the category's name is valid
      var nameCheck = catArray.filter(cat => cat.name == category)
      if (nameCheck.length == 0){
        alert("Invalid category!")
        category = ""
      }
      var query = db.ref('categories').orderByChild('name').equalTo(category)
      var categoryKey
      query.once('value', function(snapshot){
        snapshot.forEach(function(itemSnapshot) {
         categoryKey = Object.keys(snapshot.val())[0]
       })
      })
      // set the new category in firebase
      tasksRef.child(this.task['.key']).child('category').set(categoryKey)
    },
    editDescription(){
      var newDesc = prompt("What is the new description for the task?", this.task.description)
      if (newDesc.length > 0 && newDesc != null){
        tasksRef.child(this.task['.key']).child('description').set(newDesc)
      }
    },
    editDeadline(){
      var newDeadline = prompt("What is the new deadline? Please format as MM/DD/YYYY", this.task.deadline)
      //checks the mm/dd/yyyy format and updates firebase
      if (newDeadline.length == 10 && newDeadline != null && newDeadline.includes("/")){
        tasksRef.child(this.task['.key']).child('deadline').set(newDeadline)
      }
    },
    closeModal(){
      this.$emit('close')
    },
    fileUpload(event){
      // adds an image to a task using FileReader 
      var file = event.target.files[0]
      var task = tasksRef.child(this.task['.key']).child('images')
      var reader = new FileReader()
      reader.onload = function(e){
        task.push(e.target.result)
      }
      reader.readAsDataURL(file)
    },
    addComment(){
      var newComment = prompt("What is the comment you would like to add?")
      //tasksRef.child(this.task['.key']).child('comments').child(""+newComment+"").setValue(newComment) doesn't work rip
      tasksRef.child(this.task['.key']).child('comments').push(newComment)
    },
    addTodo(){
      var newtodo = prompt("What is the hashtag you would like to add?")
      tasksRef.child(this.task['.key']).child('todos').push(newtodo)
    },
    addUser(){
      var newUser = prompt("Who would you like to add to this card?")
      var users = db.ref('users')
      var userArray = []
      //makes an array of all users to check the existence of the user 
      users.on('value', function(snapshot){
        snapshot.forEach(function(user){
          var user = user.val()
          userArray.push(user)
        })
      })
      // checks if the user's name is valid
      var nameCheck = userArray.filter(user => user.name == newUser)
      if (nameCheck.length > 0){
        tasksRef.child(this.task['.key']).child('users').push(nameCheck[0])
      }
      else{
        alert("Sorry, that user doesn't exist!")
      }
    },
    //the 4 methods below can't be condensed into one removeItem(item) because the query is different for each
    removeImage(image){
      var query = tasksRef.child(this.task['.key']).child('images')
      query.once('value', function(snapshot){ 
        snapshot.forEach(function(itemSnapshot) {
          if (itemSnapshot.val() == image)
           itemSnapshot.ref.remove()
       }) 
      })
    },
    removeUser(user){
      var query = tasksRef.child(this.task['.key']).child('users').orderByChild('name').equalTo(user.name)
      query.once('value', function(snapshot){
        snapshot.forEach(function(itemSnapshot) {
         itemSnapshot.ref.remove()
       }) 
      })
    },
    removeComment(comment){
      var query = tasksRef.child(this.task['.key']).child('comments')
      query.once('value', function(snapshot){ 
        snapshot.forEach(function(itemSnapshot) {
          if (itemSnapshot.val() == comment)
           itemSnapshot.ref.remove()
       }) 
      })
    },
    removeTodo(todo){
      var query = tasksRef.child(this.task['.key']).child('todos')
      query.once('value', function(snapshot){ 
        snapshot.forEach(function(itemSnapshot) {
          if (itemSnapshot.val() == todo)
           itemSnapshot.ref.remove()
       }) 
      })
    }
  }
}
</script>

<style scoped>
/* Bootstrap has a default display: none on modal classes, so this forces it to appear and it toggles based on Vue instead */
.modal{
  display: block;
  font-size: .8em;
}
img{
  max-width: 9.375em;
  max-height: 9.375em;
}
</style>
