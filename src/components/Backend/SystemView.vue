<template>
  <div class="system">
    <el-container>
      <el-main>
        <!--    管理员信息设置-->
        <el-card>
          <template #header>
            <div class="card-header">
              <span>管理员信息设置</span>
            </div>
          </template>
          <el-button
            type="primary"
            @click="updatePasswordDialog(updatePasswordFormRef)"
            >修改管理员密码</el-button
          >
        </el-card>

        <!--    图书种类信息设置-->
        <!-- <el-card>
          <template #header>
            <div class="card-header">
              <span>图书种类设置</span>
            </div>
          </template>
          <el-button
            type="primary"
            @click="addBookTypeDialog(addBookTypeFormRef)"
            >添加图书种类</el-button
          >
          <el-button
            type="primary"
            @click="deleteBookTypeDialog(deleteBookTypeFormRef)"
            >删除图书种类</el-button
          >
        </el-card> -->

        <!--        修改密码对话框-->
        <el-dialog
          v-model="updatePasswordDialogVisible"
          title="修改密码"
          width="500px"
          :close-on-click-modal="false"
        >
          <el-form
            :model="updatePasswordForm"
            :rules="updatePasswordRules"
            ref="updatePasswordFormRef"
          >
            <el-form-item
              label="邮箱"
              :label-width="formLabelWidth"
              prop="email"
            >
              <el-input
                v-model="updatePasswordForm.email"
                autocomplete="off"
              ></el-input>
            </el-form-item>
            <el-form-item
              label="验证码"
              :label-width="formLabelWidth"
              prop="code"
            >
              <div class="flex-container">
                <el-input
                  v-model="updatePasswordForm.code"
                  autocomplete="off"
                  class="input-with-spacing"
                >
                </el-input>
                <el-button
                  type="warning"
                  @click="addEmailButton(updatePasswordFormRef)"
                  class="button-with-spacing"
                >
                  获取验证码
                </el-button>
              </div>
            </el-form-item>
            <el-form-item
              label="新密码"
              :label-width="formLabelWidth"
              prop="password"
            >
              <el-input
                v-model="updatePasswordForm.password"
                type="password"
                autocomplete="off"
              />
            </el-form-item>
          </el-form>
          <template #footer>
            <span class="dialog-footer">
              <el-button @click="updatePasswordDialogVisible = false"
                >取消</el-button
              >
              <el-button
                type="primary"
                @click="updatePassword(updatePasswordFormRef)"
              >
                修改
              </el-button>
            </span>
          </template>
        </el-dialog>

        <!--        添加图书种类对话框-->
        <el-dialog
          v-model="addBookTypeDialogVisible"
          title="添加图书种类"
          width="500px"
          :close-on-click-modal="false"
        >
          <el-form
            :model="addBookTypeForm"
            :rules="BookTypeRules"
            ref="addBookTypeFormRef"
          >
            <el-form-item label="分类名称" prop="categoryName">
              <el-input v-model="addBookTypeForm.categoryName" autocomplete="off" />
            </el-form-item>
            <el-form-item label="分类别名" prop="categoryAlias">
              <el-input v-model="addBookTypeForm.categoryAlias" autocomplete="off" />
            </el-form-item>
          </el-form>
          <template #footer>
            <span class="dialog-footer">
              <el-button @click="addBookTypeDialogVisible = false"
                >取消</el-button
              >
              <el-button
                type="primary"
                @click="addBookType(addBookTypeFormRef)"
              >
                添加
              </el-button>
            </span>
          </template>
        </el-dialog>

        <!--        删除图书种类对话框-->
        <el-dialog
          v-model="deleteBookTypeDialogVisible"
          title="删除图书种类"
          width="500px"
          :close-on-click-modal="false"
        >
          <el-form
            :model="deleteBookTypeForm"
            :rules="BookTypeRules"
            ref="deleteBookTypeFormRef"
          >
            <el-form-item label="种类名称" prop="categoryName">
              <el-input
                v-model="deleteBookTypeForm.categoryName"
                autocomplete="off"
              />
            </el-form-item>
          </el-form>
          <template #footer>
            <span class="dialog-footer">
              <el-button @click="deleteBookTypeDialogVisible = false"
                >取消</el-button
              >
              <el-button
                type="danger"
                @click="deleteBookType(deleteBookTypeFormRef)"
              >
                删除
              </el-button>
            </span>
          </template>
        </el-dialog>
      </el-main>
    </el-container>
  </div>
</template>

<script lang="ts" setup>
import { reactive, ref } from "vue";
import type { FormInstance, FormRules } from "element-plus";
import axios from "axios";
import { ElMessageBox } from "element-plus";
// 标签长度
let formLabelWidth = 120;

// 修改密码对话框
let updatePasswordDialogVisible = ref(false);
const updatePasswordDialog = (formEl: FormInstance | undefined) => {
  updatePasswordDialogVisible.value = true;
  if (!formEl) return;
  formEl.resetFields();
};

