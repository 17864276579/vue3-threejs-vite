<template>
  <a-menu mode="horizontal" style="position: fixed;">
    <a-sub-menu key="demo">
      <template #title>路由</template>
      <a-menu-item key="1">
        <RouterLink to="/">主页</RouterLink>
      </a-menu-item>
      <a-menu-item key="2">
        <RouterLink to="/car">小车</RouterLink>
      </a-menu-item>
    </a-sub-menu>
  </a-menu>
  <RouterView> </RouterView>
</template>

<script setup lang="ts"></script>

<style>
html,
#app,
body {
  height: 100%;
  width: 100%;
  margin: 0;
  padding: 0;
}
</style>
