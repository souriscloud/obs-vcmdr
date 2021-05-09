<template>
  <b-container>
    <b-row>
      <h1>OBS VCMDR</h1>
    </b-row>
    <b-row>
      <b-container v-if="!obsConnected">
        <b-row>
          <b-col>
            <b-form-input v-model="obsAddress" placeholder="localhost:4444" />
          </b-col>
          <b-col>
            <b-form-input v-model="obsPassword" placeholder="yourObsWebsocketPassword" />
          </b-col>
          <b-col>
            <b-button variant="success" @click="initConnection">Connect</b-button>
          </b-col>
        </b-row>
      </b-container>
      <p v-else>Connected to {{ obsAddress }}</p>
    </b-row>
    <b-container v-if="obsConnected">
      <b-row>
        <b-col>
          <b-button variant="danger" @click="destroyConnection">Disconnect</b-button>
        </b-col>
        <b-col>
          <b-button variant="info" @click="fetchScenes">Refresh scenes</b-button>
        </b-col>
      </b-row>
      <hr />

      <b-row>
        <b-col>
          <b-list-group>
            <div v-for="(scene, sceneIndex) in obsData.scenes" :key="sceneIndex">
              <b-list-group-item v-if="scene.name === obsData.currentSceneName" active button disabled>
                {{ scene.name }} [sources: {{ scene.sources.length }}]
              </b-list-group-item>
              <b-list-group-item v-else button @click="setScene(scene.name)">
                {{ scene.name }} [sources: {{ scene.sources.length }}]
              </b-list-group-item>
            </div>
          </b-list-group>
        </b-col>
        <b-col>
          <p>some info here</p>
        </b-col>
      </b-row>
    </b-container>
    <b-container v-else>
    </b-container>
  </b-container>
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
        AuthenticationSuccess: async data => {
          console.log('[Events]: AuthOK')
          this.obsConnected = true
          await this.fetchScenes()
        },
        AuthenticationFailure: data => {
          console.log('[Events]: AuthFail')
          console.log(data)
        },
        SwitchScenes: data => {
          console.log('[Events]: SwitchScenes =>', data.sceneName)
          this.obsData.currentSceneName = data.sceneName
        },
        error: err => {
          console.log('[Events]: ERROR')
          console.error(err)
        }
      },

      obsData: {
        scenes: [],
        currentSceneName: null
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
      this.obsData.scenes = []
      const response = await this.obsInstance.send('GetSceneList')
      if (response) {
        response.scenes.forEach(scene => {
          this.obsData.scenes.push(scene)
        })
        const current = await this.obsInstance.send('GetCurrentScene')
        if (current) {
          this.obsData.currentSceneName = current.name
        }
      }
    },

    async setScene (sceneName) {
      await this.obsInstance.send('SetCurrentScene', {
        'scene-name': sceneName
      })
    },

    destroyConnection () {
      console.log('# disconnect called')
      this.obsInstance.disconnect()
      this.obsInstance = null
    }
  },

  beforeDestroy () {
    this.destroyConnection()
  }
}
</script>

<style scoped>

</style>
