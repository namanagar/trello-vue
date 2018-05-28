<template>
  <div class="col-lg-12">
    <div class="row">
      <div class="col-lg-3">
        <h3>Create a List</h3>
        <input v-model="name" class="form-control" placeholder="Name">
        <button class="btn btn-primary" @click="submit">Submit</button>
      </div>
    </div>
    <hr>
  </div>
</template>

<script>
module.exports = {
  data: function() {
    return {
      name: ''
    }
  },
  props: ['lengthlists'],
  methods: {
    submit() {
      if (this.name === ''){
        alert("Must submit at least one character!")
      }
      else{
        this.$emit('create-list', {name : this.name, index: this.lengthlists})
        this.name = ''
      }
    }
  }
}
</script>

<style scoped>
div{
  margin-top: .5em;
  margin-left: 2em;
}
button{
  margin-top: 1em;
}
</style>
