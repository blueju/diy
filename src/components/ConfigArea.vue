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
          v-for="(secondItem,secondIndex) in item.list"
          :key="secondIndex"
          @click="editConfig(index,secondIndex,item,secondItem)"
          draggable="true"
          @dragstart="dragstart($event,index,item,secondItem)"
        >{{ secondItem.brand }} {{ secondItem.model }} ￥{{ secondItem.price }}</a-tag>
      </a-card>
    </section>
    <!-- 配置弹窗 -->
    <section>
      <a-modal
        class="configDialog"
        :title="configDialogData.title"
        :maskClosable="false"
        :visible="showConfigDialog"
        @ok="submitConfigForm"
        @cancel="configDialogFormCancel"
      >
        <!-- 表单行内块布局 Horizontal -->
        <a-form onSubmit>
          <a-form-item label="品牌">
            <a-input v-model="configDialogData.formData.brand" />
          </a-form-item>
          <a-form-item label="型号">
            <a-input v-model="configDialogData.formData.model" />
          </a-form-item>
          <a-form-item label="价格">
            <a-input v-model="configDialogData.formData.price" />
          </a-form-item>
          <a-form-item label="链接">
            <a-input v-model="configDialogData.formData.purchaseLinks" />
          </a-form-item>
        </a-form>
      </a-modal>
    </section>
  </div>
</template>

<script>
import config from "./../../static/configAreaConfig";

export default {
  data() {
    return {
      // 是否显示配置弹窗对话框
      showConfigDialog: false,
      // 配置
      config: config,
      // 配置弹窗的配置数据
      configDialogData: {
        index: 0, // 一级索引，代表配件，默认为0，即CPU
        secondIndex: 0, // 二级索引，代表具体配件，默认为0
        type: "",
        title: "",
        formData: {
          brand: "",
          model: "",
          price: "",
          purchaseLinks: ""
        }
      }
    };
  },
  methods: {
    dragstart(e, index, item, secondItem) {
      // debugger;
      let draggedItem = {
        name: item.name,
        type: item.type,
        index: index,
        itemData: secondItem
      };
      e.dataTransfer.setData("draggedItem", JSON.stringify(draggedItem));
      // localStorage.setItem("dragData", JSON.stringify(dragData));
    },
    //start ,end ,add,update, sort, remove 得到的都差不多
    start: function(evt) {
      console.log(evt);
    },
    end: function(evt) {
      console.log(evt);
      evt.item; //可以知道拖动的本身
      evt.to; // 可以知道拖动的目标列表
      evt.from; // 可以知道之前的列表
      evt.oldIndex; // 可以知道拖动前的位置
      evt.newIndex; // 可以知道拖动后的位置
    },
    // move: function(evt, originalEvent) {
    //   console.log(evt);
    //   console.log(originalEvent); //鼠标位置
    // },
    move({ relatedContext, draggedContext }) {
      const relatedElement = relatedContext.element;
      const draggedElement = draggedContext.element;
      return (
        (!relatedElement || !relatedElement.fixed) && !draggedElement.fixed
      );
    },
    /**
     * 添加配置
     * index  索引
     * item
     */
    addConfig(index, item, secondItem) {
      // 打开配置弹窗对话框
      this.showConfigDialog = true;
      this.configDialogData.index = index;
      this.configDialogData.type = "add";
      this.configDialogData.title = `添加 ${item.name} 配置`;
    },
    /**
     * 修改配置
     * index  索引
     * item
     */
    editConfig(index, secondIndex, item, secondItem) {
      // 打开配置弹窗对话框
      this.showConfigDialog = true;
      this.configDialogData.index = index;
      this.configDialogData.secondIndex = secondIndex;
      this.configDialogData.type = "edit";
      this.configDialogData.title = `编辑 ${item.name} 配置`;
      this.configDialogData.formData = JSON.parse(JSON.stringify(secondItem));
    },
    // 提交配置弹窗表单
    submitConfigForm() {
      let index = this.configDialogData.index;
      let secondIndex = this.configDialogData.secondIndex;
      switch (this.configDialogData.type) {
        case "add":
          // 将配置表单数据添加到
          this.config[index].list.push(
            JSON.parse(JSON.stringify(this.configDialogData.formData))
          );
          break;
        case "edit":
          this.config[index].list[secondIndex] = JSON.parse(
            JSON.stringify(this.configDialogData.formData)
          );
          break;
      }

      // 清空配置弹窗表单
      for (let i in this.configDialogData.formData) {
        this.configDialogData.formData[i] = "";
      }
      this.showConfigDialog = false;
    },
    // 取消配置弹窗表单，清空
    configDialogFormCancel() {
      this.showConfigDialog = false;
      for (let i in this.configDialogData.formData) {
        this.configDialogData.formData[i] = "";
      }
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
