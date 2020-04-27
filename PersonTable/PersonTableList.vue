<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <a-form-item label="公司">
              <a-input placeholder="请输入公司" v-model="queryParam.company"></a-input>
            </a-form-item>
          </a-col>
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <a-form-item label="关系ID">
              <a-select v-model="queryParam.relationship">
                <a-select-option value="1">
                  董事
                </a-select-option>
                <a-select-option value="2">
                  法人
                </a-select-option>
                <a-select-option value="3">
                  实际控股人
                </a-select-option>
              </a-select>
            </a-form-item>
          </a-col>
          <template v-if="toggleSearchStatus">
            <a-col :xl="6" :lg="7" :md="8" :sm="24">
              <a-form-item label="关系人姓名">
                <a-input placeholder="请输入关系人姓名" v-model="queryParam.relationshipPerson"></a-input>
              </a-form-item>
            </a-col>
            <a-col :xl="6" :lg="7" :md="8" :sm="24">
              <a-form-item label="人员类型">
                <a-checkbox-group v-model="queryParam.personTypeId">
                  <a-checkbox :value="1">主体</a-checkbox>
                </a-checkbox-group>
              </a-form-item>
            </a-col>
            <a-col :xl="6" :lg="7" :md="8" :sm="24">
              <a-form-item label="备注">
                <a-input placeholder="请输入备注" v-model="queryParam.remarks"></a-input>
              </a-form-item>
            </a-col>
          </template>
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
              <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
              <a-button type="primary" @click="searchReset" icon="reload" style="margin-left: 8px">重置</a-button>
              <a @click="handleToggleSearch" style="margin-left: 8px">
                {{ toggleSearchStatus ? '收起' : '展开' }}
                <a-icon :type="toggleSearchStatus ? 'up' : 'down'" />
              </a>
            </span>
          </a-col>
        </a-row>
      </a-form>
    </div>

    <!-- 操作按钮区域 -->
    <div class="table-operator">
      <a-button @click="handleAdd" type="primary" icon="plus">新增</a-button>
      <a-button type="primary" icon="download" @click="handleExportXls('人员信息表')">导出</a-button>
      <a-upload
        name="file"
        :showUploadList="false"
        :multiple="false"
        :headers="tokenHeader"
        :action="importExcelUrl"
        @change="handleImportExcel"
      >
        <a-button type="primary" icon="import">导入</a-button>
      </a-upload>
      <a-dropdown v-if="selectedRowKeys.length > 0">
        <a-menu slot="overlay">
          <a-menu-item key="1" @click="batchDel"><a-icon type="delete" />删除</a-menu-item>
        </a-menu>
        <a-button style="margin-left: 8px"> 批量操作 <a-icon type="down"/></a-button>
      </a-dropdown>
    </div>

    <!-- table区域-begin -->
    <div>
      <div class="ant-alert ant-alert-info" style="margin-bottom: 16px;">
        <i class="anticon anticon-info-circle ant-alert-icon"></i> 已选择
        <a style="font-weight: 600">{{ selectedRowKeys.length }}</a
        >项
        <a style="margin-left: 24px" @click="onClearSelected">清空</a>
      </div>

      <a-table
        ref="table"
        size="middle"
        bordered
        rowKey="id"
        :columns="columns"
        :dataSource="dataSource"
        :pagination="ipagination"
        :loading="loading"
        :rowSelection="{ selectedRowKeys: selectedRowKeys, onChange: onSelectChange }"
        @change="handleTableChange"
      >
        <span slot="action" slot-scope="text, record">
          <a @click="handleEdit(record)">编辑</a>

          <a-divider type="vertical" />
          <a-dropdown>
            <a class="ant-dropdown-link">更多 <a-icon type="down"/></a>
            <a-menu slot="overlay">
              <a-menu-item>
                <a-popconfirm title="确定删除吗?" @confirm="() => handleDelete(record.id)">
                  <a>删除</a>
                </a-popconfirm>
              </a-menu-item>
            </a-menu>
          </a-dropdown>
        </span>
      </a-table>
    </div>
    <!-- table区域-end -->

    <!-- 表单区域 -->
    <PersonTableModal ref="modalForm" @ok="modalFormOk"></PersonTableModal>
  </a-card>
</template>

<script>
import PersonTableModal from './modules/PersonTableModal'
import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import { getAction } from '@/api/manage'
export default {
  name: 'PERSON_TABLEList',
  mixins: [JeecgListMixin],
  components: {
    PersonTableModal
  },
  data() {
    return {
      description: '人员信息表管理页面',
      // 表头
      columns: [
        {
          title: '#',
          dataIndex: '',
          key: 'rowIndex',
          width: 60,
          align: 'center',
          customRender: function(t, r, index) {
            return parseInt(index) + 1
          }
        },
        {
          title: '公司',
          align: 'center',
          dataIndex: 'company'
        },
        {
          title: '关系ID(关联KEYWORD_TABLE的ID)',
          align: 'center',
          dataIndex: 'relationship'
        },
        {
          title: '关系人姓名',
          align: 'center',
          dataIndex: 'relationshipPerson'
        },
        {
          title: '人员类型ID(关联KEYWORD_TABLE的ID)',
          align: 'center',
          dataIndex: 'personTypeId'
        },
        {
          title: '备注',
          align: 'center',
          dataIndex: 'remarks'
        },
        {
          title: '时间戳',
          align: 'center',
          dataIndex: 'timeStamp'
        },
        {
          title: '删除标识',
          align: 'center',
          dataIndex: 'deleteIdentifier'
        },
        {
          title: '保留',
          align: 'center',
          dataIndex: 'reserve'
        },
        {
          title: '保留1',
          align: 'center',
          dataIndex: 'reserve1'
        },
        {
          title: '保留2',
          align: 'center',
          dataIndex: 'reserve2'
        },
        {
          title: '创建人ID',
          align: 'center',
          dataIndex: 'introductionId'
        },
        {
          title: '操作',
          dataIndex: 'action',
          align: 'center',
          scopedSlots: { customRender: 'action' }
        }
      ],
      url: {
        list: '/PERSON_TABLE/pERSON_TABLE/list',
        delete: '/PERSON_TABLE/pERSON_TABLE/delete',
        deleteBatch: '/PERSON_TABLE/pERSON_TABLE/deleteBatch',
        exportXlsUrl: 'PERSON_TABLE/pERSON_TABLE/exportXls',
        importExcelUrl: 'PERSON_TABLE/pERSON_TABLE/importExcel'
      }
    }
  },
  computed: {
    importExcelUrl: function() {
      return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`
    }
  },
  methods: {
    loadData(arg) {
      if (!this.url.list) {
        this.$message.error('请设置url.list属性!')
        return
      }
      //加载数据 若传入参数1则加载第一页的内容
      if (arg === 1) {
        this.ipagination.current = 1
      }
      var params = this.getQueryParams() //查询条件
      if (params.personTypeId) {
        params.personTypeId = params.personTypeId[0]
      }
      // alert(JSON.stringify(params))
      this.loading = true
      getAction(this.url.list, params).then(res => {
        if (res.success) {
          this.dataSource = res.result.records
          this.ipagination.total = res.result.total
        }
        if (res.code === 510) {
          this.$message.warning(res.message)
        }
        this.loading = false
      })
    },
    searchQuery() {
      this.loadData(1)
    }
  }
}
</script>
<style scoped>
@import '~@assets/less/common.less';
</style>
