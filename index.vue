<template>
  <ContentWrap>
    <!-- 搜索工作栏 -->
    <el-form
      class="-mb-15px"
      :model="queryParams"
      ref="queryFormRef"
      :inline="true"
      label-width="68px"
    >
      <el-form-item label="群组id(或者唯一标识)" prop="cardId">
        <el-input
          v-model="queryParams.cardId"
          placeholder="请输入群组id(或者唯一标识)"
          clearable
          @keyup.enter="handleQuery"
          class="!w-240px"
        />
      </el-form-item>
      <el-form-item label="标题" prop="title">
        <el-input
          v-model="queryParams.title"
          placeholder="请输入标题"
          clearable
          @keyup.enter="handleQuery"
          class="!w-240px"
        />
      </el-form-item>
      <el-form-item label="拼接地址" prop="username">
        <el-input
          v-model="queryParams.username"
          placeholder="请输入拼接地址"
          clearable
          @keyup.enter="handleQuery"
          class="!w-240px"
        />
      </el-form-item>
      <el-form-item label="群组类型" prop="type">
        <el-select v-model="queryParams.type" placeholder="请选择状态" clearable class="!w-240px">
          <el-option
            v-for="dict in ['channel', 'group', 'supergroup']"
            :key="dict"
            :label="dict"
            :value="dict"
          />
        </el-select>
      </el-form-item>
      <el-form-item label="群组状态" prop="status">
        <el-select v-model="queryParams.status" placeholder="请选择状态" clearable class="!w-240px">
          <el-option
            v-for="dict in getIntDictOptions(DICT_TYPE.COLLECT_GROUP_STATUS)"
            :key="dict.value"
            :label="dict.label"
            :value="dict.value"
          />
        </el-select>
      </el-form-item>
      <el-form-item label="是否公开" prop="isPublic">
        <el-input
          v-model="queryParams.isPublic"
          placeholder="请输入是否公开"
          clearable
          @keyup.enter="handleQuery"
          class="!w-240px"
        />
      </el-form-item>
      <el-form-item label="创建时间" prop="createTime">
        <el-date-picker
          v-model="queryParams.createTime"
          value-format="YYYY-MM-DD HH:mm:ss"
          type="daterange"
          start-placeholder="开始日期"
          end-placeholder="结束日期"
          :default-time="[new Date('1 00:00:00'), new Date('1 23:59:59')]"
          class="!w-220px"
        />
      </el-form-item>
      <el-form-item>
        <el-button @click="handleQuery">
          <Icon icon="ep:search" class="mr-5px" />
          搜索
        </el-button>
        <el-button @click="resetQuery">
          <Icon icon="ep:refresh" class="mr-5px" />
          重置
        </el-button>
        <el-button
          type="primary"
          plain
          @click="openForm('create')"
          v-hasPermi="['collect:group:create']"
        >
          <Icon icon="ep:plus" class="mr-5px" />
          新增
        </el-button>
        <el-button
          type="success"
          plain
          @click="handleExport"
          :loading="exportLoading"
          v-hasPermi="['collect:group:export']"
        >
          <Icon icon="ep:download" class="mr-5px" />
          导出
        </el-button>
        <el-button type="warning" plain @click="handleImport" v-hasPermi="['collect:group:import']">
          <Icon icon="ep:download" class="mr-5px" />
          导入
        </el-button>
      </el-form-item>
    </el-form>
  </ContentWrap>

  <!-- 列表 -->
  <ContentWrap>
    <el-table v-loading="loading" :data="list" :stripe="true" :show-overflow-tooltip="true">
      <el-table-column label="id" align="center" prop="id" />
      <el-table-column label="群组id" align="center" prop="cardId" />
      <el-table-column label="创建人id" align="center" prop="createdUserId" />
      <el-table-column label="标题" align="center" prop="title" />
      <el-table-column label="usernames" align="center" prop="usernames">
        <template #default="scope">
          <div class="card-list" v-for="tag in scope.row.usernames" :key="tag">
            <el-tag :key="tag" round class="ml-2px">
              {{ tag }}
            </el-tag>
          </div>
        </template>
      </el-table-column>
      <el-table-column label="群组类型" align="center" prop="type">
        <template #default="scope">
          <el-tag v-if="scope.row.type">{{ scope.row.type }}</el-tag>
        </template>
      </el-table-column>
      <el-table-column label="人数" align="center" prop="memberCount" />
      <el-table-column label="是否公开" align="center" prop="isPublic" />
      <el-table-column label="频道说明" align="center" prop="description" />
      <el-table-column label="数据状态" align="center" prop="status">
        <template #default="scope">
          <dict-tag :type="DICT_TYPE.COLLECT_GROUP_STATUS" :value="scope.row.status" />
        </template>
      </el-table-column>
      <el-table-column label="权重" align="center" prop="manualWeigh">
        <template #default="scope">
          <el-tag v-if="scope.row.manualWeigh">{{ scope.row.manualWeigh }}</el-tag>
        </template>
      </el-table-column>
      <el-table-column
        label="创建时间"
        align="center"
        prop="createTime"
        :formatter="dateFormatter"
        width="180px"
      />
      <el-table-column label="操作" align="center" min-width="120px">
        <template #default="scope">
          <el-button
            link
            type="primary"
            @click="openForm('update', scope.row.id)"
            v-hasPermi="['collect:group:update']"
          >
            编辑
          </el-button>
          <el-button
            link
            type="danger"
            @click="handleDelete(scope.row.id)"
            v-hasPermi="['collect:group:delete']"
          >
            删除
          </el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- 分页 -->
    <Pagination
      :total="total"
      v-model:page="queryParams.pageNo"
      v-model:limit="queryParams.pageSize"
      @pagination="getList"
    />
  </ContentWrap>

  <!-- 表单弹窗：添加/修改 -->
  <GroupForm ref="formRef" @success="getList" />
  <GroupImportFrom ref="importFormRef" @success="getList" />
