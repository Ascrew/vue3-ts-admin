<!--
 * @Author: zhangyu
 * @Date: 2022-07-19 15:14:28
 * @LastEditors: Please set LastEditors
 * @Description:  后台管理登录界面
 * @FilePath: src/views/login/index.vue
-->
<template>
  <div class="login-container">
    <div class="main">
      <el-form ref="formRef" :model="form" :rules="rules" label-width="120px">
        <el-form-item label="用户名" prop="username">
          <el-input
            v-model="form.username"
            placeholder="请输入用户名"
          ></el-input>
        </el-form-item>
        <el-tooltip
          :visible="capsTooltip"
          content="Caps lock is on"
          placement="right"
          manual
        >
          <el-form-item label="密码" prop="password">
            <el-input
              v-model="form.password"
              placeholder="请输入密码"
              @keyup="checkCapslock"
              @blur="capsTooltip = false"
              @keyup.enter="login"
            ></el-input>
          </el-form-item>
        </el-tooltip>
        <el-form-item>
          <el-button
            :loading="loading"
            class="login-btn"
            type="primary"
            @click="login"
            >登录</el-button
          >
        </el-form-item>
      </el-form>
    </div>
  </div>
</template>

<script lang="ts" setup>
import { reactive, ref, watch } from "vue";
import { ElMessage } from "element-plus";
import { loginApi } from "@/api/admin/adminUser";
import { useStore } from "vuex";
import {
  useRoute,
  useRouter,
  LocationQuery,
  RouteLocationNormalizedLoaded,
} from "vue-router";

interface Form {
  username: string;
  password: string;
}

const store = useStore();
const route = useRoute();
const router = useRouter();
const getOtherQuery = (query: LocationQuery) => {
  return Object.keys(query).reduce((acc, cur) => {
    if (cur !== "redirect") {
      acc[cur] = query[cur];
    }
    return acc;
  }, {} as LocationQuery);
};
watch(route, (newVal: RouteLocationNormalizedLoaded) => {
  const query = newVal.query as LocationQuery;
  if (query) {
    redirect.value = query.redirect as string;
    otherQuery = getOtherQuery(query);
  }
});

const formRef = ref();
const form = reactive<Form>({
  username: "" as string,
  password: "" as string,
});
const rules = reactive({
  username: [{ required: true, message: "请输入用户名", trigger: "blur" }],
  password: [{ required: true, message: "请输入密码", trigger: "blur" }],
});
const loading = ref(false);
const redirect = ref("");
// const otherQuery: Dictionary<string> = {};
let otherQuery = reactive({});

const login = async () => {
  await formRef.value.validate((valid: boolean) => {
    if (valid) {
      loading.value = true;
      loginApi(form)
        .then(async (res) => {
          if (res.code === 0) {
            ElMessage.success("登录成功");
            await store.dispatch("Login", res);
            router.push({
              path: redirect.value || "/",
              query: otherQuery,
            });
          }
        })
        .finally(() => {
          loading.value = false;
        });
    }
  });
};

let capsTooltip = ref(false);
const checkCapslock = (e: KeyboardEvent) => {
  const { key } = e;
  capsTooltip.value =
    key !== null && key.length === 1 && key >= "A" && key <= "Z";
};
</script>

<style lang="scss" scoped>
.login-container {
  overflow: hidden;
  background: no-repeat center/cover;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
}
:deep(.login-btn) {
  width: 100%;
}
</style>
