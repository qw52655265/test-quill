<template>
  <div>
    <el-form :model="form" label-width="120px">
      <el-form-item label="Text1">
        <el-input v-model="form.text1" />
      </el-form-item>
      <el-form-item label="Text2">
        <quill-editor v-model="form.text2" />
      </el-form-item>
      <!-- 这里使用了v-if，动态生成的编辑器工具类有bug， -->
      <el-form-item v-show="isShow" label="Text3">
        <quill-editor v-model="form.text3" />
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="isShow = !isShow">
          toggleShow
        </el-button>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
import { reactive, toRefs } from "vue";
import QuillEditor from "./components/QuillEditor.vue";
export default {
  name: "App",
  components: { QuillEditor },
  setup() {
    const state = reactive({
      form: {
        text1: "text1",
        text2: "text2",
        text3: "text3",
      },
      isShow: false,
    });
    return {
      ...toRefs(state),
    };
  },
};
</script>
