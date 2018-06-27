<template>
  <div>
    <!-- <h2>All Pods in default namespaces</h3> -->
    <h3>
      All Pod in default namespaces:
      {{ msg }}
    </h3>
    <span>Connection ubuntu</span>
    <div class="container">
      <my-terminal></my-terminal>
    </div>
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
      msg: 'No resources found.'
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
          this.msg = pods
        }
      }).catch(err => {
        console.log(err)
      })
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1, h2 {
  font-weight: normal;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
