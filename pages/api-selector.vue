<template>
  <v-row justify="center" align="center">
    <v-col cols="12" sm="8" md="6">
      <v-card>
        <v-card-title class="headline"> API Selector </v-card-title>
        <v-card-text>
          <v-expansion-panels id="api-selector-app">
            <template v-for="api in apiListLocalRepresentation">
              <v-expansion-panel :key="api.id">
                <v-expansion-panel-header>
                  {{ api.title }}
                </v-expansion-panel-header>
                <v-expansion-panel-content>
                  <div>
                    <p v-if="$fetchState.pending">Fetching api spec...</p>
                    <p v-else-if="$fetchState.error">An error occurred :(</p>
                    <div v-else>
                      Endpoints:<br />
                      <v-expansion-panels id="api-selector-app-endpoints">
                        <template v-for="endpoint in api.endpoints">
                          <v-expansion-panel :key="getLabel(endpoint)">
                            <v-expansion-panel-header>
                              <v-checkbox
                                :key="getLabel(endpoint)"
                                :label="getLabel(endpoint)"
                                v-model="endpoint.checked"
                                @click.native="checkEndpoint($event)"
                              ></v-checkbox>
                            </v-expansion-panel-header>
                            <v-expansion-panel-content>
                              <v-textarea
                                :key="getLabel(endpoint)"
                                v-model="endpoint.description"
                                label="description"
                                rows="1"
                                auto-grow
                                required
                              ></v-textarea>
                            </v-expansion-panel-content>
                          </v-expansion-panel>
                        </template>
                      </v-expansion-panels>
                    </div>
                  </div>
                  <v-card-actions>
                    <v-spacer />
                    <v-btn color="primary" @click="submit(api)">
                      Submit changes
                    </v-btn>
                  </v-card-actions>
                </v-expansion-panel-content>
              </v-expansion-panel>
            </template>
          </v-expansion-panels>
        </v-card-text>
        <v-card-actions>
          <v-spacer />
          <v-btn color="primary"> Confirm Selection </v-btn>
        </v-card-actions>
      </v-card>
    </v-col>
  </v-row>
</template>
<script>
export default {
  async asyncData({ $axios }) {
    const apiList = await $axios.$get('http://localhost:8082/apis')
    return { apiList }
  },
  data() {
    return {
      apiList: [],
      apiListLocalRepresentation: [],
      dialog: false,
    }
  },
  async fetch() {
    for (const api of this.apiList) {
      api.data = await fetch('http://localhost:8082/apis/' + api.id).then(
        (res) => res.json()
      )
    }
    for (const api of this.apiList) {
      const apiLocalRepresentation = {
        title: api.title,
        id: api.id,
        endpoints: [],
      }
      Object.entries(api.data.spec.paths).forEach((entry) => {
        const [pathKey, value] = entry
        Object.entries(value).forEach((entry2) => {
          const [methodKey, value2] = entry2
          const endpointLocalRepresentation = {
            path: pathKey,
            method: methodKey,
            description:
              value2.description === undefined ? '' : value2.description,
            checked: false,
            modified: false,
          }
          apiLocalRepresentation.endpoints.push(endpointLocalRepresentation)
        })
      })
      this.apiListLocalRepresentation.push(apiLocalRepresentation)
    }
  },
  methods: {
    checkEndpoint(event) {
      event.cancelBubble = true
    },
    getLabel(selection) {
      return selection.method + ' ' + selection.path
    },
    submit(api) {
      const requestBody = {
        id: api.id,
        spec: {
          paths: {},
        },
        selections: [],
        username: 'todo',
      }
      for (const endpoint of api.endpoints) {
        requestBody.selections.push({
          path: endpoint.path,
          method: endpoint.method,
          selected: endpoint.checked,
        })
        if (!(endpoint.path in requestBody.spec.paths))
          requestBody.spec.paths[endpoint.path] = {}
        requestBody.spec.paths[endpoint.path][endpoint.method] = {
          description: endpoint.description,
        }
      }
      console.log(requestBody)
      this.$axios.post(
        'http://localhost:8082/apis/' + api.id + '/update',
        requestBody
      )
    },
  },
}
</script>
