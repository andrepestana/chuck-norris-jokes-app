<template>
  <q-layout view="lHh Lpr lFf">
    <q-drawer
      v-model="leftDrawerOpen"
      show-if-above
      bordered
      content-class="bg-grey-1"
    >
      <q-list>
        <q-item-label
          header
          class="text-grey-8 text-h6"
        >
          Chuch Norris Jokes
        </q-item-label>
        <EssentialLink
          v-for="link in essentialLinks"
          :key="link.title"
          v-bind="link"
        />
      </q-list>
    </q-drawer>

    <q-page-container>
      <router-view />
    </q-page-container>
  </q-layout>
</template>

<script>
import EssentialLink from 'components/EssentialLink'

export default {
  name: 'MainLayout',

  components: {
    EssentialLink
  },
  data () {
    return {
      leftDrawerOpen: false,
      essentialLinks: [
        {
          title: 'Pay me a coffee',
          caption: 'if it made you smile',
          icon: 'local_cafe',
          link: 'https://andrepestana.github.io'
        },
        {
          title: 'About the Author',
          caption: '',
          icon: 'emoji_people',
          link: 'https://andrepestana.github.io'
        }
      ]
    }
  }
}
</script>
<style>
.q-header, .q-toolbar, .q-layout {
  background: transparent;
  color: black;
}
</style>