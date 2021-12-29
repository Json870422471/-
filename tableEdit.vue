<template>
  <div class="table-container" v-if="flag">
    <!--:max-height="fixedHeight ? tableHeight : ''"-->
    <el-table
      ref="listTable"
      v-loading="loading"
      :data="tableData"
      :border="border"
      :stripe="stripe"
      :max-height="fixedHeight ? tableHeight : ''"
      size="mini"
      :empty-text="emptyText"
      style="width: 100%"
      :class="selectionLimit > -1 ? 'table-container__selection-limit' : ''"
      :header-cell-style="tableHeaderStyle"
      @select="handleSelect"
      @cell-click="rowClick"
      :row-class-name="tableRowClassName"
      @select-all="handleSelectAll"
      @selection-change="handleSelectionChange"
    >
      <el-table-column
        v-if="selection"
        type="selection"
        width="40"
        align="center"
      >
      </el-table-column>
      <template>
        <el-table-column :width="40" align="center" type="index" label="序号">
          <template slot-scope="scope">
            <!-- <img
              v-if="scope.$index === tableData.length - 1"
              src="./img/icon-add.png"
              alt=""
              @click="handleAddTableRow(scope)"
              style="cursor: pointer"
            /> -->

            <div>{{ scope.$index + 1 }}</div>
            <!-- <div v-else>{{ scope.$index + 1 }}</div> -->
          </template>
        </el-table-column>
        <el-table-column
          v-for="col in columns"
          :key="col.label"
          :render-header="(v, c) => modelRenderLastHeader(v, c, col)"
          align="center"
          :fixed="col.fixed ? col.fixed : false"
          :prop="col.prop"
          :width="col.width ? col.width : 150"
          :label="col.label"
        >
          <template slot-scope="scope">
            <common-input
              :width="col.inputWidth || col.width"
              :disabled="
                col.disabled
                  ? col.disabled(scope.row[col.prop], scope.row, col, {
                      total,
                      index: scope.$index,
                    })
                  : false
              "
              @input="(v) => handleChangeInput(v, col, scope)"
              :type="col.type"
              v-if="
                col.type == 'input' ||
                col.type == 'number' ||
                col.type == 'float'
              "
              :max="col.max || Infinity"
              :min="col.min || 0"
              :fixed="col.fixed || ''"
              :placeholder="col.disabled ? '' : '请输入' + col.label"
              v-model="scope.row[col.prop]"
              :clearable="false"
            ></common-input>
            <el-autocomplete
              :disabled="
                col.disabled
                  ? col.disabled(scope.row[col.prop], scope.row, col, {
                      total,
                      index: scope.$index,
                    })
                  : false
              "
              v-model="scope.row[col.prop]"
              v-else-if="col.type == 'severInput'"
              :placeholder="col.disabled ? '' : '请输入' + col.label"
              :fetch-suggestions="
                (v, fn) => apiSerachGoods(v, fn, col.prop, scope)
              "
              value-key="goodsName"
              @select="(v) => handleSelectGoods(v, scope)"
            ></el-autocomplete>
            <selectDict
              :disabled="
                col.disabled
                  ? col.disabled(scope.row[col.prop], scope.row, col, {
                      total,
                      index: scope.$index,
                    })
                  : false
              "
              v-else-if="col.type == 'dictCode'"
              v-model="scope.row[col.prop]"
              :dictCode="col.dictCode"
              :placeholder="col.disabled ? '' : '请选择' + col.label"
              :clearable="col.clearable || false"
            ></selectDict>
            <common-select
              :disabled="
                col.disabled
                  ? col.disabled(scope.row[col.prop], scope.row, col, {
                      total,
                      index: scope.$index,
                    })
                  : false
              "
              v-else-if="col.type === 4"
              v-model="scope.row[col.prop]"
              :options="isUrgents"
              @change="(v) => handleChangeSelect(v, col, scope)"
              :placeholder="col.disabled ? '' : '请选择' + col.label"
            ></common-select>
            <!-- <common-input
              :disabled="
                col.disabled
                  ? col.disabled(scope.row[col.prop], scope.row, col, {
                      total,
                      index: scope.$index,
                    })
                  : false
              "
              @input="(v) => handleChangeInput(v, col, scope)"
              type="number"
              v-else-if="col.type == 'number'"
              :placeholder="col.disabled ? '' : '请输入' + col.label"
              v-model="scope.row[col.prop]"
              :clearable="false"
              :style="col.style"
            ></common-input> -->
            <el-upload
              v-else-if="
                col.type == 'upload' &&
                !(col.disabled
                  ? col.disabled(scope.row[col.prop], scope.row, col, {
                      total,
                      index: scope.$index,
                    })
                  : false)
              "
              class="upload-demo"
              multiple
              :action="url"
              :headers="header"
              :limit="col.limit"
              :file-list="scope.row[col.prop]"
              :disabled="
                col.disabled
                  ? col.disabled(scope.row[col.prop], scope.row, col, {
                      total,
                      index: scope.$index,
                    })
                  : false
              "
            >
              <el-button
                size="small"
                :disabled="
                  col.disabled
                    ? col.disabled(scope.row[col.prop], scope.row, col, {
                        total,
                        index: scope.$index,
                      })
                    : false
                "
                type="primary"
                >点击上传</el-button
              >
              <!-- <div slot="tip" class="el-upload__tip">
                只能上传jpg/png文件，且不超过500kb
              </div> -->
            </el-upload>
            <span
              v-else
              :style="
                (col.styleFormatter
                  ? col.styleFormatter(scope.row[col.prop], scope.row, col, {
                      total,
                      index: scope.$index,
                    })
                  : scope.row[col.prop] === 0
                  ? String(scope.row[col.prop])
                  : scope.row[col.prop]) || {}
              "
              >{{
                (col.formatter
                  ? col.formatter(scope.row[col.prop], scope.row, col, {
                      total,
                      index: scope.$index,
                    })
                  : scope.row[col.prop] === 0
                  ? String(scope.row[col.prop])
                  : scope.row[col.prop]) || formatterNull
              }}</span
            >
          </template>
        </el-table-column>

        <!-- <el-table-column align="center" v-if="tableData.length !== 1">
          <img
            src="./img/icon-del.png"
            slot-scope="scope"
            alt=""
            style="cursor: pointer"
            @click="handleDelTableRow(scope)"
          />
        </el-table-column> -->
      </template>
    </el-table>
  </div>
