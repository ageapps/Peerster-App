<template>
  <div id="app" class="container-fluid">
    <h1 class="title">Peerster UI</h1>
    <transition appear name="fade" mode="out-in">
      <div v-if="!started" id="dashboard" class="col-md-8 col-sm-12 container-fluid">
        <PeerForm @create-peer="startPeer"/>
      </div>
      <div v-if="started" id="dashboard" class="container-fluid">
        <p>Peer: {{name}}</p>
        <p>Address: {{address}}</p>
        <button type="button" class="btn btn-outline-danger" @click="deletePeer" >Delete this Peer</button>
        <div class="row">
          <div class="col-md-6 col-sm-12">
            <NewMessageInput title="Send Message" @new-message="onNewMessage" />
            <NewMessageInput title="Send Private Message" v-bind:peers="hops" isprivate @new-message="onNewMessage" />
            <NewRequestInput title="Request MetaHash" v-bind:peers="hops" @new-request="onNewRequest" />
            <UploadInput title="Upload File" @upload-file="onUpload" />
            <PeerList v-bind:peers="nodes"  title="Peers connected"/>
            <MessageList v-bind:messages="messages"  title="Messages"/>
          </div>
          <div class="col-md-6 col-sm-12">
            <SimpleInput title="Add new peer" submittext="Add" @new-input="onNewPeer"/>
            <SimpleInput title="Keyboards: txt,file..." submittext="Search" @new-input="onNewSearch"/>
            <FileList v-bind:files="files" v-bind:baseurl="baseurl"  title="Files" @request-file="onNewSearch"/>
            <HopsList v-bind:peers="hops"  title="Hops"/>
            <PrivateList v-bind:messages="privateMesages"  title="Private Messages"/>
          </div>
        </div>
      </div>
    </transition>
    <transition name="fade">
      <div v-if="alerting" class="alert alert-danger" role="alert">
        {{alertText}}
      </div>
    </transition>
  </div>
</template>

<script>
import HelloWorld from './components/HelloWorld.vue'
import NewMessageInput from './components/NewMessageInput.vue'
import NewRequestInput from './components/NewRequestInput.vue'
import UploadInput from './components/UploadInput.vue'
import SimpleInput from './components/SimpleInput.vue'
import PeerList from './components/PeerList.vue'
import HopsList from './components/HopsList.vue'
import FileList from './components/FileList.vue'
import PrivateList from './components/PrivateList.vue'
import MessageList from './components/MessageList.vue'
import PeerForm from './components/PeerForm.vue'
import axios from 'axios';


