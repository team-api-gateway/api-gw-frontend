<template>
  <v-row justify="center" align="center">
    <v-col cols="12" sm="8" md="6">
      <v-card>
        <v-card-title class="headline">
          API Selector
        </v-card-title>
        <v-card-text>
          <div id="api-selector-app">
            <template
              v-for="api in apiList">
              <div :key="api.id">
                <input type="checkbox"/>
                <span>{{ api.name }}</span>
              </div>
            </template>
          </div>
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
  data() {
    return {
      apiList: [
        { id: 0, name: 'User Management API' },
        { id: 1, name: 'Customer API' },
        { id: 2, name: 'Project API' }
      ]
    }
  }
}
</script>
