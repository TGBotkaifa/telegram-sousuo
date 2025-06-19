<template>
  <Dialog :title="dialogTitle" v-model="dialogVisible">
    <el-form
      ref="formRef"
      :model="formData"
      :rules="formRules"
      label-width="100px"
      v-loading="formLoading"
    >
      <el-form-item label="群组id(或者唯一标识)" prop="cardId">
        <el-input v-model="formData.cardId" placeholder="请输入群组id(或者唯一标识)" />
      </el-form-item>
      <el-form-item label="创建人id" prop="createdUserId">
        <el-input v-model="formData.createdUserId" placeholder="请输入创建人id" />
      </el-form-item>
      <el-form-item label="标题" prop="title">
        <el-input v-model="formData.title" placeholder="请输入标题" />
      </el-form-item>
      <el-form-item label="拼接地址---JSON" prop="username">
        <el-input v-model="formData.username" placeholder="请输入拼接地址---JSON" />
      </el-form-item>
      <el-form-item label="channel、group、supergroup" prop="type">
        <el-select v-model="formData.type" placeholder="请选择channel、group、supergroup">
          <el-option label="请选择字典生成" value="" />
        </el-select>
      </el-form-item>
      <el-form-item label="人数" prop="memberCount">
        <el-input v-model="formData.memberCount" placeholder="请输入人数" />
      </el-form-item>
      <el-form-item label="是否公开" prop="isPublic">
        <el-input v-model="formData.isPublic" placeholder="请输入是否公开" />
      </el-form-item>
      <el-form-item label="频道说明" prop="description">
        <Editor v-model="formData.description" height="150px" />
      </el-form-item>
    </el-form>
    <template #footer>
      <el-button @click="submitForm" type="primary" :disabled="formLoading">确 定</el-button>
      <el-button @click="dialogVisible = false">取 消</el-button>
    </template>
  </Dialog>
</template>
<script setup lang="ts">
import { GroupApi, GroupVO } from '@/api/collect/bot/group'

/** 群组信息 表单 */
defineOptions({ name: 'GroupForm' })

const { t } = useI18n() // 国际化
const message = useMessage() // 消息弹窗

const dialogVisible = ref(false) // 弹窗的是否展示
const dialogTitle = ref('') // 弹窗的标题
const formLoading = ref(false) // 表单的加载中：1）修改时的数据加载；2）提交的按钮禁用
const formType = ref('') // 表单的类型：create - 新增；update - 修改
const formData = ref({
  id: undefined,
  cardId: undefined,
  createdUserId: undefined,
  title: undefined,
  username: undefined,
  type: undefined,
  memberCount: undefined,
  isPublic: undefined,
  description: undefined,
})
const formRules = reactive({
  username: [{ required: true, message: '拼接地址---JSON不能为空', trigger: 'blur' }],
})
const formRef = ref() // 表单 Ref

/** 打开弹窗 */
const open = async (type: string, id?: number) => {
  dialogVisible.value = true
  dialogTitle.value = t('action.' + type)
  formType.value = type
  resetForm()
  // 修改时，设置数据
  if (id) {
    formLoading.value = true
    try {
      formData.value = await GroupApi.getGroup(id)
    } finally {
      formLoading.value = false
    }
  }
}
defineExpose({ open }) // 提供 open 方法，用于打开弹窗

/** 提交表单 */
const emit = defineEmits(['success']) // 定义 success 事件，用于操作成功后的回调
const submitForm = async () => {
  // 校验表单
  await formRef.value.validate()
  // 提交请求
  formLoading.value = true
  try {
    const data = formData.value as unknown as GroupVO
    if (formType.value === 'create') {
      await GroupApi.createGroup(data)
      message.success(t('common.createSuccess'))
    } else {
      await GroupApi.updateGroup(data)
      message.success(t('common.updateSuccess'))
    }
    dialogVisible.value = false
    // 发送操作成功的事件
    emit('success')
  } finally {
    formLoading.value = false
  }
}

/** 重置表单 */
const resetForm = () => {
  formData.value = {
    id: undefined,
    cardId: undefined,
    createdUserId: undefined,
    title: undefined,
    username: undefined,
    type: undefined,
    memberCount: undefined,
    isPublic: undefined,
    description: undefined,
    status: undefined,
  }
  formRef.value?.resetFields()
}
</script>
