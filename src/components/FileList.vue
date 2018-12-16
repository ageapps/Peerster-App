<template>
  <div class="card">
      <div class="card-header">
        {{title}}
      </div>
      <div class="card-body">    
        <ul class="list-group">
          <li class="list-group-item" v-for="(file,key) in files" :key = "key">
            <div class="card">
            <div class="card-header bg-dark text-white">
              {{file.name}}
              <span class="badge badge-secondary">{{file.size/1000}} KB</span>
              <span v-show="file.blob" class="badge badge-success">File Indexed</span>
              <span v-show="!file.blob" class="badge badge-danger">Not Indexed</span>
            </div>
            <div class="card-body">
              <div v-show="file.blob">
                <a target="blanc" v-bind:href="getUrl(file.name)">[{{key}}]</a>
              </div>
              <div v-show="!file.blob">
                [{{key}}]
              </div>
              <button v-show="!file.blob" @click="requestFile(file)" class="btn btn-secondary">Request file</button>
            </div>
            </div>
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
    },
    requestFile(file){
        if (file && !file.blob){
          this.$emit('request-file', file.name)
        }
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
