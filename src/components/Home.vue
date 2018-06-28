<template>
  <v-app>
    <v-container>
      <div class="title mt-3">Select Namespace</div>
      <v-select :items="namespaces"
        item-text="metadata.name"
        item-value="metadata.name"
        v-model="namespace"
        @input="changeNamespace()">
      </v-select>

      <v-card class="my-3">
        <v-toolbar color="indigo" dark>
          <v-toolbar-title>Pods in {{ namespace }} namespace</v-toolbar-title>
          <v-spacer></v-spacer>
          <v-btn icon @click.stop="refreshPods()">
            <v-icon>refresh</v-icon>
          </v-btn>
        </v-toolbar>
        <v-card-title v-if="!pods.length" primary-title>
          <h3 class="subheading mb-0">{{ msg }}</h3>
        </v-card-title>
        <v-list v-else two-line>
          <template v-for="(pod, index) in pods" v-if="pod.status.phase === 'Running'">
            <v-divider v-if="index !== 0" :key="index"></v-divider>
            <v-subheader :key="pod.metadata.uid">{{ pod.metadata.name }}</v-subheader>
            <template v-for="container in pod.spec.containers">
              <v-list-tile :key="container.name" @click="selectPod(pod.metadata.name, container.name)">
                <v-list-tile-avatar>
                  <v-icon :class="'blue white--text'">alternate_email</v-icon>
                </v-list-tile-avatar>
                <v-list-tile-content>
                  <v-list-tile-title inset>{{ container.name }}</v-list-tile-title>
                  <v-list-tile-sub-title inset>{{ pod.status.phase }}</v-list-tile-sub-title>
                </v-list-tile-content>
                <v-list-tile-action>
                  <v-btn icon ripple>
                    <v-icon color="black lighten-1">exit_to_app</v-icon>
                  </v-btn>
                </v-list-tile-action>
              </v-list-tile>
            </template>
          </template>
        </v-list>
      </v-card>

      <v-dialog v-model="dialog" fullscreen transition="dialog-bottom-transition">
        <v-card>
          <v-toolbar dark color="primary">
            <v-toolbar-title>{{ container }} / {{ pod }}</v-toolbar-title>
            <v-spacer></v-spacer>
            <v-btn icon dark @click.native="closeDialog()">
              <v-icon>close</v-icon>
            </v-btn>
          </v-toolbar>
          <div style="width: 98%; margin: 20px 0 0 1%;">
            <console v-if="pod"
              :namespace="namespace"
              :pod="pod"
              :container="container"></console>
          </div>
        </v-card>
      </v-dialog>
    </v-container>
  </v-app>
</template>

<script>
import axios from 'axios'
import Console from './Console'

export default {
  name: 'Home',
  components: {
    Console
  },
  data () {
    return {
      msg: 'No resources found.',
      dialog: false,
      namespace: 'default',
      namespaces: [],
      pods: [],
      pod: '',
      container: ''
    }
  },
  created () {
    this.getNamespace()
    this.getPod(this.namespace)
  },
  methods: {
    selectPod (pod, container) {
      this.dialog = true
      this.pod = pod
      this.container = container
    },
    closeDialog () {
      this.pod = ''
      this.dialog = false
    },
    changeNamespace () {
      this.getPod(this.namespace)
    },
    refreshPods () {
      this.getPod(this.namespace)
    },
    getNamespace () {
      var namespaceAPI = 'api/v1/namespaces'
      axios.get(namespaceAPI).then(res => {
        this.namespaces = res.data.items
      }).catch(err => {
        console.log(err)
      })
    },
    getPod (namespace) {
      var podAPI = `api/v1/namespaces/${namespace}/pods`
      axios.get(podAPI).then(res => {
        this.pods = res.data.items
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
