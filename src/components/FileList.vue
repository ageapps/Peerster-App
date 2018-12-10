<template>
  <div class="card">
      <div class="card-header">
        {{title}}
      </div>
      <div class="card-body">    
        <ul class="list-group">
          <li class="list-group-item" v-for="(file,key) in files" :key = "key">
            [{{key}}]
            <a target="blanc" v-bind:href="getUrl(file)">{{file}}</a>
          </li>
        </ul>
      </div>
  </div>
</template>

<script>
export default {
  name: 'FilesList',
  props: {
    title: String,
    files: Object,
    baseurl: String
  },
  methods: {
    getUrl(file){
      return this.baseurl+ "/files/" + file
    }
  }
  // data() {
  //   return {
  //   }
  // }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

</style>
