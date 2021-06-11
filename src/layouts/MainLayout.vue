<template>
  <q-layout view="lHh Lpr lFf">
    <q-header elevated>
      <q-toolbar>
        <q-toolbar-title>
          Locate Hub
        </q-toolbar-title>
        <q-btn flat @click="drawerRight = !drawerRight" round dense icon="menu" />
      </q-toolbar>
    </q-header>

    <q-page-container>
      <router-view />
    </q-page-container>
  </q-layout>
</template>

<script>

  export default {
    name: 'MainLayout',
    data() {
      return {
        drawerRight: false
      }
    }
  }
</script>
