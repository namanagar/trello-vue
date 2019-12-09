<template>
  <div class="modal" v-show="show">
    <div class="modal-dialog">
      <div class="modal-content">
        <div class="modal-header">
          <h3>Categories & Filters</h3>
          <button type="button" class="close" @click="closeModal">
            <span aria-hidden="true">&times;</span>
          </button>
        </div>
        <div class="modal-body">
          <div class="row-fluid">
            <div class="col-xs-12">
              <ul>
                <li v-for="category in categories" @click="editCategory(category)" :style="{ color: category.color }">
                  {{  category.name  }}
                </li>
              </ul>
              <small class="text-muted">Click on a label to edit it.</small>
              <button type="button" class="btn btn-default btn-primary" @click="createCategory">Create a Category</button>
            </div>
            <hr>
            <div class="row-fluid">
              <div class="col-xs-12">
                <h5>Filters</h5>
              </div>
            </div>
            <div class="row-fluid">
            <div class="col-xs-12">
              <template v-for="category in categories">
                <label>
                  <input :id="category['.key']" :value="category['.key']" type="checkbox" v-model="inputChecked">{{  category.name  }}
                </label>
              </template>
            </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
module.exports = {
  data: function() {
    return {
      inputChecked: this.checkedcategories,
    }
  },
  props: ['categories', 'checkedcategories', 'show'],
  methods: {
    closeModal(){
      this.$emit('close')
    },
    createCategory(){
      var name = prompt("What do you want to name the new category?")
      var color = prompt("Please enter a hex value for the color of the category, including the #")
      if (name != null && name.length != 0){
        if (color.charAt(0) === '#' && color.length === 7){
          this.$emit('create-cat', {name: name, color: color})
        }
        else alert("Your color is formatted wrong!")
      }
      else alert("Must enter a name!")
    },
    editCategory(category){
      var newName = prompt("What is the new name for this category?", category.name)
      //failsafe to stop users from making blank category names
      if (newName.length == 0 || newName == null){
        newName = category.name
      }
      var newColor = prompt("What is the new color for this category? Please enter it as a hex value with a #", category.color)
      //makes sure color is valid hex, then updates firebase
      if (newColor.charAt(0) === '#' && newColor.length === 7){
        this.$emit('edit-cat', {category: category, name: newName, color: newColor})
      }
      else alert("Your color is formatted wrong!")
    },
  },
  watch: {
    inputChecked(val) {
      this.$emit('input', val);
    }
  }
}
</script>

<style scoped>

button{
  margin-top: 1em;
}

/* 
img{
  max-width: 18.750em;
  max-height: 18.750em;
} */

.modal {
  display: block;
}

</style>
