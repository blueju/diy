<template>
  <!-- 配置区 -->
  <div id="ConfigArea">
    <!-- 配件 -->
    <section class="machineFitting">
      <a-card v-for="(item,index) in config" :key="index" :title="item.name">
        <a-button
          slot="extra"
          type="primary"
          shape="circle"
          icon="plus"
          @click="addConfig(index,item)"
        ></a-button>
        <a-tag
          color="#108ee9"
          v-for="(secondItem,index) in item.list"
          :key="index"
          @click="editConfig(index,item,secondItem)"
        >{{ secondItem.brand }} {{ secondItem.model }} {{ secondItem.price }}</a-tag>
      </a-card>
    </section>
    <!-- 配置弹窗 -->
    <section>
      <a-modal
        class="configDialog"
        :title="configDialogData.title"
        v-model="showConfigDialog"
        @ok="closeConfigDialog"
      >
        <!-- 表单行内块布局 Horizontal -->
        <a-form onSubmit>
          <a-form-item label="品牌">
            <a-input placeholder="input placeholder" />
          </a-form-item>
          <a-form-item label="型号">
            <a-input placeholder="input placeholder" />
          </a-form-item>
          <a-form-item label="价格">
            <a-input placeholder="input placeholder" />
          </a-form-item>
          <a-form-item label="链接">
            <a-input placeholder="input placeholder" />
          </a-form-item>
        </a-form>
      </a-modal>
    </section>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // 是否显示配置弹窗对话框
      showConfigDialog: false,
      // 配置
      config: [
        // CPU
        {
          name: "CPU",
          list: [
            {
              brand: "AMD324234",
              model: "2600X",
              price: "1000",
              purchaseLinks: "http://www.baidu.com"
            },
            {
              brand: "Intel",
              model: "9400F",
              price: "1299",
              purchaseLinks: "http://www.jd.com"
            }
          ]
        },
        // 主板
        {
          name: "主板",
          list: [
            {
              brand: "MSI",
              model: "B450M",
              price: "1299",
              purchaseLinks: "http://www.baidu.com"
            }
          ]
        }
      ],
      // 配置弹窗的配置数据
      configDialogData: {
        index: 0, // 默认为1，即CPU
        title: "",
        formData: {}
      }
    };
  },
  methods: {
    /**
     * 添加配置
     * index  索引
     * item
     */
    addConfig(index, item, secondItem) {
      // 打开配置弹窗对话框
      this.showConfigDialog = true;
      this.configDialogData.config = index;
      this.configDialogData.title = `添加 ${item.name} 配置`;
    },
    /**
     * 修改配置
     * index  索引
     * item
     */
    editConfig(index, item) {
      // 打开配置弹窗对话框
      this.showConfigDialog = true;
      this.configDialogData.config = index;
      this.configDialogData.title = `编辑 ${item.name} 配置`;
    },
    // 关闭配置弹窗对话框
    closeConfigDialog() {
      this.showConfigDialog = false;
    }
  }
};
</script>

<style lang="stylus">
#ConfigArea {
  padding: 10px;

  // 配件
  .machineFitting {
    // 配件卡片下外边距
    .ant-card {
      margin-bottom: 10px;
    }
  }

  // 配置弹窗
  .configDialog >>> .ant-form-item {
    margin-bottom: 10px;
  }
}
</style>
