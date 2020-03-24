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
                :labelCol="{span: 5}"
                :wrapperCol="{span: 18, offset: 1}">
                <!--<select  @change="handleDateChange"  ref="updateTime" style="width: 150px;height: 30px" >-->
                  <!--<option label="2020" value="2020-12-01"> </option>-->
                  <!--<option label="2019" value="2019-12-01"> </option>-->
                  <!--<option label="2018" value="2018-12-01"> </option>-->
                  <!--<option label="2017" value="2017-12-01"> </option>-->
                  <!--<option label="2016" value="2016-12-01"> </option>-->
                <!--</select>-->
                <select v-model="selected"  style="width: 150px;height: 30px" @change="handleDateChange">
                  <option v-for="option in options" v-bind:value="option.value">
                    {{ option.text }}
                  </option>
                </select>
              </a-form-item>
            </a-col>

          </a-row>
        </div>

        <span style="float: right; margin-top: 3px;">
          <a-button type="primary" @click="search">查询</a-button>
        </span>
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
      </a-table>
    </div>
  </a-card>
</template>

<script>
export default {
  name: 'Pay',
  data () {
    return {
      options: [],
      advanced: false,
      dataSource: [],
      queryParams: new Date().getFullYear() + '-12-01',
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
        title: '收支流水',
        dataIndex: '收支流水'
      }, {
        title: '费用报销',
        dataIndex: '费用报销'

      }, {
        title: '总计',
        dataIndex: '总计'

      }]
    }
  },
  mounted () {
    this.fetch()
  },
  methods: {

    handleDateChange (value) {
      this.queryParams = value.target.value
    },
    exportExcel () {
      this.$export('crm/purchase/excel', {
        'queryParams': this.queryParams
      })
    },
    search () {
      this.fetch({
        ...this.queryParams
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
          { text: new Date().getFullYear(), value: new Date().getFullYear() + '-12-01' },
          { text: new Date().getFullYear() - 1, value: new Date().getFullYear() - 1 + '-12-01' },
          { text: new Date().getFullYear() - 2, value: new Date().getFullYear() - 2 + '-12-01' },
          { text: new Date().getFullYear() - 3, value: new Date().getFullYear() - 3 + '-12-01' },
          { text: new Date().getFullYear() - 4, value: new Date().getFullYear() - 4 + '-12-01' },
          { text: new Date().getFullYear() - 5, value: new Date().getFullYear() - 5 + '-12-01' }
        ]
      })
    }
  }
}
</script>

<style lang="less" scoped>
  @import "../../../../static/less/Common";
</style>
