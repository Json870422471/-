<template>
  <div class="table-container" v-if="flag">
    <!--:max-height="fixedHeight ? tableHeight : ''"-->
    <el-table
      ref="listTable"
      v-loading="loading"
      :data="tableData"
      :border="border"
      :stripe="stripe"
      :height="height ? height : null"
      :max-height="fixedHeight && height == 0 ? tableHeight : null"
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
        key="selection"
        type="selection"
        :align="align"
        width="40"
      >
      </el-table-column>
      <el-table-column
        v-if="selectionLimit > -1"
        key="selectionLimit"
        :align="align"
        width="40"
      >
        <template slot="header"><span>选择</span></template>
        <template slot-scope="scope">
          <i
            :key="scope.row[selectedIndex]"
            :class="`${
              selectedIds.indexOf(scope.row[selectedIndex]) > -1
                ? 'el-icon-check checked'
                : ''
            } icon-selection ${
              selectedIds.length >= selectionLimit &&
              selectedIds.indexOf(scope.row[selectedIndex]) < 0
                ? 'disabled'
                : ''
            }`"
            @click="handleLimitChange(scope.row)"
          ></i>
          <!--          <el-checkbox :key="scope.row[selectedIndex]" :checked="selectedIds.indexOf(scope.row[selectedIndex])>-1" :disabled="selectedIds.length>=selectionLimit&&selectedIds.indexOf(scope.row[selectedIndex])<0" @change="(e)=>handleLimitChange(e, scope.row)"></el-checkbox>-->
        </template>
      </el-table-column>
      <el-table-column
        v-if="index"
        :align="align"
        type="index"
        width="60"
        :index="filterIndex"
        label="序号"
      ></el-table-column>
      <el-table-column
        v-for="col in renderColumns"
        :key="col.prop"
        :label="col.label"
        :min-width="col.width || width"
        :width="col.width"
        :align="col.align || align"
        v-bind="initColBind(col)"
      >
        <template slot-scope="scope">
          <component
            v-if="col.component"
            :is="col.component"
            v-bind="
              bindComponentEvent(
                col.formatter
                  ? col.formatter(scope.row[col.prop], scope.row, col, {
                      total,
                      index: scope.$index,
                    })
                  : scope.row[col.prop],
                scope,
                col
              )
            "
            v-on="col.listeners"
          ></component>
          <span
            v-else-if="col.dictCode"
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
            >{{ getDictCodeLabel(col.dictCode, scope.row[col.prop]) }}</span
          >
          <span
            class="listenersText"
            v-else-if="col.event"
            @click="
              click(scope.row[col.prop], scope.row, col, {
                total,
                index: scope.$index,
              })
            "
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
    </el-table>
    <pagination
      @changePage="handleChangePage"
      @handleSizeChange="handleSizeChange"
      :total="total"
      :pageSize="pageSize"
      v-show="showPagination"
      :currentPage="pageIndex"
    ></pagination>
  </div>
</template>

<script>
import { dictItemByCode } from "@/api/common";
export default {
  name: "tableTmp",
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
      default: function () {},
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
    dataMap: {
      //
      required: false,
      type: [String],
      default: "records",
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
    height: {
      // 表格固定高度
      required: false,
      type: [String, Number],
      default: 0,
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
    };
  },
  created() {
    this.columns.forEach((i) => {
      if (i.dictCode) {
        this.flag = false;
        this.$request(dictItemByCode, { dictCode: i.dictCode }).then(
          ({ code, data }) => {
            if (code == 200) {
              this[i.dictCode] = data;
              this.flag = true;
            }
          }
        );
      }
    });
  },
  updated() {
    this.$nextTick(() => {
      this.$refs.listTable.doLayout();
    });
  },
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
    async fetchTableList() {
      const {
        tableParams,
        method,
        pageSize,
        pageIndex,
        isPaging,
        dataArray,
        dataMap,
        pageMap,
      } = this;
      const payload = {
        ...tableParams,
      };
      if (isPaging) {
        payload["pageSize"] = pageSize;
        payload["pageNo"] = pageIndex;
      }
      this.loading = true;
      //console.log(method);
      const { code, data } = await this.$request(method, payload);
      // console.log(data, code);
      this.loading = false;
      this.handleFetch(data);
      if (code === 200) {
        const tableData = data
          ? dataMap
            ? isPaging && pageMap
              ? (data[pageMap] && data[pageMap][dataMap]) || []
              : data[dataMap] || []
            : data
          : [];
        if (tableData instanceof Array) {
          this.total =
            (pageMap && data
              ? data[pageMap] && data[pageMap].total
              : data.total) || tableData.length;
          const isRender =
            this.tableData.length && tableData.length > this.tableData.length;
          this.tableData = tableData.map((v, i) => {
            v.index = pageSize * (pageIndex - 1) + (i + 1);
            return v;
          });
          isRender && this.$forceUpdate();
        } else {
          this.tableData = [tableData];
        }
        if (this.tableData.length && this.selection) {
          this.$nextTick(() => {
            this.setSelections();
          });
        }
      } else {
        this.tableData = [];
      }
      this.$emit("tableDataChange", this.tableData);
    },
    // 数据字典回显
    getDictCodeLabel(dictCode, value) {
      let index = this[dictCode].findIndex((i) => i.value == value);
      return index === -1 ? "" : this[dictCode][index].label;
    },
    doLayout() {
      this.$nextTick(() => {
        this.$refs.listTable.doLayout();
      });
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
    clearSections() {
      // 清空勾选数据
      this.isSelectAll = false;
      this.selectedPageRows = {};
      const layout = this.$refs.listTable;
      const ctx = layout.$children[0];
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
      const props = { row, col, column, cellValue };
      const handler = col.propsHandler;
      return (handler && handler(props)) || props;
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
        .el-input-number {
          padding-right: -10px;
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
}
</style>
