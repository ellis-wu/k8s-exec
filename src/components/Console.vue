<template>
  <div class="console" id="terminal"></div>
</template>
<script>
import Terminal from './Xterm'

export default {
  name: 'Console',
  data () {
    return {
      terminalSocket: null
    }
  },
  created () {
    // var uri = 'wss://172.20.3.38:6443/api/v1/namespaces/default/pods/nginx-8586cf59-lrxd4/exec?command=sh&container=nginx&container=nginx&stdin=true&stdout=true&tty=true'
    var uri = 'wss://172.20.3.38:6443/api/v1/namespaces/default/pods/ubuntu/exec?command=bash&container=ubuntu&container=ubuntu&stdin=true&stdout=true&tty=true'
    let ws = new WebSocket(uri, 'v4.channel.k8s.io')
    this.terminalSocket = ws
    var xterm = new Terminal({
      // cols: 100,
      // rows: 20,
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

    // if you websocket using bas64 protocol, you need attach key envent to send Enter
    // xterm.attachCustomKeyEventHandler(function (e) {
    //   if (e.keyCode === 13 && e.type === 'keypress') {
    //     // ws.send(0 + 'DQ==')
    //   }
    // })

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
    console.log('beforeDestroy')
    this.terminalSocket.close()
  }
}
</script>
