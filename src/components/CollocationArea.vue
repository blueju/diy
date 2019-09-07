<template>
  <!-- 搭配区 -->
  <div id="CollocationArea">
    <a-card title="搭配区">
      <section id="cpu">
        <h1>CPU</h1>
        <div class="layArea">
          <a-icon type="plus" />
        </div>
      </section>
      <section id="mainboard">
        <h1>主板</h1>
        <div class="layArea">
          <a-icon type="plus" />
        </div>
      </section>
    </a-card>
  </div>
</template>

<script>
export default {
  data() {
    return {};
  }
};
</script>

<style lang="stylus">
#CollocationArea {
  padding: 10px 10px 10px 0;
  // 每块装卸区的距下边距
  section {
    margin-bottom: 10px;
  }
  // 装卸区
  .layArea {
    width: 100%;
    height: 60px;
    border: 1px dashed #c1c1c1;
    position: relative;
    // 装卸区图标
    .anticon {
      // 上下左右居中
      position: absolute;
      left: 50%;
      top: 50%;
      transform: translate(-50%, -50%);
      // 颜色大小
      font-size: 24px;
      color: #c1c1c1;
    }
  }
}
</style>