</template>

<script>
import { upload } from "@/api/common";
import config from "@/config";
import Storage from "@/utils/storage";
export default {
  name: "tableEdit",
  props: {
    columns: {
      // 列
      required: true,
      type: [Array],
      default: () => {
        return [];
      },
    },
    tableData: {
      // 列表数据
      required: true,
      type: [Array],
      default: () => {
        return [];
      },
    },
    tableRowClassName: {
      //
      required: false,
      type: [Function],
      default: () => {},
    },
    dataArray: {
      // 返回数据类型是否为列表，
      required: false,
      type: [Boolean],
      default: true,
    },
    isPaging: {
      // 是否分页
      required: false,
      type: [Boolean],
      default: false,
    },
    index: {
      // 是否添加索引列
      required: false,
      type: [Boolean],
      default: true,
    },
    selection: {
      // 是否添加列多选列
      required: false,
      type: [Boolean],
      default: false,
    },
    selectionLimit: {
      // 添加多选列并限制最大勾选数
      required: false,
      type: [Number],
      default: -1,
    },
    selectedIndex: {
      // 多选索引字段
      required: false,
      type: [String],
      default: "id",
    },
    tableHeaderStyle: {
      // 多选索引字段
      required: false,
      type: [Object],
      default: () => {
        return { background: "#F0F0F0" };
      },
    },
    stripe: {
      //是否为斑马纹
      required: false,
      type: [Boolean],
      default: false,
    },
    border: {
      //是否带有纵向边框
      required: false,
      type: [Boolean],
      default: true,
    },
    formatterNull: {
      // 自定义空字符的显示
      required: false,
      type: [String],
      default: "",
    },
    align: {
      // 居中，会被column.align覆盖
      required: false,
      type: [String],
      default: "center",
    },
    width: {
      // label宽度
      required: false,
      type: [String, Number],
      default: 180,
    },
    tableHeight: {
      // 表格高度
      required: false,
      type: [String, Number],
      default: "540px",
    },
    fixedHeight: {
      // 是否固定表格高度
      required: false,
      type: [Boolean],
      default: true,
    },
    emptyText: {
      // 空数据时显示的文本内容
      required: false,
      type: [String],
      default: "暂无数据",
    },
  },

  data() {
    return {
      flag: true,
      loading: false,
      pageSize: 20,
      total: 0,
      pageIndex: 1,
      tableParams: {},
      isSelectAll: false,
      selectedPageRows: {}, // selection 列选中值
      selectedIds: [], // selectionLimit 列选中唯一值
      selectedRows: [], // selectionLimit 列选中行
      url: config.baseUrl + upload.url,
      header: {
        platform: "seller",
        Authorization: Storage.getItem("token"),
      },
    };
  },
  updated() {
    this.$nextTick(() => {
      this.$refs.listTable.doLayout();
    });
  },
  created() {},
  computed: {
    showPagination() {
      return this.isPaging && this.total > this.pageSize;
    },
    renderColumns() {
      const vm = this;
      let columns = this.columns.map((item) => {
        if (item.eventType) {
          const publicItem = {
            listeners: {
              onCurrentEmit({ row, col, val }) {
                vm.$emit(col.eventType, row, val, col);
              },
            },
          };
          item = Object.assign({}, item, publicItem);
        }
        return item;
      });
      return columns;
    },
    tableHeightProps() {
      return this.fixedHeight ? this.tableHeight : "";
    },
  },
  methods: {
    // 订单明细表头
    modelRenderLastHeader(h, { column }, item) {
      const { require, label } = item;
      if (require) {
        return h("span", {}, [
          h(
            "span",
            {
              style: {
                color: "red",
              },
            },
            "*"
          ),
          h("span", {}, label),
        ]);
      } else {
        return h("span", {}, [h("span", {}, label)]);
      }
    },
    // 输入框事件
    handleChangeInput(v, col, scope) {
      this.$emit("handleChangeInput", v, col, scope);
    },
    // 下拉框事件
    handleChangeSelect(v, col, scope) {
      this.$emit("handleChangeSelect", v, col, scope);
    },
    // 点击事件
    click(val, row, cel) {
      this.$emit(cel.event, val, row, cel);
    },
    reloadTableList() {
      this.tableParams = JSON.parse(JSON.stringify(this.params));
      this.pageIndex = 1;
      this.fetchTableList();
    },
    refreshTableList() {
      this.tableParams = JSON.parse(JSON.stringify(this.params));
      this.fetchTableList();
    },
    clearTableList() {
      this.pageIndex = 1;
      this.tableData = [];
    },
    clearSelection() {
      // 清空勾选数据
      this.isSelectAll = false;
      this.selectedPageRows = {};
      const ctx = this.$refs.listTable;
      ctx.clearSelection();
    },
    clearLimitSelections() {
      this.selectedIds = [];
      this.selectedRows = [];
    },
    initLimitSelections(selectedRows) {
      const { selectedIndex } = this;
      this.selectedIds = selectedRows.map((item) => item[selectedIndex]) || [];
      this.selectedRows = selectedRows;
    },

    filterIndex(index) {
      return (this.pageIndex - 1) * 10 + index + 1;
    },

    initColBind(col) {
      const bind = Object.assign({}, col);
      delete bind.component;
      delete bind.listeners;
      delete bind.propsHandler;
      return bind;
    },
    bindComponentEvent(cellValue, { row, column }, col) {
      const prop = { row, col, column, cellValue };
      const handler = col.propsHandler;
      return (handler && handler(prop)) || prop;
    },
    toggleRowSelection(selections, type) {
      const ctx = this.$refs.listTable;
      selections &&
        selections.forEach((selection) => {
          ctx.toggleRowSelection(selection, type);
        });
    },
    toggleAllSelection() {
      const ctx = this.$refs.listTable;
      ctx.toggleAllSelection();
    },
    // 设置勾选项目
    setSelections() {
      let rows = this.getSelectionsByIndex(
        this.selectedPageRows[this.pageIndex]
      );
      let selections =
        rows && rows.length ? rows : this.isSelectAll ? this.tableData : [];
      if (!selections || !selections.length) {
        return;
      }
      this.toggleRowSelection(selections, true);
    },
    // 获取已勾选的数据列表
    getSelectionsByIndex(indexs) {
      const { selectedIndex } = this;
      let res = [];
      if (indexs && indexs.length) {
        this.tableData &&
          this.tableData.forEach((item) => {
            if (!this.isSelectAll && indexs.indexOf(item[selectedIndex]) > -1) {
              res.push(item);
            } else if (
              this.isSelectAll &&
              indexs.indexOf(item[selectedIndex]) < 0
            ) {
              res.push(item);
            }
          });
      }
      return res;
    },
    // 获取勾选项索引
    getToggleSelectionsIndex(selections) {
      const { selectedIndex } = this;
      let selectionsIds = [];
      selections &&
        selections.forEach((item) => {
          selectionsIds.push(item[selectedIndex]);
        });
      if (this.isSelectAll) {
        let unSelectionsIds = [];
        this.tableData &&
          this.tableData.forEach((item) => {
            let id = item[selectedIndex];
            if (selectionsIds.indexOf(id) < 0) {
              unSelectionsIds.push(id);
            }
          });
        return unSelectionsIds.length ? unSelectionsIds : null;
      }
      return selectionsIds.length ? selectionsIds : null;
    },
    handleChangePage(page) {
      this.pageIndex = page;
      this.fetchTableList();
    },
    handleSizeChange(pageSize) {
      this.pageSize = pageSize;
      this.fetchTableList();
    },
    handleLimitChange(row) {
      const { selectedIds, selectedRows, selectedIndex, selectionLimit } = this;
      if (
        selectedIds.length >= selectionLimit &&
        selectedIds.indexOf(row[selectedIndex]) < 0
      )
        return;
      const id = row[selectedIndex];
      const index = this.selectedIds.findIndex((v) => v === id);
      if (index > -1) {
        selectedIds.splice(index, 1);
        selectedRows.splice(index, 1);
      } else {
        selectedIds.push(id);
        selectedRows.push(row);
      }
      this.selectedIds = selectedIds;
      this.selectedRows = selectedRows;
      this.handleLimitSelect(selectedRows);
    },
    handleFetch(data) {
      this.$emit("onFetch", data);
    },
    handleSelectionChange(val) {
      this.$emit("handleSelectionChange", val);
    },
    handleSelect(selection) {
      this.selectedPageRows[this.pageIndex] =
        this.getToggleSelectionsIndex(selection);
      const selectedIds = [];
      for (let index in this.selectedPageRows) {
        selectedIds.push(...this.selectedPageRows[index]);
      }
      this.$emit("onSelect", selectedIds);
    },
    handleSelectAll(selection) {
      this.isSelectAll = !!selection.length;
      this.handleCleatSelectedPageRows();
      this.$emit("onSelectAll", this.isSelectAll);
    },
    handleCleatSelectedPageRows() {
      this.selectedPageRows = {}; // 初始化列表勾选数据
      this.$emit("onSelect", []);
    },
    rowClick(row) {
      this.$emit("onRowClick", row);
    },

    handleLimitSelect(selectedRows = []) {
      if (!selectedRows.length) {
        this.selectedIds = [];
        this.selectedRows = [];
      }
      this.$emit("onSelect", selectedRows);
    },
  },
};
</script>

<style scoped lang="scss">
// 表格样式
/deep/.el-table {
  .el-table__row {
    .el-table__cell {
      padding: 0;

      .cell {
        overflow: none;
        padding: 0;

        .el-input__inner {
          padding: 0;
          text-align: center;
          border-color: transparent;
        }

        .el-input__inner:focus {
          outline: none;
          border-color: #29a086 !important;
        }
      }
    }
  }
}
.listenersText {
  cursor: pointer;
}
.icon-selection {
  display: inline-block;
  width: 14px;
  height: 14px;
  border-radius: 2px;
  border: 1px solid #dcdfe6;
  font-size: 12px;
  line-height: 16px;
  text-align: center;

  &.checked {
    background-color: #29a086;
    color: #fff;
    border: none;
  }

  &.disabled {
    cursor: not-allowed;
    background-color: #ebebeb;
    &.checked {
      background-color: #a0cfff;
    }
  }
}

/deep/.el-table .warning-row {
  background: #f8dede;
  color: #eb6f5f;
  .el-input__inner {
    background: #f8dede;
    color: #eb6f5f;
  }
}
</style>
