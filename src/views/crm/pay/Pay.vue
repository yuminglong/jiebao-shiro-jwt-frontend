<template>
  <a-card :bordered="false" class="card-area">
    <div :class="advanced ? 'search' : null">
      <!-- 搜索区域 -->
      <a-form layout="horizontal">
        <div :class="advanced ? null: 'fold'">
          <a-row >
            <a-col :md="12" :sm="24" >
              <a-form-item
                label="查询年份"
                :labelCol="{span: 3}"
                :wrapperCol="{span: 10, offset: 1}">
                <a-select @change="handleDateChange" v-model="queryParams">
                  <a-select-option v-for="option in options" :key="option.value"  v-bind:value ="option.value">
                    {{ option.text }}
                  </a-select-option>
                </a-select>
                  </a-form-item>
                </a-col>
          </a-row>
        </div>
      </a-form>
    </div>
    <div>
      <div class="operator">
        <a-button v-hasPermission="'pay:export'" type="primary" ghost @click="exportExcel">导出Excel</a-button>
      </div>
      <!-- 表格区域 -->
      <a-table :columns="columns"
               :dataSource="dataSource"
               :pagination="pagination"
               :loading="loading"
               :scroll="{ x: 900 }"
               :rowKey="record => record.id"
               @change="handleTableChange">
        <template slot="operation" slot-scope="text,record">
          <a-icon type="switcher" theme="twoTone" twoToneColor="#4a9ff5" @click="view(record)" title="查看"></a-icon>
        </template>
      </a-table>
    </div>
    <!-- 开支明细查看 -->
    <pay-detail
      :payDetailData="payDetail.data"
      :payDetailVisiable="payDetail.visiable"
      @close="handlePayDetailClose">
    </pay-detail>
  </a-card>
</template>

<script>
import PayDetail from './PayDetail'
export default {
  name: 'Pay',
  components: {PayDetail},
  data () {
    return {
      payDetail: {
        visiable: false,
        data: {}
      },
      options: [],
      advanced: false,
      dataSource: [],
      queryParams: new Date().getFullYear(),
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
        title: '支出时间',
        dataIndex: 'month1'
      }, {
        title: '采购支出',
        dataIndex: '采购支出'
      }, {
        title: '开支流水',
        dataIndex: '收支流水'
      }, {
        title: '费用报销',
        dataIndex: '费用报销'

      }, {
        title: '总计',
        dataIndex: '总计'

      }, {
        title: '操作',
        dataIndex: 'operation',
        scopedSlots: { customRender: 'operation' }
      }]
    }
  },
  mounted () {
    this.fetch()
  },
  methods: {
    view (record) {
      this.payDetail.data = record
      this.payDetail.visiable = true
    },
    handlePayDetailClose () {
      this.payDetail.visiable = false
    },
    handleDateChange () {
      this.fetch({
        ...this.queryParams
      })
    },
    exportExcel () {
      this.$export('crm/purchase/excel', {
        'queryParams': this.queryParams
      })
    },
    reset () {
      // 重置列排序规则
      this.sortedInfo = null
      // 重置查询参数
      this.queryParams = '2020-12-01'
      this.fetch()
    },
    handleTableChange (pagination, filters, sorter) {
      this.fetch({
        sortField: sorter.field,
        sortOrder: sorter.order,
        ...this.queryParams,
        ...filters
      })
    },
    fetch (params = {}) {
      this.loading = true
      this.$get('crm/purchase/crmPay', {
        'queryParams': this.queryParams
      }).then((r) => {
        let data = r.data
        this.dataSource = data
        this.loading = false
        new Date().getFullYear()
        this.options = [
          { text: new Date().getFullYear(), value: new Date().getFullYear() },
          { text: new Date().getFullYear() - 1, value: new Date().getFullYear() - 1 },
          { text: new Date().getFullYear() - 2, value: new Date().getFullYear() - 2 },
          { text: new Date().getFullYear() - 3, value: new Date().getFullYear() - 3 },
          { text: new Date().getFullYear() - 4, value: new Date().getFullYear() - 4 },
          { text: new Date().getFullYear() - 5, value: new Date().getFullYear() - 5 }
        ]
      })
    }
  }
}
</script>

<style lang="less" scoped>
  @import "../../../../static/less/Common";
</style>
