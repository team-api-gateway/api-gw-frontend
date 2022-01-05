<template>
  <div class="p-6 justify-center text-center overflow-auto">
    <logo />
    <div v-if="!$auth.loggedIn">
      <h1 class="text-4xl">API Gateway Frontend</h1>
      <a class="tracking-tight cursor-pointer" @click="$auth.loginWith('aad')"
        >Sign in with Microsoft</a
      >
    </div>
    <div v-else>
      <h1 class="text-4xl">
        Hi, {{ jwt_decoded.given_name }} {{ jwt_decoded.family_name }}
      </h1>
      <hr class="m-6" />
      <div>email: {{ jwt_decoded.email }}</div>
      <div>oid: {{ jwt_decoded.oid }}</div>
    </div>
  </div>
</template>

<script lang="ts">
import Logo from '~/components/Logo.vue'

export default {
  components: {
    Logo,
  },
  asyncData(context: any) {
    return {
      jwt_decoded: context.app.$auth.$storage.getUniversal('jwt_decoded'),
    }
  },
  data() {
    return { jwt_decoded: null }
  },
}
</script>
