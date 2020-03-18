<template>
  <div class="table-panel">
    <div class="search-bar" @keydown.enter="search">
      <el-input
        v-model="searchForm.name"
        placeholder="请输入商品名"
        size="small"
      ></el-input>
      <el-input
        v-model="searchForm.startPrice"
        placeholder="最低价"
        size="small"
      ></el-input>
      <i style="margin: 0 9px 0 0; color: #bbb;">--</i>
      <el-input
        v-model="searchForm.endPrice"
        placeholder="最高价"
        size="small"
      ></el-input>
      <el-select
        v-model="searchForm.special"
        placeholder="是否特价"
        value=""
        size="small"
      >
        <el-option label="是" :value="1"></el-option>
        <el-option label="否" :value="0"></el-option>
      </el-select>
      <el-select
        v-model="searchForm.newProduct"
        placeholder="是否新产品"
        value=""
        size="small"
      >
        <el-option label="是" :value="1"></el-option>
        <el-option label="否" :value="0"></el-option>
      </el-select>
      <el-select
        v-model="searchForm.statusCode"
        value=""
        size="small"
        placeholder="请选择商品状态"
      >
        <el-option
          v-for="(item, index) in statuses"
          :key="index"
          :label="item.statusName"
          :value="item.statusCodeId"
        ></el-option>
      </el-select>
      <div></div>
      <el-button
        type="primary"
        icon="el-icon-search"
        size="mini"
        @click="search"
        >查询</el-button
      >
      <el-button type="primary" plain size="mini" @click="handleClick('add')"
        >增加</el-button
      >
    </div>
    <el-table
      :data="tableData"
      style="width: 100%"
      @row-click="openDetails"
      v-loading.fullscreen="isLoading"
    >
      <el-table-column prop="name" label="商品名"></el-table-column>
      <el-table-column prop="categoryName" label="分类名"></el-table-column>
      <el-table-column prop="price" label="价格"></el-table-column>
      <el-table-column prop="unitName" label="单位名称"></el-table-column>
      <el-table-column prop="inventoryNum" label="库存数"></el-table-column>
      <el-table-column label="是否新产品">
        <template slot-scope="scope">
          <template v-if="scope.row.newProduct">是</template>
          <template v-else>否</template>
        </template>
      </el-table-column>
      <el-table-column prop="statusName" label="状态"></el-table-column>
      <el-table-column label="操作" width="300">
        <template slot-scope="scope">
          <el-button
            @click.stop="handleClick('update', scope.row)"
            type="primary"
            size="mini"
            >编辑</el-button
          >
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="searchForm.pageNum"
      :page-sizes="[5, 10, 15, 20]"
      :page-size="searchForm.pageSize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total"
      class="page"
    >
    </el-pagination>
  </div>
</template>

<script>
import _ from "lodash";
import materialDetailPage from "@/views/material-list-details-page";
import materialEditPage from "@/views/material-list-edit-page";
export default {
  name: "order-list-page",
  data() {
    return {
      searchForm: {
        name: "",
        startPrice: "",
        endPrice: "",
        special: "",
        newProduct: "",
        categoryId: "",
        statusCode: "",
        pageNum: 1,
        pageSize: 5
      },
      currentPage: 1,
      tableData: [],
      isLoading: false,
      total: 0,
      statuses: []
    };
  },
  methods: {
    search() {
      this.isLoading = true;
      this.$api.orderManageAPI.getOrderList(this.searchForm).then(res => {
        if (_.isEqual(res.code, "success")) {
          this.tableData = res.data;
          this.total = res.total;
          this.isLoading = false;
        } else {
          this.$vb.plugin.message.error(res.msg);
        }
      });
    },
    handleClick(state, ...val) {
      switch (state) {
        case "detail":
          this.$vb.plugin.openLayer(
            materialDetailPage,
            this,
            { colNum: "two-col", materialId: val[0].materialId },
            "订单详情",
            800,
            "80%"
          );
          break;
        case "add":
          this.$vb.plugin.openLayer(
            materialEditPage,
            this,
            { state: state },
            "商品增加",
            800,
            "80%"
          );
          break;
        case "update":
          this.$vb.plugin.openLayer(
            materialEditPage,
            this,
            { state: state, materialId: val[0].materialId },
            "商品编辑",
            800,
            "80%"
          );
          break;
      }
    },
    handleSizeChange(size) {
      this.searchForm.pageSize = size;
      this.search();
    },
    handleCurrentChange(page) {
      this.searchForm.pageNum = page;
      this.search();
    },
    openDetails(row) {
      this.handleClick("detail", row);
    }
  },
  mounted() {
    this.search();
    // 获取商品状态集合
    this.$api.orderManageAPI.getMaterialStatus({}).then(res => {
      this.statuses = res.data;
    });
  }
};
</script>

<style scoped lang="scss">
@import "~@/assets/scss/order-list-page.scss";
</style>