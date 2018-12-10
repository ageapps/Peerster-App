<template>
    <div class="card">
      <div class="card-body">
        <div class="input-group mb-3">
          <div class="input-group-prepend">
            <button class="btn btn-outline-secondary" @click="toggleUpload()" type="button" id="button-addon2">Select File</button>
          </div>
          <input hidden type="file"  class="form-control" id="file" ref="file" v-on:change="handleFileUpload()"/>
          <input disabled type="text"  class="form-control" :placeholder="title" v-model="fileName"/>
          <div class="input-group-append">
            <button class="btn btn-outline-secondary" @click="submitFile()" type="button" id="button-addon2">Upload</button>
          </div>
        </div>
      </div>
    </div>
</template>

<script>

export default {
  name: 'UploadInput',
  props: {
    title: String,
  },
  methods: {
    selectNode(peer){
      this.selectedHop = peer;
      this.destination = peer;
      this.toggleDropdown();
    },
    toggleDropdown(){
      this.dropdownDisplay = this.dropdownDisplay == 'none' ? 'block' : 'none'
    },
    addMessage(){
      if(this.newMessage){
        var data ={
          message: this.newMessage,
        }
        if (this.isprivate){
          data.destination = this.destination;
        }
        this.$emit('new-message', data)
        this.newMessage = ''
        this.destination = ''
      }
    },
    handleFileUpload(){
      this.file = this.$refs.file.files[0];
      this.fileName=this.file.name
    },
    toggleUpload(){
      var elem = this.$refs.file
      elem.click()
    },
    submitFile(){
      var data ={
          file: this.file,
        }
      this.$emit('upload-file', data)
    },
  },
  data(){
    return {
      file: '',
      fileName: ''
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

</style>
