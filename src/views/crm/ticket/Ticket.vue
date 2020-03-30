<template>
  <a-card :bordered="false" class="card-area">
    <div :class="advanced ? 'search' : null">
      <!-- 搜索区域 -->
      <a-form layout="horizontal">
        <a-row >
          <div :class="advanced ? null: 'fold'">
            <a-col :md="12" :sm="24" >
              <a-form-item
                label="合同单号"
                :labelCol="{span: 4}"
                :wrapperCol="{span: 18, offset: 2}">
                <a-input v-model.number="queryParams.billParentid" type="number" ref="input"/>
              </a-form-item>
            </a-col>
            <a-col :md="12" :sm="24" >
              <a-form-item
                label="回款情况"
                :labelCol="{span: 4}"
                :wrapperCol="{span: 18, offset: 2}">
                <a-select v-model="queryParams.billHkstate">
                  <a-select-option value="未回款">
                    未回款
                  </a-select-option>
                  <a-select-option value="已回款">
                    已回款
                  </a-select-option>
                </a-select>
              </a-form-item>
            </a-col>
            <template v-if="advanced">
              <a-col :md="12" :sm="24" >
                <a-form-item
                  label="开票日期"
                  :labelCol="{span: 4}"
                  :wrapperCol="{span: 18, offset: 2}">
                  <range-date @change="handleDateChange" ref="billDate"></range-date>
                </a-form-item>
              </a-col>
            </template>
          </div>
          <span style="float: right; margin-top: 3px;">
            <a-button type="primary" @click="search">查询</a-button>
            <a-button style="margin-left: 8px" @click="reset">重置</a-button>
             <a @click="toggleAdvanced" style="margin-left: 8px">
              {{advanced ? '收起' : '展开'}}
              <a-icon :type="advanced ? 'up' : 'down'" />
            </a>
          </span>
        </a-row>
      </a-form>
    </div>
    <div>
      <!-- 表格区域 -->
      <a-table :columns="columns"
               :dataSource="dataSource"
               :pagination="pagination"
               :loading="loading"
               :scroll="{ x: 900 }"
               @change="handleTableChange">
        <template slot="operation" slot-scope="text,record">
          <a-icon  type="eye" theme="twoTone" twoToneColor="#42b983"
                   @click="view(record)" title="订单详情"></a-icon>
        </template>
      </a-table>
    </div>

    <ticket-info
      :ticketInfoData="ticketInfo.data"
      :ticketInfoVisiable="ticketInfo.visiable"
      @close="handleTicketInfoClose">
    </ticket-info>

  </a-card>
</template>

