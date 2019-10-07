<template>
  <!-- 搭配区 -->
  <div id="CollocationArea">
    <a-card title="搭配区">
      <a-button
        slot="extra"
        type="primary"
        class="config-button config-button-save"
        @click="saveConfig"
      >保存配置</a-button>
      <a-button
        slot="extra"
        type="primary"
        class="config-button config-button-import"
        @click="importConfig"
      >导入配置</a-button>
      <a-button
        slot="extra"
        type="primary"
        class="config-button config-button-export"
        @click="exportConfig"
      >导出配置</a-button>
      <a-button
        slot="extra"
        type="primary"
        class="config-button config-button-reset"
        @click="removeConfig"
      >清空配置</a-button>
      <span slot="extra" class="total-price">￥ {{ totalPrice}}</span>
      <section id="item.type" v-for="(item,index) in fittings" :key="index">
        <h1>{{ item.name }}</h1>
        <div class="layArea" @dragover="dragover" @drop="drop">
          <!-- 添加 图标 -->
          <a-icon class="plus-icon" type="plus" v-show="!item.haveData" />
          <!-- 删除 图标 -->
          <a-icon
            class="delete-icon"
            type="delete"
            v-show="item.haveData"
            @click="deleteFittingsItem(item,index)"
          />
          <!-- 购物车图标 -->
          <a-icon
            v-show="item.haveData"
            class="shopping-cart-icon"
            type="shopping-cart"
            @click="goShopping(item.formData.purchaseLinks)"
          />
          <ul class="detail" v-show="item.formData.brand">
            <li class="brand">{{ item.formData.brand }}</li>
            <li class="model">{{ item.formData.model }}</li>
            <li class="price">￥{{ item.formData.price }}</li>
          </ul>
        </div>
      </section>
    </a-card>
  </div>
</template>

<script>
import fittingsConfig from "./../../static/collocationAreaConfig";

export default {
  data() {
    return {
      fittings: fittingsConfig
    };
  },
  computed: {
    totalPrice() {
      let totalPrice = 0;
      for (let i = 0; i < this.fittings.length; i++) {
        totalPrice = totalPrice + Number(this.fittings[i].formData.price);
      }
      return totalPrice;
    }
  },
  methods: {
    // 清空配置
    removeConfig() {
      let object = this.fittings;
      for (const key in object) {
        if (object.hasOwnProperty(key)) {
          object[key].haveData = false;
          for (const secondKey in object[key].formData) {
            if (object[key].formData.hasOwnProperty(secondKey)) {
              object[key].formData[secondKey] = "";
            }
          }
        }
      }
    },

    // 导入配置
    importConfig() {
      let importConfig = JSON.parse(localStorage.getItem("saveConfig"));
      if (this.isAllEmpty(importConfig)) {
        this.$message.success("无本地数据");
        return;
      }
      this.fittings = importConfig;
      this.$message.success("导入配置成功");
    },

    // 判断搭配配置是否为空
    isAllEmpty(data) {
      let isAllEmpty;
      for (const key in data) {
        if (data.hasOwnProperty(key)) {
          if (data[key].haveData) {
            isAllEmpty = false;
          } else {
            isAllEmpty = true;
          }
        }
        break;
      }
      return isAllEmpty;
    },

    // 导出配置
    exportConfig() {
      debugger;
      let needExportData = this.fittings;
      if (this.isAllEmpty(needExportData)) {
        this.$message.success("配置为空，请添加配置后再进行导出");
        return;
      }
      // 列标题
      let columnTitle = `<tr><td></td><td>品牌</td><td>型号</td><td>价格</td><td>商品链接</td></tr>`;
      let excelMainData = "";
      // 循环遍历，每行加入tr标签，每个单元格加td标签
      for (let i = 0; i < needExportData.length; i++) {
        excelMainData = excelMainData + "<tr>";
        for (let key in needExportData[i]) {
          if (needExportData[i].hasOwnProperty(key)) {
            excelMainData =
              excelMainData + `<td>${needExportData[i].name}</td>`;
            for (let secondKey in needExportData[i].formData) {
              if (needExportData[i].formData.hasOwnProperty(secondKey)) {
                //增加\t为了不让表格显示科学计数法或者其他格式
                excelMainData =
                  excelMainData +
                  `<td>${needExportData[i].formData[secondKey] + "\t"}</td>`;
              }
            }
          }
          break;
        }
        excelMainData = excelMainData + "</tr>";
      }
      let excel = columnTitle + excelMainData;

      //Worksheet名
      let worksheet = "电脑配置清单";
      let url = "data:application/vnd.ms-excel;base64,";

      //下载的表格模板数据
      let template = `<html xmlns:o="urn:schemas-microsoft-com:office:office" 
      xmlns:x="urn:schemas-microsoft-com:office:excel" 
      xmlns="http://www.w3.org/TR/REC-html40">
      <head><!--[if gte mso 9]><xml><x:ExcelWorkbook><x:ExcelWorksheets><x:ExcelWorksheet>
        <x:Name>${worksheet}</x:Name>
        <x:WorksheetOptions><x:DisplayGridlines/></x:WorksheetOptions></x:ExcelWorksheet>
        </x:ExcelWorksheets></x:ExcelWorkbook></xml><![endif]-->
        </head><body><table>${excel}</table></body></html>`;
      //下载模板
      // window.location.href = url + base64(template);
      window.location.href =
        url + window.btoa(unescape(encodeURIComponent(template)));
    },

    // 保存配置
    saveConfig() {
      if (this.isAllEmpty(this.fittings)) {
        this.$message.success("配置为空，请添加配置后再进行保存");
        return;
      }
      // 以哪种形式保存？浏览器localStorage保存：文件保存
      let _this = this;
      this.$confirm({
        closable: true,
        title: "请选择保存方式？",
        okText: "配置文件下载保存",
        cancelText: "浏览器本地保存",
        onOk() {
          let downloadButton = document.createElement("a");
          let blob = new Blob([JSON.stringify(_this.fittings)]);
          downloadButton.href = window.URL.createObjectURL(blob);
          downloadButton.download =
            "电脑清单配置文件" + new Date().toLocaleString() + ".json";
          downloadButton.click();
          downloadButton = null;
        },
        onCancel() {
          // 存入localStorage
          localStorage.setItem("saveConfig", JSON.stringify(_this.fittings));
          _this.$message.success("保存配置成功");
        }
      });
    },

    // 点击删除按钮，删除此项配件项
    deleteFittingsItem(item, index) {
      for (let key in item.formData) {
        if (item.formData.hasOwnProperty(key)) {
          item.formData[key] = "";
        }
      }
      this.fittings[index].haveData = false;

      document.getElementsByClassName("plus-icon")[index].style.display =
        "block";
      // 实线变成虚线
      document.getElementsByClassName("layArea")[index].style.border =
        "1px dashed #c1c1c1";
    },
    // 点击购物车图标，前往购物链接
    goShopping(link) {
      window.open(link);
    },
    dragover(e) {
      e.preventDefault();
    },
    drop(e) {
      e.preventDefault();
      // let dragData = JSON.parse(localStorage.getItem("dragData"));
      let draggedItem = JSON.parse(e.dataTransfer.getData("draggedItem"));
      this.fittings[draggedItem.index].formData = JSON.parse(
        JSON.stringify(draggedItem.itemData)
      );
      this.fittings[draggedItem.index].haveData = true;
      // 虚线变成实线
      document.getElementsByClassName("layArea")[
        draggedItem.index
      ].style.border = "1px solid #c1c1c1";
      document.getElementsByClassName("plus-icon")[
        draggedItem.index
      ].style.display = "none";
      // let itemData = e.dataTransfer.getData(dragData.itemName.toLowerCase());
      // e.target.appendChild(document.getElementById(draggedElementId));
    }
  }
};
</script>

