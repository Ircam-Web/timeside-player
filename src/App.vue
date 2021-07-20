<template>
  <div id="app">
    <div class="header">
      <h1>Welcome to timeside-player</h1>
      <router-link
        :to="{ name: 'list' }"
      >
        Home
      </router-link>
      <SelectAPI />
      <button class="logout" @click="logout">
        Logout
      </button>
    </div>

    <router-view />
  </div>
</template>

<script lang="ts">
import { defineComponent, onMounted } from '@vue/composition-api'
import { useToasted } from '@/utils/vue-toasted'
import SelectAPI from '@/components/SelectAPI.vue'
import List from '@/components/List.vue'

import { persistentToken } from '@/utils/api'

export default defineComponent({
  name: 'App',
  components: {
    List,
    SelectAPI
  },
  setup () {
    const toasted = useToasted()
    onMounted(() => {
      toasted.success('🎉 Welcome to timeside-player !')
    })

    function logout () {
      persistentToken.removeToken()
      window.location.reload(false)
    }

    return {
      logout
    }
  }
})
</script>

<style lang="less" scoped>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
  margin-left: 50px;
}

#app ::v-deep {
  *, *::before, *::after {
    box-sizing: border-box;
  }
}
</style>
