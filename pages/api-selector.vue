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
                  <v-checkbox
                    :label=api.title
                    @click.native="checkApi($event)"
                  ></v-checkbox>
                </v-expansion-panel-header>
                <v-expansion-panel-content>
                  <!--todo: somehow display api.data (once it is loaded)-->
                  Endpoints
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
      .$get('http://localhost:8080/apis')
    return { apiList };
  },
  data() {
    return {
      apiList: []
    }
  },
  methods: {
    checkApi(event) {
      event.cancelBubble = true;
    },
    async expandApi(api) {
      api.data = await this.$axios
        .$get('http://localhost:8080/apis/' + api.id);
    }
  }
}
</script>