<style lang="stylus">
#CollocationArea {
  padding: 10px 10px 10px 0;

  .config-button {
    margin-right: 16px;
  }

  // 保存配置按钮
  .config-button-save {
    // 使用默认的蓝色
  }

  // 导入配置按钮
  .config-button-import {
    background-color: #13c2c2;
    border-color: #13c2c2;
  }

  // 导出配置按钮
  .config-button-export {
    background-color: #52c41a;
    border-color: #52c41a;
  }

  // 清空配置按钮
  .config-button-reset {
    background-color: #ff4d4f;
    border-color: #ff4d4f;
  }

  // 总价
  .total-price {
    font-size: 18px;
    font-weight: bold;
    color: #F56C6C;
  }

  .detail {
    list-style: none;
    margin: 0;
    padding: 0;
    display: flex;
    flex-direction: row;
    justify-content: space-around;
    line-height: 60px;

    li {
      font-size: 22px;
      color: #606266;
    }

    .price {
      color: #F56C6C;
    }
  }

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

    // 装卸区 添加 图标
    .plus-icon {
      // 上下左右居中
      position: absolute;
      left: 50%;
      top: 50%;
      transform: translate(-50%, -50%);
      // 颜色大小
      font-size: 24px;
      color: #c1c1c1;
    }

    // 装卸区 删除 图标
    .delete-icon {
      // 上下左右居中
      position: absolute;
      right: 4px;
      top: 4px;
      // 颜色大小
      font-size: 16px;
      color: #c1c1c1;

      // 鼠标悬浮时样式
      &:hover {
        cursor: pointer;
      }
    }

    // 装卸区 购物车 图标
    .shopping-cart-icon {
      // 上下左右居中
      position: absolute;
      right: 4px;
      top: calc(4px + 16px + 4px);
      // 颜色大小
      font-size: 16px;
      color: #c1c1c1;

      // 鼠标悬浮时样式
      &:hover {
        cursor: pointer;
      }
    }
  }
}
</style>
