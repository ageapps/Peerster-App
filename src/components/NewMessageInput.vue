<template>
    <div class="card">
      <div class="card-body">
        <div class="input-group mb-3">
          <input type="text" class="form-control" placeholder="Mesage" aria-label="Recipient's username" aria-describedby="button-addon2"  v-model="newMessage">
          <div class="input-group-append">
            <button class="btn btn-outline-secondary" @click="addMessage" type="button" id="button-addon2">Send</button>
          </div>
        </div>
        <div v-show="isprivate" type="button">
          <input type="text" class="form-control" placeholder="Destination" aria-label="Recipient's destination" aria-describedby="button-addon2"  v-model="destination">
        </div>
        <div v-show="!isprivate" type="button">
          <input type="file" class="form-control" placeholder="File" aria-label="Recipient's file" aria-describedby="button-addon3"  v-on:change="uploadFile">
        </div>

      </div>
    </div>
</template>

<script>
export default {
  name: 'NewMessageInput',
  props: {
    title: String,
    peers: Array,
    isprivate: {
      type: Boolean,
      default: false
    }
  },
  methods: {
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
    uploadFile(){

    }
  },
  data(){
    return {
      newMessage: '',
      destination: ''
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

</style>
