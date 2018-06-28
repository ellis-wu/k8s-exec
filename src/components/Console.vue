<template>
  <div id="terminal"></div>
</template>
<script>
import Terminal from './Xterm'
import 'xterm/dist/xterm.css'

export default {
  name: 'Console',
  props: {
    namespace: {
      type: String,
      default: ''
    },
    pod: {
      type: String,
      default: ''
    },
    container: {
      type: String,
      default: ''
    }
  },
  data () {
    return {
      terminalSocket: null
    }
  },
  mounted () {
    const URI = require('urijs')
    const path = URI(`wss://172.22.132.40:6443/api/v1/namespaces/${this.namespace}/pods/${this.pod}/exec`)
    path.addQuery('container', this.container)
    path.addQuery('command', 'sh')
    path.addQuery('stdout', 1)
    path.addQuery('stdin', 1)
    path.addQuery('stderr', 1)
    path.addQuery('tty', 1)

    let ws = new WebSocket(path.toString(), 'v4.channel.k8s.io')
    this.terminalSocket = ws
    var xterm = new Terminal({
      col: 120,
      row: 24,
      fontSize: 14,
      cursorBlink: 5,
      scrollback: 30,
      tabStopWidth: 8
    })
    xterm.open(document.getElementById('terminal'))

    ws.onerror = function () {
      console.log('WebSocket Error')
    }

    ws.onmessage = function (msg) {
      var reader = new FileReader()
      reader.readAsText(msg.data.slice(1))
      reader.onload = function () {
        xterm.write(reader.result)
      }
    }

    ws.onopen = function () {
      console.log('WebSocket Open')
    }

    ws.onclose = function () {
      console.log('WebSocket Close')
    }

    xterm.on('data', function (data) {
      var buf = new ArrayBuffer(data.length + 1)
      var bufView = new Uint8Array(buf)
      bufView[0] = '0'
      for (var i = 0; i < data.length; i++) {
        bufView[i + 1] = data.charCodeAt(i)
      }
      ws.send(buf)
    })
  },
  beforeDestroy () {
    var buf = new ArrayBuffer('exit'.length + 2)
    var bufView = new Uint8Array(buf)
    bufView[0] = '0'
    for (var i = 0; i < 'exit'.length; i++) {
      bufView[i + 1] = 'exit'.charCodeAt(i)
    }
    bufView[5] = '13'
    this.terminalSocket.send(buf)
  }
}
</script>