var BACKEND_URL = "http://localhost:8080"
var REFRESH_PERIOD = 5
export default {
  name: 'app',
  components: {
    HelloWorld,
    NewMessageInput,
    NewRequestInput,
    UploadInput,
    SimpleInput,
    PeerList,
    HopsList,
    FileList,
    PrivateList,
    MessageList,
    PeerForm,
  },
  data() {
    return {
      name: 'nodeA',
      address: '0.0.0.0:0000',
      started: false,
      messages: [],
      nodes: [],
      files: {},
      hops: {},
      privateMesages: {},
      loading: false,
      alerting: false,
      alertText: "",
      messageTimerID: "",
      peerTimerID: "",
      hopsTimerID: "",
      privateTimerID: "",
      filesTimerID: "",
      baseurl: BACKEND_URL,
    }
  },
  methods: {
    onNewPeer(data){
      // console.log("New Peer: " + data)
       var params = {
        name: this.name,
        node: data
      }
      axios.post(BACKEND_URL+"/node", params)
      .then((response)  =>  {
        this.loading = false;
        this.peers = response.data;
      }, (error)  =>  {
        this.loading = false;
        this.showAlert(error.message);
        return
      })      
      this.peers.push(data)
    },
    onNewSearch(data){
      // console.log("New Peer: " + data)
       var params = {
        name: this.name,
        search: data
      }
      axios.post(BACKEND_URL+"/search", params)
      .then(()  =>  {
        this.loading = false;
      }, (error)  =>  {
        this.loading = false;
        this.showAlert(error.message);
        return
      })      
    },
    onUpload(data){
      let formData = new FormData();
      formData.append('file', data.file);
      formData.append('name', this.name);
      axios.post( BACKEND_URL+ '/upload',formData,
      {
        headers: {
            'Content-Type': 'multipart/form-data'
        }
      }).then(() => {
        this.showAlert("File uploaded!")
      }).catch(() => {
        this.showAlert("Error uploading the file!")
      });
    },
    onNewMessage(data){
      // console.log("New Message: " + data)
      var url = BACKEND_URL+"/message";
      var params = {
        name: this.name,
        msg: data.message
      }
      if (data.destination){
        params.destination = data.destination
        url = BACKEND_URL+"/private";
      }
      this.loading = true;
      axios.post( url, params)
      .then((response)  =>  {
        this.loading = false;
        this.messages = response.data;
      }, (error)  =>  {
        this.loading = false;
        this.showAlert(error.message);
        return
      })      
    },
    onNewRequest(data){
      // console.log("New Message: " + data)
      var url = BACKEND_URL+"/request";
      var params = {
        name: this.name,
        file: data.fileName,
        destination: data.destination,
        hash: data.requestHash
      }
      this.loading = true;
      axios.post( url, params)
      .then(()  =>  {
        this.loading = false;
      }, (error)  =>  {
        this.loading = false;
        this.showAlert(error.message);
        return
      })      
    },
    startPeer(data){
      this.loading = true;
      var params = {
        name: data.name,
        address: data.address
      }
      if(data.peers && data.peers.length > 0){
        params.peers=data.peers.join(',')
      }
      axios.post(BACKEND_URL+"/start", params)
      .then((response)  =>  {
        this.loading = false;
        this.peers = data.peers
        this.name = response.data.name
        this.address = response.data.address
        this.started = true
      }, (error)  =>  {
        this.loading = false;
        this.showAlert(error.message);
        return
      })
      this.startGettingMessages();    
      this.startGettingPrivateMessages();    
      this.startGettingPeers();    
      this.startGettingHops();    
      this.startGettingFiles();    
    },
    deletePeer(){
      this.loading = true;
      var params = {
        name: this.name,
      }
      // console.log("deleting...")
      axios.post(BACKEND_URL+"/delete", params)
      .then(()  =>  {
        // console.log(response)
        this.exit()
      }, (error)  =>  {
        // console.log(error)
        this.loading = false;
        this.showAlert(error.message);
        return
      })
    },
    showAlert(text){
      this.alerting = true;
      this.alertText = text;
      setTimeout(() => {  
        this.alerting = false;
      },4 * 1000);
    },
    startGettingMessages(){
      this.messageTimerID = setInterval(() => {  
        this.getMessages()
      },REFRESH_PERIOD * 1000);
    },
    startGettingFiles(){
      this.filesTimerID = setInterval(() => {  
        this.getFiles()
      },REFRESH_PERIOD * 1000);
    },
    startGettingPrivateMessages(){
      this.privateTimerID = setInterval(() => {  
        this.getPrivateMessages()
      },REFRESH_PERIOD * 1000);
    },
    startGettingPeers(){
      this.peerTimerID = setInterval(() => {  
        this.getPeers()
      },REFRESH_PERIOD * 1000);
    },
    startGettingHops(){
      this.hopsTimerID = setInterval(() => {  
        this.getHops()
      },REFRESH_PERIOD * 1000);
    },
    getMessages(){
      var url = BACKEND_URL+"/message";
      this.getData(url, (data) => {
        if (data) {
          this.messages = data;
        }
      }); 
    },
    getPrivateMessages(){
      var url = BACKEND_URL+"/private";
      this.getData(url, (data) => {
        if (data) {
          // console.log(data)
          this.privateMesages = data;
        }
      }); 
    },
    getPeers(){
      var url = BACKEND_URL+"/node";
      this.getData(url, (data) => {
        if (data) {
          this.nodes = data;
        }
      });
    },
    getHops(){
      var url = BACKEND_URL+"/routes";
      this.getData(url, (data) => {
        if (data) {
          this.hops = data;
        }
      });
    },
    getFiles(){
      var url = BACKEND_URL+"/files";
      this.getData(url, (data) => {
        if (data) {
          this.files = data;
        }
      });
    },
    getData(url, cb){
      var params = {
        name: this.name
      }
      this.loading = true;
      axios.get(url, {
        params:params
      }).then((response)  =>  {
        // console.log(response)
        this.loading = false;
        cb(response.data)
      }, (error)  =>  {
        this.loading = false;
        this.exit()
        this.showAlert(error.message);
        cb(null)
      });    
    },
    
    exit(){
        clearInterval(this.messageTimerID);
        clearInterval(this.peerTimerID);
        clearInterval(this.hopsTimerID);
        clearInterval(this.privateTimerID);
        clearInterval(this.filesTimerID);
        this.nodes = []
        this.files = {}
        this.messages = []
        this.hops = {}
        this.privateMesages = {}
        this.name = ""
        this.address = ""
        this.started = false
    }
  }
}
</script>

<style scoped>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  margin-top: 60px;
}
.title{
  text-align: center;
}
.btn{
  margin-bottom: 20px;
}
.fade-enter-active, .fade-leave-active {
  transition: opacity .5s;
}
.fade-enter, .fade-leave-to /* .fade-leave-active below version 2.1.8 */ {
  opacity: 0;
}
.alert{
  position: absolute;
  bottom: 0;
  left: 20px;
}
</style>
