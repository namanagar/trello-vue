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
            </div>
            <hr>
            <div class="row-fluid">
              <div class="col-xs-12">
                <input v-model="newCat" class="form-control" placeholder="Enter category name">
                <label class="btn btn-primary">Set Category Color
                  <input type="color" value="#ff0000" @change="setColor($event)" hidden>
                </label>
                <button type="button" class="btn btn-default btn-success" :disabled="!!newCat" @click="createCategory">Create a New Category</button>
              </div>
            </div>
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
      newCat: '',
      color: ''
    }
  },
  props: ['categories', 'checkedcategories', 'show'],
  methods: {
    closeModal(){
      this.$emit('close')
    },
    createCategory(){
      this.$emit('create-cat', {name: this.newCat, color: this.color})
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
    setColor(event){
      this.color = event.target.value
    }
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
