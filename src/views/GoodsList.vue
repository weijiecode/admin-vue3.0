<template>
  <el-card class="box-card">
    <template #header>
      <div class="card-header">
        <img src="../assets/logo.png" style="width: 20px; height: 20px" />
        <span>商品列表</span>
      </div>
    </template>
    <el-row :gutter="10">
      <el-col :span="4">
        <el-input v-model="searchContent" placeholder="请输入搜索商品名称"></el-input>
      </el-col>
      <el-col :span="4">
        <el-button type="primary" @click="handleSearch">搜索</el-button>
        <el-button type="primary" @click="adddialog">添加</el-button>
      </el-col>
    </el-row>
    <el-table :data="tableData" style="width: 100%">
      <el-table-column label="编号" width="180">
        <template #default="scope">
          <div style="display: flex; align-items: center">
            <el-icon><timer /></el-icon>
            <span style="margin-left: 10px">{{ scope.row.id }}</span>
          </div>
        </template>
      </el-table-column>
      <el-table-column label="商品名称" width="180">
        <template #default="scope">
          <el-popover
            effect="light"
            trigger="hover"
            placement="top"
            width="auto"
          >
            <template #default>
              <div>商品名称: {{ scope.row.title }}</div>
              <div>单价: {{ scope.row.price }}</div>
            </template>
            <template #reference>
              <el-tag>{{ scope.row.title }}</el-tag>
            </template>
          </el-popover>
        </template>
      </el-table-column>
      <el-table-column label="图片" width="180">
        <template #default="scope">
          <div style="display: flex; align-items: center">
            <img
              :src="url + scope.row.thumbnail"
              alt=""
              style="width: 100px; height: 100px"
            />
          </div>
        </template>
      </el-table-column>
      <el-table-column label="Operations">
        <template #default="scope">
          <el-button size="small" @click="handleEdit(scope.$index, scope.row)"
            >编辑</el-button
          >
          <el-button
            size="small"
            type="danger"
            @click="handleDelete(scope.$index, scope.row)"
            >删除</el-button
          >
        </template>
      </el-table-column>
    </el-table>
     <el-pagination background layout="prev, pager, next" :page-size="pageSize"
     :current-page="currentPage"
     @current-change="changePage"
      :total="total" />
  </el-card>
  <AddProduct :centerDialogvisible="centerDialogvisible" @tovisible="dialogisvisible"></AddProduct>
  <EditProduct
  v-if='editDialogVisible===true'
   :editDialogVisible="editDialogVisible" :goodsId="pId"></EditProduct>
</template>
<script>
import axios from "axios";
import { onMounted, reactive, toRefs } from "vue";
import AddProduct from "./AddProduct.vue"
import { ElMessageBox, ElNotification } from "element-plus";
import EditProduct from './EditProduct.vue'
//所有数据
// function loadDate(state) {
//   axios.get("/products").then((res) => {
//     console.log(res);
//     state.tableData = res.data;
//   });
// }
function loadDate(state) {
  const params = {
    pageNumber: state.currentPage,
    pageSize: state.pageSize,
    search: state.searchContent
  }
  axios.get("/goods",{ params }).then((res) => {
    console.log(res);
    state.tableData = res.data.list;
    state.total = res.data.totalCount;
  });
}
// 编辑商品信息
function useEdit(state) {
  const handleEdit = (index, row) => {
    state.editDialogVisible = true;
    state.pId = row.id
  }
  return {handleEdit}
}
// 搜索商品数据
function useHandleSearch(state) {
  const handleSearch = () => {
    state.currentPage = 1;
    const params = {
      pageNumber: state.currentPage,
      pageSize: state.pageSize,
      search: state.searchContent,
    };
    axios.get('/goods',{params}).then((res) => {
      state.tableData = res.data.list;
      state.total = res.data.totalCount;
    })
  }
  return {handleSearch}
}
// 删除商品数据
function useDelete(state) {
  const handleDelete = (index, row) => {
    // console.log("index=", index);
    // console.log("row=", row);
    ElMessageBox.confirm("你确定要删除该记录吗?", "提示", {
      confirmButtonText: "确定",
      cancelButtonText: "取消",
      type: "warning",
      callback: (action) => {
        if (action === "confirm") {
          axios.delete('/goods/delete',{
            params: {id: row.id}
          }).then((res) => {
            if (res.data.code === "ok") {
                ElNotification({
                  title: "成功",
                  message: "删除成功",
                  type: "success",
                  duration: 2000,
                });
                loadDate(state);
              } else {
                ElNotification({
                  title: "失败",
                  message: "删除失败",
                  type: "error",
                  duration: 2000,
                });
              }
          })
        }
      },
    });
  };
  return {
    handleDelete,
  };
}
export default {
  components: {
    AddProduct,
    EditProduct
  },
  setup() {
    const state = reactive({
      tableData: [],
      centerDialogvisible: false,
      editDialogVisible: false, // 控制编辑商品窗口的显示与隐藏
      total: 0,
      currentPage: 1,
      pageSize: 3,
      searchContent: "", // 搜索条件
      pId: 0, // 要编辑的商品的id
    });
    const url = import.meta.env.VITE_APP_URL;
    const dialogisvisible = (visible,data) => {
      state.centerDialogvisible = visible
      if(data !== undefined){
        loadDate(state);
      }
    }
    const adddialog = () => {
      state.centerDialogvisible = true
    }
    onMounted(() => {
      loadDate(state);
    });
    // 页面切换执行的方法（该方法简单直接在setup里面定义）
    const changePage = (val)=> {
      // val的值为点的页码值
      state.currentPage = val
      loadDate(state);
    }
    return {
      ...toRefs(state),
      dialogisvisible,
      adddialog,
      url,
      ...useDelete(state),
      changePage,
      ...useHandleSearch(state),
      ...useEdit(state)
    };
  },
};
</script>
<style scoped>
</style>
