<template>
  <el-dialog
    v-model="centerDialogvisible"
    title="添加商品信息"
    width="670px"
    center
    @close="closeDialog(false)"
  >
    <el-form
      ref="validateForm"
      :model="goodsForm"
      :rules="rules"
      label-width="100px"
      :size="formSize"
    >
      <el-form-item label="商品名称" prop="title">
        <el-input v-model="goodsForm.title"></el-input>
      </el-form-item>
      <el-form-item label="商品价格" prop="price">
        <el-input v-model="goodsForm.price"></el-input>
      </el-form-item>
      <el-form-item label="商品图片" prop="coverImg">
        <el-upload
          class="avatar-uploader"
          :action="uploadURL"
          :show-file-list="false"
          :on-success="handleAvatarSuccess"
          :before-upload="handleBeforeUpload"
        >
          <img v-if="imageUrl" :src="imageUrl" class="avatar" />
          <el-icon v-else class="avatar-uploader-icon"><Plus /></el-icon>
        </el-upload>
      </el-form-item>
      <el-form-item label="商品详情" prop="goodsDetail">
        <QuillEditor theme="snow" ref="editor" />
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="submitAdd">添加商品</el-button>
      </el-form-item>
    </el-form>
    <template #footer>
      <span class="dialog-footer">
        <el-button @click="closeDialog(false)">取消</el-button>
        <el-button type="primary" @click="submitAdd"
          >确认</el-button
        >
      </span>
    </template>
  </el-dialog>
</template>
<script>
import axios from "axios"
import { reactive, toRefs, ref } from "@vue/reactivity";
import { QuillEditor } from "@vueup/vue-quill";
import "@vueup/vue-quill/dist/vue-quill.snow.css";
import {ElMessage} from "element-plus"
// 添加商品的方法
function useAdd(state, validateForm, editor, emit) {
    const submitAdd = ()=> {
       state.goodsForm.goodsDetail =
       editor.value.getText().replace(/(\r\n|\n|\r)/gm, "<br />") !== "<br />"
        ? editor.value.getHTML()
        : "";
        validateForm.value.validate((valid) => {
            // vaild为true时通过效验
            if(valid) {
              const params = {
                title: state.goodsForm.title,
                price: state.goodsForm.price,
                thumbnail: state.goodsForm.coverImg,
                goodsDetail: state.goodsForm.goodsDetail,
              }
              axios.post('/goods/add',params).then( (res)=> {
                 validateForm.value.resetFields();
                 editor.value.setText("")
                 console.log(res)
                 emit('tovisible',false, res.data)
              })
            }
        })
    }
    return {submitAdd}
}
// 完成上传文件效验
const handleBeforeUpload = (file) => {
  console.log(file)
  const arr = file.name.split('.')[1]
  if(!['jpg','png'].includes(arr)){
    ElMessage.error('请上传jpg或png类型的图片格式');
    return false;
  }
}
export default {
  components: {
    QuillEditor,
  },
  props: {
    centerDialogvisible: Boolean,
  },
  emit: ["tovisible"],
  setup(props, { emit }) {
      const validateForm = ref()
      const state = reactive({
      centerDialogvisible: props.centerDialogvisible,
      uploadURL:import.meta.env.VITE_APP_URL+"/goods/fileUpload",
      imageUrl: "",
      goodsForm: {
        title: "",
        price: 0,
        coverImg: "",
        goodsDetail: "",
      },
    });
    const closeDialog = (data) => {
      emit("tovisible", data);
      state.imageUrl = "";
      state.goodsForm.title = "";
      state.goodsForm.price = 0;
      state.goodsForm.coverImg = "";
      state.goodsForm.goodsDetail = "";
    };
    const editor = ref()
    // 上传成功之后，执行处理的函数
    const handleAvatarSuccess = (response) => {
        console.log(response)
        // 检验规则是否有上传图片到表单
        state.goodsForm.coverImg = response.msg
        state.imageUrl = import.meta.env.VITE_APP_URL + response.msg
    }
    const rules = {
      title: [{ required: true, message: "请输入商品名称", trigger: "blur" }],
      price: [{ required: true, message: "请输入商品价格", trigger: "blur" }],
      coverImg: [
        { required: true, message: "请上传商品主图", trigger: "blur" },
      ],
      goodsDetail: [
        { required: true, message: "请输入商品详情", trigger: "blur" },
      ],
    };
    return {
      ...toRefs(state),
      closeDialog,
      rules,
      handleAvatarSuccess,
      ...useAdd(state, validateForm, editor, emit),
      validateForm,
      editor,
      handleBeforeUpload
    };
  },
};
</script>

<style scoped lang="scss">
::v-deep(.el-form-item__content) {
  display: block;
}

::v-deep(.avatar-uploader .el-upload) {
  border: 1px dashed #d9d9d9;
  border-radius: 6px;
  cursor: pointer;
  position: relative;
  overflow: hidden;
}
::v-deep(.avatar-uploader .el-upload:hover) {
  border-color: #409eff;
}
::v-deep(.avatar-uploader-icon) {
  font-size: 28px;
  color: #8c939d;
  width: 178px;
  height: 178px;
  line-height: 178px;
  text-align: center;
}
::v-deep(.avatar) {
  width: 80px;
  height: 80px;
}
</style>