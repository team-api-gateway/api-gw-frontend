<template>
  <v-row justify="center" align="center">
    <v-col cols="auto" sm="8" md="10">
      <v-card>
        <v-card-title class="headline"> API-Specifications </v-card-title>
        <v-card-text>
          <v-expansion-panels id="api-selector-app">
            <template v-for="api in apiListLocalRepresentation">
              <v-expansion-panel :key="api.id" class="api-spec-expansion-panel">
                <v-expansion-panel-header>
                  <div class="api-spec-name">{{ api.title }}</div>
                </v-expansion-panel-header>
                <v-expansion-panel-content>
                  <div>
                    <p v-if="$fetchState.pending">Fetching api spec...</p>
                    <p v-else-if="$fetchState.error">An error occurred :(</p>
                    <div v-else>
                      <div class="endpoint-name">Endpoints</div>
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
                              <div v-if="endpoint.parameters.length == 0">
                                Endpoint has no Parameters
                              </div>
                              <div v-else>Parameters:</div>
                              <template
                                v-for="parameter in endpoint.parameters"
                              >
                                <v-textarea
                                  :key="parameter.name"
                                  v-model="parameter.description"
                                  :label="parameter.name"
                                  rows="1"
                                  auto-grow
                                  required
                                ></v-textarea>
                              </template>
                            </v-expansion-panel-content>
                          </v-expansion-panel>
                        </template>
                      </v-expansion-panels>
                    </div>
                  </div>
                  <v-card-actions>
                    <v-spacer />
                    <v-tooltip bottom>
                      <template v-slot:activator="{ on, attrs }">
                        <v-btn
                          class="endpoint-button"
                          color="secondary"
                          v-bind="attrs"
                          v-on="on"
                          @click="fetchApiDetails(api)"
                        >
                          Refresh
                        </v-btn>
                      </template>
                      <span>Fetches this API from the server again.</span>
                    </v-tooltip>
                    <v-tooltip bottom>
                      <template v-slot:activator="{ on, attrs }">
                        <v-btn
                          class="endpoint-button"
                          color="secondary"
                          v-bind="attrs"
                          v-on="on"
                          @click="revert(api)"
                        >
                          Revert
                        </v-btn>
                      </template>
                      <span>
                        Overwrites all your changes on this API with the cached
                        values last fetched from the server.
                      </span>
                    </v-tooltip>
                    <v-tooltip bottom>
                      <template v-slot:activator="{ on, attrs }">
                        <v-btn
                          class="endpoint-button"
                          color="primary"
                          v-bind="attrs"
                          v-on="on"
                          @click="submit(api)"
                        >
                          Save
                        </v-btn>
                      </template>
                      <span>
                        Sends all changes you made on this API to the server and
                        saves them.
                      </span>
                    </v-tooltip>

                    <v-dialog v-model="dialog[api.id]" width="800">
                      <template v-slot:activator="{ on, attrs }">
                        <v-tooltip bottom>
                          <template v-slot:activator="{ on: tooltip }">
                            <v-btn
                              class="endpoint-button"
                              color="primary"
                              dark
                              v-bind="attrs"
                              v-on="{ ...tooltip, ...on }"
                            >
                              Publish API
                            </v-btn>
                          </template>
                          <span>
                            Opens a dialog, that allows you to publish the API
                            on the API Gateway.
                          </span>
                        </v-tooltip>
                      </template>

                      <v-card>
                        <v-card-title class="text-h5 grey darken-2">
                          {{ api.title }}: Selected Endpoints
                        </v-card-title>
                        <v-list two-line>
                          <template v-for="endpoint in api.endpoints">
                            <template v-if="endpoint.checked">
                              <v-list-item :key="getLabel(endpoint)">
                                <v-list-item-content>
                                  <v-row>
                                    <v-col cols="4">{{
                                      getLabel(endpoint)
                                    }}</v-col>
                                    <v-col
                                      ><div>
                                        <strong>Description: </strong>
                                        <div>{{ endpoint.description }}</div>
                                      </div>
                                      <div>
                                        <div
                                          v-if="endpoint.parameters.length > 0"
                                        >
                                          Parameters:
                                        </div>
                                        <ul>
                                          <li
                                            v-for="parameter in endpoint.parameters"
                                            :key="parameter.name"
                                          >
                                            <div>
                                              <strong
                                                >{{ parameter.name }}:
                                              </strong>
                                              {{ parameter.description }}
                                            </div>
                                          </li>
                                        </ul>
                                      </div></v-col
                                    >
                                  </v-row>
                                </v-list-item-content>
                              </v-list-item>
                            </template>
                          </template>
                        </v-list>

                        <v-divider></v-divider>

                        <v-card-actions>
                          <v-spacer></v-spacer>
                          <v-btn text @click="dialog[api.id] = false">
                            Cancel
                          </v-btn>
                          <v-btn color="primary" text @click="publish(api)">
                            Save and Publish
                          </v-btn>
                        </v-card-actions>
                      </v-card>
                    </v-dialog>
                  </v-card-actions>
                </v-expansion-panel-content>
              </v-expansion-panel>
            </template>
          </v-expansion-panels>
        </v-card-text>
      </v-card>
    </v-col>
  </v-row>
</template>
<script lang="ts">
export default {
  async asyncData(context: any) {
    const apiList = (await context.$axios.get('/apis')).data
    return {
      apiList,
      jwt_decoded: context.app.$auth.$storage.getUniversal('jwt_decoded'),
    }
  },
  data() {
    return {
      apiList: [],
      apiListLocalRepresentation: [],
      dialog: {},
      jwt_decoded: undefined,
    }
  },
  async fetch() {
    for (const api of this.apiList) {
      api.data = (await this.$axios.get('/apis/' + api.id)).data
    }
    for (const api of this.apiList) {
      const endpoints: any[] = []
      Object.entries(api.data.spec.paths).forEach((entry) => {
        const [pathKey, value]: [string, any] = entry
        Object.entries(value).forEach((entry2) => {
          const [methodKey, value2]: [string, any] = entry2
          const endpointLocalRepresentation: any = {
            path: pathKey,
            method: methodKey,
            description:
              value2.description === undefined ? '' : value2.description,
            parameters:
              value2.parameters === undefined
                ? []
                : JSON.parse(JSON.stringify(value2.parameters)),
            checked: api.data.selections.find((selection: any) => {
              return (
                selection.path.toLowerCase() === pathKey.toLowerCase() &&
                selection.method.toLowerCase() === methodKey.toLowerCase()
              )
            }).selected,
            modified: false,
          }
          endpoints.push(endpointLocalRepresentation)
        })
      })
      this.apiListLocalRepresentation.push({
        title: api.title,
        id: api.id,
        endpoints,
      })
    }
  },
  methods: {
    checkEndpoint(event) {
      event.cancelBubble = true
    },
    getLabel(selection) {
      return selection.method.toUpperCase() + ' ' + selection.path
    },
    async fetchApiDetails(apiLocalRepresentation) {
      const api = this.apiList.find(
        (api) => api.id === apiLocalRepresentation.id
      )
      api.data = (
        await this.$axios.get('/apis/' + apiLocalRepresentation.id)
      ).data
      this.revert(apiLocalRepresentation)
    },
    revert(apiLocalRepresentation) {
      const api = this.apiList.find(
        (api) => api.id === apiLocalRepresentation.id
      )
      for (const endpointIndex in apiLocalRepresentation.endpoints) {
        const endpoint = apiLocalRepresentation.endpoints[endpointIndex]
        const value2 = api.data.spec.paths[endpoint.path][endpoint.method]
        endpoint.description =
          value2.description === undefined ? '' : value2.description
        endpoint.parameters =
          value2.parameters === undefined
            ? []
            : JSON.parse(JSON.stringify(value2.parameters))
        endpoint.checked = api.data.selections.find((selection: any) => {
          return (
            selection.path.toLowerCase() === endpoint.path.toLowerCase() &&
            selection.method.toLowerCase() === endpoint.method.toLowerCase()
          )
        }).selected
        endpoint.modified = false
      }
    },
    submit(api) {
      const selections: any[] = []
      const requestBody = {
        id: api.id,
        spec: {
          paths: {},
        },
        selections,
        username: this.jwt_decoded.name,
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
          parameters: endpoint.parameters,
        }
      }
      console.log(requestBody)
      this.$axios.post('/apis/' + api.id + '/update', requestBody)
    },
    publish(api) {
      this.dialog[api.id] = false
      this.submit(api)
      this.$axios.put('/apis/' + api.id + '/spec')
    },
  },
}
</script>
