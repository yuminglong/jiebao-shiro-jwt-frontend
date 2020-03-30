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
                <a-input v-model.number="queryParams.ht_id" type="number" />
              </a-form-item>
            </a-col>
            <a-col :md="12" :sm="24" >
              <a-form-item
                label="待收款"
                :labelCol="{span: 4}"
                :wrapperCol="{span: 18, offset: 2}">
                <a-input v-model.number="queryParams.waitMoney" type="number"/>
              </a-form-item>
            </a-col>
            <template v-if="advanced">
              <a-col :md="12" :sm="24" >
                <a-form-item
                  label="签订日期"
                  :labelCol="{span: 4}"
                  :wrapperCol="{span: 18, offset: 2}">
                  <range-date @change="handleDateChange" ref="ht"></range-date>
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
               :rowKey="record => record.id"
               @change="handleTableChange">
      </a-table>
    </div>
  </a-card>
</template>

<script>
  import RangeDate from '../../../components/datetime/RangeDate'
  export default {
    name: 'Collect',
    components: {RangeDate},
    data () {
      return {
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
          dataIndex: 'ht_id'
        }, {
          title: '客户名称',
          dataIndex: 'ht_customerid'
        }, {
          title: '项目名称',
          dataIndex: 'ht_title'
        }, {
            title: '所有者',
            dataIndex: 'ht_preside'
        }, {
          title: '合同签订日期',
          dataIndex: 'ht_date'
        }, {
          title: '合同金额',
          dataIndex: 'ht_summoney'
        }, {
          title: '已收款金额',
          dataIndex: 'afterMoney'
        }, {
          title: '待收款金额',
          dataIndex: 'waitMoney'
        }]
      }
    },
    mounted () {
      this.fetch()
    },
    methods: {
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
      handleDateChange (value) {
        if (value) {
          this.queryParams.createTimeFrom = value[0]
          this.queryParams.createTimeTo = value[1]
        }
      },
    /*  exportExcel () {
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
          this.$refs.ht.reset()
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
        this.$get('/crm/contract-order', {
          ...params
        }).then((r) => {
          let data = r.data
          const pagination = {...this.pagination}
          this.dataSource = data
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
