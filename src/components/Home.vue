<template>
  <div>
    <div class="pod_content">
      <div class="title">All Pod in default namespaces:</div>
      <div v-if="pods" v-for="(item, index) in pods" :key="index"> * {{ item }}</div>
      <div v-else>{{ msg }}</div>
    </div>
    <span class="title">Connection to ubuntu Pod</span>
    <my-terminal></my-terminal>
  </div>
</template>

<script>
import axios from 'axios'
import Console from './Console'

export default {
  name: 'Home',
  components: {
    'my-terminal': Console
  },
  data () {
    return {
      msg: 'No resources found.',
      pods: []
    }
  },
  created () {
    this.getPod()
  },
  methods: {
    getPod () {
      var pods = 'api/v1/namespaces/default/pods'
      axios.get(pods).then(res => {
        if (res.data.items.length === 0) {
          this.msg = 'No resources found.'
        } else {
          var pods = res.data.items.map(data => {
            return data.metadata.name
          })
          this.pods = pods
        }
      }).catch(err => {
        console.log(err)
      })
    }
  }
}
</script>

<style scoped>
.pod_content {
  margin: 10px 0;
}
.title {
  font-weight: bold;
  font-size: 1.17em;
}
</style>
