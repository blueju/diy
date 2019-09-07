<template>
  <div id="app">
    <a-row>
      <!-- 配置区 -->
      <a-col :span="6">
        <ConfigArea />
      </a-col>
      <!-- 搭配区 -->
      <a-col :span="18">
        <CollocationArea />
      </a-col>
    </a-row>
  </div>
</template>

<script>
// 配置区
import ConfigArea from "./components/ConfigArea";
// 搭配区
import CollocationArea from "./components/CollocationArea";

export default {
  name: "App",
  components: {
    CollocationArea,
    ConfigArea
  }
};
</script>

<style>
#app {
  width: 100%;
  height: 100%;
  background-color: #f2f4f5;
}
</style>
