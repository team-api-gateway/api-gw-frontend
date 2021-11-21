<template>
  <v-row justify="center" align="center">
    <v-col cols="12" sm="8" md="6">
      <v-card>
        <v-card-title class="headline">
          API Selector
        </v-card-title>
        <v-card-text>
          <v-expansion-panels id="api-selector-app">
            <template
              v-for="api in apiList">
              <v-expansion-panel :key="api.id">
                <v-expansion-panel-header  @click.native="expandApi(api)">
                  {{ api.title }}
                </v-expansion-panel-header>
                <v-expansion-panel-content>
                  <div>
                    <p v-if="$fetchState.pending">Fetching api spec...</p>
                    <p v-else-if="$fetchState.error">
                      An error occurred :(
                    </p>
                    <div v-else>
                      Endpoints:<br>
                      <template
                        v-for="selection in api.data.selections">
                        <v-checkbox
                          :label=getLabel(selection)
                          :value=selection.checked
                          :key=getLabel(selection)
                        ></v-checkbox>
                        {{ getDescription(api.data, selection) }}
                      </template>
                    </div>
                  </div>
                </v-expansion-panel-content>
              </v-expansion-panel>
            </template>
           </v-expansion-panels>
        </v-card-text>
        <v-card-actions>
          <v-spacer />
          <v-btn
            color="primary"
          >
            Confirm Selection
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-col>
  </v-row>
</template>
<script>
export default {
  async asyncData({ $axios }) {
    const apiList = await $axios
      .$get('http://localhost:8082/apis')
    return { apiList };
  },
  data() {
    return {
      apiList: []
    }
  },
  async fetch() {
    for (const api of this.apiList) {
      api.data = await fetch(
        'http://localhost:8082/apis/' + api.id
        ).then(res => res.json());
    }
  },
  methods: {
    checkApi(event) {
      event.cancelBubble = true;
    },
    async expandApi(api) {
      api.data = await this.$axios
        .$get('http://localhost:8082/apis/' + api.id);
    },
    getLabel(selection) {
      return selection.method + " " + selection.path;
    },
    getDescription(data, selection) {
      const description = data.spec.paths[selection.path][selection.method.toLowerCase()].description;
      if (typeof description !== 'undefined' && description !== null){
        return description;
      }
      return "no description";
    }
  }
}
</script>
