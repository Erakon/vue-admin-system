<!--商品分类列表-->
<template>
  <div class="table-panel">
    <div class="search-bar" @keydown.enter="search">
      <el-input
        v-model="searchForm.name"
        placeholder="请输入品牌名"
        size="small"
      />
      <el-button
        type="primary"
        icon="el-icon-search"
        size="small"
        @click="search"
        >查询</el-button
      >
      <el-button type="primary" plain size="small" @click="handleClick(0)"
        >增加</el-button
      >
    </div>
    <el-table
      :data="tableData"
      style="width: 100%"
      @row-click="openDetails"
      v-loading.fullscreen="isLoading"
    >
      <el-table-column prop="brandName" label="品牌名" />
      <el-table-column fixed="right" width="150" label="操作">
        <template slot-scope="scope">
          <el-button
            @click.stop="handleClick(1, scope.row)"
            type="primary"
            size="mini"
            >变动</el-button
          >
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="searchForm.pageNum"
      :page-sizes="[10, 20, 30, 40]"
      :page-size="searchForm.pageSize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total"
      class="page"
    >
    </el-pagination>
  </div>
</template>
<script>
import lodash from "lodash";
import commodityBrandEditPage from "@/views/commodity-brand-edit-page";
import commodityBrandDetailsPage from "@/views/commodity-brand-details-page";

export default {
  name: "commodity-classify-list-page",
  data() {
    return {
      tableData: [],
      searchForm: {
        name: "",
        pageNum: 1,
        pageSize: 10
      },
      currentPage: 1,
      isLoading: false,
      total: 0
    };
  },
  mounted() {
    this.search();
  },
  methods: {
    search() {
      this.isLoading = true;
      this.$api.commodityBrandMangeAPI
        .getCommodityBrandList(this.searchForm)
        .then(res => {
          if (lodash.isEqual(res.code, "success")) {
            this.tableData = res.data;
            this.total = res.total;
            this.isLoading = false;
          } else {
            this.$vb.plugin.message.error(res.msg);
          }
        });
    },
    handleClick(status, row) {
      switch (status) {
        case 0:
          // 增加
          this.$vb.plugin.openLayer(
            commodityBrandEditPage,
            this,
            { state: "add" },
            "商品品牌增加",
            400,
            220
          );
          break;
        case 1:
          // 变动
          this.$vb.plugin.openLayer(
            commodityBrandEditPage,
            this,
            Object.assign({ state: "update" }, row),
            "商品品牌变动",
            400,
            220
          );
          break;
        case 2:
          // 详情
          this.$vb.plugin.openLayer(
            commodityBrandDetailsPage,
            this,
            { brandId: row.brandId, colNum: "one-col" },
            "商品品牌详情",
            400,
            220
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
      this.handleClick(2, row);
    }
  }
};
</script>

<style scoped lang="scss">
@import "~@/assets/scss/commodity-brand-list.scss";
</style>
