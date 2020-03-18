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
                <select  @change="handleDateChange"  ref="updateTime" style="width: 150px;height: 30px" >
                  <option label="2020" value="2020-12-01"> </option>
                  <option label="2019" value="2019-12-01"> </option>
                  <option label="2018" value="2018-12-01"> </option>
                  <option label="2017" value="2017-12-01"> </option>
                  <option label="2016" value="2016-12-01"> </option>
                </select>
              </a-form-item>
            </a-col>

          </a-row>
        </div>

        <span style="float: right; margin-top: 3px;">
          <a-button type="primary" @click="search">查询</a-button>
          <!--<a-button style="margin-left: 8px" @click="reset">重置</a-button>-->
        </span>
      </a-form>
    </div>
    <div>
      <div class="operator">
        <a-button v-hasPermission="'pay:export'" type="primary" ghost @click="exportExcel">导出Excel</a-button>
      </div>
      <!-- 表格区域 -->
      <!--:rowSelection="{selectedRowKeys: selectedRowKeys, onChange: onSelectChange}"-->
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
 // import RangeDate from '@/components/datetime/RangeDate'
  export default {
    name: 'Pay',
   // components: {RangeDate },
    data () {
      return {
        advanced: false,
        dataSource: [],
       // selectedRowKeys: [],
        queryParams: "2020-12-01",
        filteredInfo: null,
       paginationInfo: null,
        sortedInfo: null,
        pagination: {
          pageSizeOptions: ['10', '20', '30', '40', '100'],
          defaultCurrent: 1,
          defaultPageSize: 10,
          showQuickJumper: true,
          showSizeChanger: true,
          showTotal: (total, range) => `显示 ${range[0]} ~ ${range[1]} 条记录，共 ${total} 条记录`
        },
        loading: false,
      }
    },
    computed: {
      columns () {
        let { sortedInfo } = this
        sortedInfo = sortedInfo || {}
        return [{
          title: '支出时间',
          dataIndex: 'month1'
        },{
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


      // onSelectChange (selectedRowKeys) {
      //   this.selectedRowKeys = selectedRowKeys
      // },

      handleDateChange (value) {
          this.queryParams = value.target.value
      },
      exportExcel () {
        let {sortedInfo} = this
        let sortField, sortOrder
        // 获取当前列的排序和列的过滤规则
        if (sortedInfo) {
          sortField = sortedInfo.field
          sortOrder = sortedInfo.order
        }
        this.$export('crm/purchase/excel', {
          'queryParams':this.queryParams
        })
      },
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
      // this.selectedRowKeys = []
       // this.$refs.payTree.reset()
        // 重置分页
        // this.pagination.current = this.pagination.defaultCurrent
        // if (this.paginationInfo) {
        //   this.paginationInfo.current = this.pagination.defaultCurrent
        //   this.paginationInfo.pageSize = this.pagination.defaultPageSize
        // }
        // 重置列排序规则
        this.sortedInfo = null
        // 重置查询参数
        this.queryParams ="2020-12-01";
        // 清空时间选择
       // this.$refs.updateTime={}
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
        // if (this.paginationInfo) {
        //   // 如果分页信息不为空，则设置表格当前第几页，每页条数，并设置查询分页参数
        //   this.pagination.current = this.paginationInfo.current
        //   this.pagination.pageSize = this.paginationInfo.pageSize
        //   params.pageSize = this.paginationInfo.pageSize
        //   params.pageNum = this.paginationInfo.current
        // } else {
        //   // 如果分页信息为空，则设置为默认值
        //   params.pageSize = this.pagination.defaultPageSize
        //   params.pageNum = this.pagination.defaultCurrent
        // }
        console.log(this.queryParams);
        this.$get('crm/purchase/crmPay', {
          //...params

            'queryParams':this.queryParams



        }).then((r) => {
          let data = r.data
        //  const pagination = { ...this.pagination }
        //  pagination.total = data.total
          this.dataSource = data
          //this.pagination = pagination
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
