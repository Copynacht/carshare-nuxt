<template>
  <v-app>
    <v-navigation-drawer
      v-model="drawer"
      :mini-variant="miniVariant"
      :clipped="clipped"
      fixed
      app
    >
      <v-list>
        <v-list-item
          v-for="(item, i) in items"
          :key="i"
          :to="item.to"
          router
          exact
        >
          <v-list-item-action>
            <v-icon>{{ item.icon }}</v-icon>
          </v-list-item-action>
          <v-list-item-content>
            <v-list-item-title v-text="item.title" />
          </v-list-item-content>
        </v-list-item>
      </v-list>
    </v-navigation-drawer>
    <v-app-bar :clipped-left="clipped" fixed app>
      <v-app-bar-nav-icon @click.stop="drawer = !drawer" />
      <v-toolbar-title v-text="title" />
    </v-app-bar>
    <v-main>
      <v-container>
        <nuxt />
      </v-container>
    </v-main>
  </v-app>
</template>

<script>
export default {
  data() {
    return {
      clipped: false,
      drawer: false,
      fixed: false,
      items: [
        {
          icon: 'mdi-home',
          title: 'ホーム',
          to: '/',
        },
        {
          icon: 'mdi-calendar-clock',
          title: '予約',
          to: '/carlist',
        },
        {
          icon: 'mdi-face',
          title: 'ユーザー情報',
          to: '/mypage',
        },
        {
          icon: 'mdi-head',
          title: '管理者画面',
          to: '/admin/manage',
        },
        {
          icon: 'mdi-exit-run',
          title: 'ログアウト',
          to: '/signout',
        },
      ],
      miniVariant: false,
      right: true,
      rightDrawer: false,
      title: 'カーシェア予約',
    }
  },
}
</script>