<script>
import RangeDate from '../../../components/datetime/RangeDate'
import TicketInfo from './TicketInfo'
export default {
  name: 'Ticket',
  components: {TicketInfo, RangeDate},
  data () {
    return {
      ticketInfo: {
        visiable: false,
        data: {}
      },
      advanced: false,
      dataSource: [],
      selectedRowKeys: [],
      queryParams: {},
      filteredInfo: null,
      sortedInfo: null,
      paginationInfo: null,
      pagination: {
        pageSizeOptions: ['10', '20', '30', '40', '100'],
        defaultCurrent: 1,
        defaultPageSize: 10,
        showQuickJumper: true,
        showSizeChanger: true,
        showTotal: (total, range) => `显示 ${range[0]} ~ ${range[1]} 条记录，共 ${total} 条记录`
      },
      loading: false
    }
  },
  computed: {
    columns () {
      return [{
        title: '合同单号',
        dataIndex: 'billParentid'
      }, {
        title: '开票金额',
        dataIndex: 'billMoney'
      }, {
        title: '开票号码',
        dataIndex: 'billNo'
      }, {
        title: '开票日期',
        dataIndex: 'billDate'
      }, {
        title: '回款/回款日期',
        dataIndex: 'billHkid'
      }, {
        title: '是否回款',
        dataIndex: 'billHkstate',
        customRender: (text) => {
          switch (text) {
            case '未回款':
              return <a-tag color="red">未回款</a-tag>
            case '已回款':
              return <a-tag color="cyan">已回款</a-tag>
            default:
              return text
          }
        }
      }, {
        title: '备注',
        dataIndex: 'billRemark'
      }, {
        title: '合同详情',
        dataIndex: 'operation',
        scopedSlots: { customRender: 'operation' }
      }
      ]
    }
  },
  mounted () {
    this.fetch()
  },
  methods: {
    handleTicketInfoClose () {
      this.ticketInfo.visiable = false
    },
    toggleAdvanced () {
      this.advanced = !this.advanced
      if (!this.advanced) {
        this.queryParams.createTimeFrom = ''
        this.queryParams.createTimeTo = ''
      }
    },
    onSelectChange (selectedRowKeys) {
      this.selectedRowKeys = selectedRowKeys
    },
    view (record) {
      this.ticketInfo.data = record
      this.ticketInfo.visiable = true
    },
    handleDateChange (value) {
      if (value) {
        this.queryParams.createTimeFrom = value[0]
        this.queryParams.createTimeTo = value[1]
      }
    },
    /* exportExcel () {
        let {sortedInfo} = this
        let sortField, sortOrder
        // 获取当前列的排序和列的过滤规则
        if (sortedInfo) {
          sortField = sortedInfo.field
          sortOrder = sortedInfo.order
        }
        this.$export('crm/data/excel', {
          sortField: sortField,
          sortOrder: sortOrder,
          ...this.queryParams
        })
      }, */
    search () {
      this.pagination.current = this.pagination.defaultCurrent
      if (this.paginationInfo) {
        this.paginationInfo.current = this.pagination.defaultCurrent
        this.paginationInfo.pageSize = this.pagination.defaultPageSize
      }
      let {sortedInfo} = this
      let sortField, sortOrder
      // 获取当前列的排序和列的过滤规则
      if (sortedInfo) {
        sortField = sortedInfo.field
        sortOrder = sortedInfo.order
      }
      this.fetch({
        sortField: sortField,
        sortOrder: sortOrder,
        ...this.queryParams
      })
    },
    reset () {
      // 取消选中
      this.selectedRowKeys = []
      // 重置分页
      this.pagination.current = this.pagination.defaultCurrent
      if (this.paginationInfo) {
        this.paginationInfo.current = this.pagination.defaultCurrent
        this.paginationInfo.pageSize = this.pagination.defaultPageSize
      }
      // 重置列排序规则
      this.sortedInfo = null
      // 重置查询参数
      this.queryParams = {}
      // 清空时间选择
      if (this.advanced) {
        this.$refs.billDate.reset()
      }
      this.fetch()
    },
    handleTableChange (pagination, filters, sorter) {
      this.paginationInfo = pagination
      this.filteredInfo = filters
      this.sortedInfo = sorter
      this.fetch({
        sortField: sorter.field,
        sortOrder: sorter.order,
        ...this.queryParams,
        ...filters
      })
    },
    fetch (params = {}) {
      this.loading = true
      if (this.paginationInfo) {
        // 如果分页信息不为空，则设置表格当前第几页，每页条数，并设置查询分页参数
        this.pagination.current = this.paginationInfo.current
        this.pagination.pageSize = this.paginationInfo.pageSize
        params.pageSize = this.paginationInfo.pageSize
        params.pageNum = this.paginationInfo.current
      } else {
        // 如果分页信息为空，则设置为默认值
        params.pageSize = this.pagination.defaultPageSize
        params.pageNum = this.pagination.defaultCurrent
      }
      this.$get('/crm/bill-record', {
        ...params
      }).then((r) => {
        let data = r.data
        const pagination = {...this.pagination}
        pagination.total = data.total
        this.dataSource = data.rows
        this.pagination = pagination
        // 数据加载完毕，关闭loading
        this.loading = false
      })
    }
  }
}
</script>

<style lang="less" scoped>
  @import "../../../../static/less/Common";
</style>
