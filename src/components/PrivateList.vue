<template>
  <div class="card">
      <div class="card-header">
        {{title}}
      </div>
      <div class="card-body">    
        <ul class="list-group">
          <li class="list-group-item" v-for="(msgs,key) in messages" :key = "key">
            Origin: {{key}}
                    <ul class="list-group">
                      <li class="list-group-item" v-for="(msg,key) in msgs" :key = "key">
                          {{msg.Destination}} -> [{{msg.ID}}] {{msg.Text}}
                      </li>
                    </ul>
          </li>
        </ul>
      </div>
  </div>
</template>

<script>
export default {
  name: 'HopsList',
  props: {
    title: String,
    messages: Object,
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
