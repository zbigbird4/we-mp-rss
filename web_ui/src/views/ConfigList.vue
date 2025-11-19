<script setup lang="ts">
import { ref, reactive, onMounted } from 'vue'
import { 
  listConfigs, 
  createConfig, 
  updateConfig, 
  deleteConfig 
} from '@/api/configManagement'
import type { ConfigManagement } from '@/types/configManagement'
import { Modal, Message } from '@arco-design/web-vue'

const columns = [
  { title: '配置键', dataIndex: 'config_key', width: '25%' },
  { title: '配置值', dataIndex: 'config_value', width: '30%', ellipsis: true, tooltip: true },
  { title: '描述', dataIndex: 'description', width: '25%' },
  { title: '操作', slotName: 'action', width: '20%' }
]

const configList = ref<any>([])
const loading = ref(false)
const error = ref('')
const pagination = reactive({
  current: 1,
  pageSize: 10,
  total: 0
})

const visible = ref(false)
const modalTitle = ref('添加配置')
const form = reactive({
  config_key: '',
  config_value: '',
  description: ''
})

const fetchConfigs = async () => {
  try {
    loading.value = true
    const res= await listConfigs({
      page: pagination.current,
      pageSize: pagination.pageSize
    })
    configList.value = res.list
    pagination.total = res.total
  } catch (err) {
    error.value = err instanceof Error ? err.message : '获取配置列表失败'
  } finally {
    loading.value = false
  }
}

const showAddModal = () => {
  modalTitle.value = '添加配置'
  Object.keys(form).forEach(key => {
    form[key] = ''
  })
  visible.value = true
}

const editConfig = (record: ConfigManagement) => {
  modalTitle.value = '编辑配置'
  Object.assign(form, record)
  visible.value = true
}

const handleSubmit = async () => {
  try {
    if (modalTitle.value === '添加配置') {
      await createConfig(form)
      Message.success('配置添加成功')
    } else {
      await updateConfig(form.config_key, form)
      Message.success('配置更新成功，部分配置可能需要重启服务后生效')
    }
    visible.value = false
    fetchConfigs()
  } catch (err) {
    error.value = err instanceof Error ? err.message : '保存配置失败'
    Message.error(error.value)
  }
}

const deleteConfigItem = async (key: string) => {
  Modal.confirm({
    title: '确认删除',
    content: '确定要删除此配置吗？',
    okText: '删除',
    cancelText: '取消',
    onOk: async () => {
      try {
        await deleteConfig(key)
        fetchConfigs()
      } catch (err) {
        error.value = err instanceof Error ? err.message : '删除配置失败'
      }
    }
  })
}

const handlePageChange = (page: number) => {
  pagination.current = page
  fetchConfigs()
}

// 判断配置项是否受保护（不可编辑）
const hiddenConfigKeys = ['db', 'secret', 'token', 'notice.wechat', 'notice.feishu', 'notice.dingding', 'safe.lic_key']
const isHiddenConfig = (key: string) => {
  return hiddenConfigKeys.some(hiddenKey => key.includes(hiddenKey))
}

onMounted(() => {
  fetchConfigs()
})
</script>

<template>
  <div class="config-management">
    <a-card title="配置管理" :bordered="false">
      <a-space direction="vertical" fill>
        <a-alert type="info" show-icon closable>
          <template #icon>
            <icon-info-circle />
          </template>
          您可以在此页面查看和编辑系统配置。受保护的配置项（如数据库连接、密钥等）无法在此编辑。部分配置修改后需要重启服务才能生效。
        </a-alert>
        
        <a-alert v-if="error" type="error" show-icon>{{ error }}</a-alert>
        
        <a-table
          :columns="columns"
          :data="configList"
          :loading="loading"
          :pagination="pagination"
          @page-change="handlePageChange"
          row-key="config_key"
        >
          <template #action="{ record }">
            <a-space>
              <a-button 
                type="text" 
                size="small" 
                @click="editConfig(record)"
                v-if="!isHiddenConfig(record.config_key)"
              >
                编辑
              </a-button>
              <a-tag v-else color="gray">受保护</a-tag>
            </a-space>
          </template>
        </a-table>
      </a-space>
    </a-card>

    <a-modal
      v-model:visible="visible"
      :title="modalTitle"
      @ok="handleSubmit"
      @cancel="visible = false"
      width="600px"
    >
      <a-form :model="form" layout="vertical">
        <a-form-item label="配置键" field="config_key">
          <a-input v-model="form.config_key" disabled />
        </a-form-item>
        <a-form-item label="配置值" field="config_value" required>
          <a-textarea 
            v-model="form.config_value" 
            :auto-size="{ minRows: 2, maxRows: 6 }"
            placeholder="请输入配置值"
          />
        </a-form-item>
        <a-form-item label="描述" field="description">
          <a-input v-model="form.description" disabled />
        </a-form-item>
      </a-form>
    </a-modal>
  </div>
</template>

<style scoped>
.config-management {
  padding: 20px;
}
</style>
