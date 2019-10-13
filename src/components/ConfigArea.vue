<template>
  <!-- 配置区 -->
  <div id="ConfigArea">
    <!-- 配件 -->
    <section class="machineFitting">
      <a-card title="硬件配置区">
        <a-button @click="saveHardwareConfigList">保存硬件配置</a-button>
        <a-button @click="triggerFileUploadDialog">导入硬件配置</a-button>
        <input
          type="file"
          id="importHardwareConfig"
          @change="importHardwareConfigList"
          style="display:none"
        />
      </a-card>
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
          @contextmenu="showDeleteMenu"
          :data-index="index"
          :data-second-index="secondIndex"
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
            <a-input-number v-model="configDialogData.formData.price" :min="1" />
          </a-form-item>
          <a-form-item label="链接">
            <a-input v-model="configDialogData.formData.purchaseLinks" />
          </a-form-item>
        </a-form>
      </a-modal>
    </section>
    <!-- 删除配件右键菜单 -->
    <section class="deleteMenu">
      <a-button type="danger" @click="deleteMachineFitting">删除此配件</a-button>
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
      },
      // 要删除的配件的事件
      needDeleteItem: null
    };
  },
  methods: {
    /**
     * 保存硬件配置列表
     */
    saveHardwareConfigList() {
      let ele = document.createElement("a");
      let blob = new Blob([JSON.stringify(this.config)]);
      let url = URL.createObjectURL(blob);
      ele.href = url;
      ele.download = "硬件配置" + new Date().toLocaleString() + ".diy_hardware";
      ele.click();
      ele = null;
      URL.revokeObjectURL(url);
    },
    /**
     * 触发文件上传弹窗
     */
    triggerFileUploadDialog() {
      document.getElementById("importHardwareConfig").click();
    },
    /**
     * 导入硬件配置信息列表
     */
    importHardwareConfigList() {
      let file = document.getElementById("importHardwareConfig").files[0];
      let fileReader = new FileReader();
      fileReader.readAsText(file);
      let _this = this;
      fileReader.onload = function() {
        _this.config = JSON.parse(this.result);
      };
    },
    // 显示删除菜单
    showDeleteMenu(e) {
      e.preventDefault();
      let menu = document.getElementsByClassName("deleteMenu")[0];
      menu.style.display = "block";
      menu.style.left = e.clientX + "px";
      menu.style.top = e.clientY + "px";
      this.needDeleteItem = e;
    },
    // 删除配件
    deleteMachineFitting() {
      let index = this.needDeleteItem.target.dataset.index;
      let secondIndex = this.needDeleteItem.target.dataset.secondIndex;
      this.config[index].list.splice(secondIndex, 1);
      document.getElementsByClassName("deleteMenu")[0].style.display = "none";
    },
    dragstart(e, index, item, secondItem) {
      let draggedItem = {
        name: item.name,
        type: item.type,
        index: index,
        itemData: secondItem
      };
      e.dataTransfer.setData("draggedItem", JSON.stringify(draggedItem));
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

  // 删除配件
  .deleteMenu {
    display: none;
    position: absolute;
  }
}
</style>