</template>

<script setup lang="ts">
import { dateFormatter } from '@/utils/formatTime'
import download from '@/utils/download'
import { GroupApi, GroupVO } from '@/api/collect/bot/group'
import GroupForm from './GroupForm.vue'
import GroupImportFrom from './GroupImportFrom.vue'
import { DICT_TYPE, getIntDictOptions } from '@/utils/dict'

/** 群组信息 列表 */
defineOptions({ name: 'CollectGroup' })

const message = useMessage() // 消息弹窗
const { t } = useI18n() // 国际化

const loading = ref(true) // 列表的加载中
const list = ref<GroupVO[]>([]) // 列表的数据
const total = ref(0) // 列表的总页数
const queryParams = reactive({
  pageNo: 1,
  pageSize: 10,
  cardId: undefined,
  createdUserId: undefined,
  title: undefined,
  usernames: undefined,
  type: undefined,
  memberCount: undefined,
  description: undefined,
  status: undefined,
  isPublic: undefined,
  manualWeigh: undefined,
  createTime: []
})
const queryFormRef = ref() // 搜索的表单
const exportLoading = ref(false) // 导出的加载中

/** 查询列表 */
const getList = async () => {
  loading.value = true
  try {
    const data = await GroupApi.getGroupPage(queryParams)
    list.value = data.list
    total.value = data.total
  } finally {
    loading.value = false
  }
}

/** 搜索按钮操作 */
const handleQuery = () => {
  queryParams.pageNo = 1
  getList()
}

/** 重置按钮操作 */
const resetQuery = () => {
  queryFormRef.value.resetFields()
  handleQuery()
}

/** 添加/修改操作 */
const formRef = ref()
const openForm = (type: string, id?: number) => {
  formRef.value.open(type, id)
}

/** 删除按钮操作 */
const handleDelete = async (id: number) => {
  try {
    // 删除的二次确认
    await message.delConfirm()
    // 发起删除
    await GroupApi.deleteGroup(id)
    message.success(t('common.delSuccess'))
    // 刷新列表
    await getList()
  } catch {}
}

/** 导出按钮操作 */
const handleExport = async () => {
  try {
    // 导出的二次确认
    await message.exportConfirm()
    // 发起导出
    exportLoading.value = true
    const data = await GroupApi.exportGroup(queryParams)
    download.excel(data, '群组信息.xls')
  } catch {
  } finally {
    exportLoading.value = false
  }
}
/** 用户导入 */
const importFormRef = ref()

const handleImport = () => {
  importFormRef.value.open()
}

/** 初始化 **/
onMounted(() => {
  getList()
})
</script>