// 修改密码表单
const updatePasswordFormRef = ref<FormInstance>();
const updatePasswordForm = reactive({
  email: "",
  password: "",
  code: "",
});

// 修改密码表单规则
const updatePasswordRules = reactive<FormRules>({
  //password: [{ required: true, message: "请输入新密码", trigger: "blur" }],
  email: [{ required: true, message: "请输入新密码", trigger: "blur" }],
});
// 邮箱验证按钮
const addEmailButton = (formEl: FormInstance | undefined) => {
  if (!formEl) return;
  formEl.validate((valid) => {
    if (valid) {
      axios.post("/api/manager/captcha/", updatePasswordForm).then((resp) => {
        console.log(resp);
        const code = resp.data.code;
        const message = resp.data.message;
        // 添加失败
        if (code == 400) {
          ElMessageBox.alert(message, {
            confirmButtonText: "确认",
          });
        }
        // 添加成功
        if (code == 200) {
          ElMessageBox.alert(message, {
            confirmButtonText: "确认",
          });
        }
      });
    } else {
      return false;
    }
  });
};

// 修改密码
const updatePassword = (formEl: FormInstance | undefined) => {
  if (!formEl) return;
  formEl.validate((valid) => {
    if (valid) {
      axios
        .post("/api/manager/updatepassword/", updatePasswordForm)
        .then((resp) => {
          const code = resp.data.code;
          const message = resp.data.message;

          // 修改失败
          if (code == 400) {
            ElMessageBox.alert(message, {
              confirmButtonText: "确认",
            });
          }
          // 修改成功
          if (code == 200) {
            ElMessageBox.alert(message, {
              confirmButtonText: "确认",
              callback: () => {
                updatePasswordDialogVisible.value = false;
              },
            });
          }
        });
    }
  });
};

// 图书种类表单规则
const BookTypeRules = reactive<FormRules>({
  categoryName: [
    { required: true, message: "请输入图书分类名称", trigger: "blur" },
  ],
  categoryAlias: [
    { required: true, message: "请输入图书分类别名", trigger: "blur" },
  ],
});

// 添加图书种类
let addBookTypeDialogVisible = ref(false);
const addBookTypeDialog = (formEl: FormInstance | undefined) => {
  addBookTypeDialogVisible.value = true;
  if (!formEl) return;
  formEl.resetFields();
};

// 添加图书种类表单
const addBookTypeFormRef = ref<FormInstance>();
const addBookTypeForm = reactive({
  categoryName: "",
  categoryAlias: "",
});

// 添加图书种类
const addBookType = (formEl: FormInstance | undefined) => {
  if (!formEl) return;
  formEl.validate((valid) => {
    if (valid) {
      axios
        .post("/category", addBookTypeForm)
        .then((resp) => {
          const code = resp.data.code;

          // 添加失败
          // if (code == ) {
          //   ElMessageBox.alert("添加图书种类失败，请重试", "信息", {
          //     confirmButtonText: "确认",
          //   });
          // }
          // 添加成功
          if (code == 0) {
            ElMessageBox.alert("添加图书种类成功", "信息", {
              confirmButtonText: "确认",
              callback: () => {
                addBookTypeDialogVisible.value = false;
              },
            });
          }
          // // 图书种类已存在
          // if (code == 2) {
          //   ElMessageBox.alert("该种类名称已存在", "信息", {
          //     confirmButtonText: "确认",
          //   });
          // }
        });
    }
  });
};

// 删除图书种类
let deleteBookTypeDialogVisible = ref(false);
const deleteBookTypeDialog = (formEl: FormInstance | undefined) => {
  deleteBookTypeDialogVisible.value = true;
  if (!formEl) return;
  formEl.resetFields();
};

// 删除图书种类表单
const deleteBookTypeFormRef = ref<FormInstance>();
const deleteBookTypeForm = reactive({
  categoryName: "",
});

// 删除图书种类
const deleteBookType = (formEl: FormInstance | undefined) => {
  if (!formEl) return;
  formEl.validate((valid) => {
    if (valid) {
      axios
        .post("http://localhost:8888/type/delete", deleteBookTypeForm)
        .then((resp) => {
          const code = resp.data.code;

          // 删除失败
          if (code == 0) {
            ElMessageBox.alert("删除图书种类失败，请重试", "信息", {
              confirmButtonText: "确认",
            });
          }
          // 删除成功
          if (code == 1) {
            ElMessageBox.alert("删除图书种类成功", "信息", {
              confirmButtonText: "确认",
              callback: () => {
                deleteBookTypeDialogVisible.value = false;
              },
            });
          }
          // 图书种类不存在
          if (code == 2) {
            ElMessageBox.alert("该种类名称不存在", "信息", {
              confirmButtonText: "确认",
            });
          }
        });
    }
  });
};
</script>

<style lang="scss">
@import "@/assets/css/system";
</style>
