<template>
  <div>
    <h1>OBS Test</h1>
    <br />
    <p>Connecting to {{ obsAddress }} with password {{ obsPassword }}</p>
    <div v-if="obsConnected">
      <button @click="destroyConnection">Disconnect</button>
      <hr />

      <div class="obs-controls">
        <ul>
          <li v-for="(scene, sceneIndex) in obsData.scenes" :key="sceneIndex">
            {{ scene.name }} [sources: {{ scene.sources.length }}]
          </li>
        </ul>
      </div>
    </div>
    <div v-else>
      <button @click="initConnection">Connect</button>
    </div>
  </div>
</template>

<script>
import * as OBSWebSocket from 'obs-websocket-js'

export default {
  name: 'OBSTest',

  data () {
    return {
      obsInstance: null,
      obsAddress: 'localhost:8791',
      obsPassword: 'toor',
      obsConnected: false,

      events: {
        ConnectionOpened: data => {
          console.log('[Events]: ConnectionOpened')
        },
        ConnectionClosed: data => {
          console.log('[Events]: ConnectionClosed')
          this.obsConnected = false
        },
        AuthenticationSuccess: data => {
          console.log('[Events]: AuthOK')
          this.obsConnected = true
          this.fetchScenes()
        },
        AuthenticationFailure: data => {
          console.log('[Events]: AuthFail')
          console.log(data)
        },
        error: err => {
          console.log('[Events]: ERROR')
          console.error(err)
        }
      },

      obsData: {
        scenes: []
      }
    }
  },

  methods: {
    initConnection () {
      console.log('# connect called')
      this.obsInstance = new OBSWebSocket()
      this.setupEvents()
      this.obsInstance.connect({ address: this.obsAddress, password: this.obsPassword })
        .then(() => {
          console.log('# connected')
        })
        .catch(err => {
          console.log('# connection error')
          console.error(err)
        })
    },

    setupEvents () {
      console.log('# event setup called')
      Object.entries(this.events).forEach(([eventName, eventCallback]) => {
        this.obsInstance.on(eventName, eventCallback)
      })
    },

    async fetchScenes () {
      const response = await this.obsInstance.send('GetSceneList')
      if (response) {
        response.scenes.forEach(scene => {
          this.obsData.scenes.push(scene)
        })
      }
    },

    destroyConnection () {
      console.log('# disconnect called')
      this.obsInstance.disconnect()
      this.obsInstance = null
    }
  }
}
</script>

<style scoped>

</style>
