<template>
  <a-card :bordered="false" class="card-area">
    <!--<div :class="advanced ? 'search' : null">-->
    <!--&lt;!&ndash; 搜索区域 &ndash;&gt;-->
    <!--<a-form layout="horizontal">-->
    <!--<div :class="advanced ? null: 'fold'">-->
    <!--<a-row >-->
    <!--<a-col :md="12" :sm="24" >-->
    <!--<a-form-item-->
    <!--label="接口名称"-->
    <!--:labelCol="{span: 5}"-->
    <!--:wrapperCol="{span: 18, offset: 1}">-->
    <!--<a-input v-model="queryParams.dataName"/>-->
    <!--</a-form-item>-->
    <!--</a-col>-->
    <!--<a-col :md="12" :sm="24" >-->
    <!--<a-form-item-->
    <!--label="更新时间"-->
    <!--:labelCol="{span: 5}"-->
    <!--:wrapperCol="{span: 18, offset: 1}">-->
    <!--<range-date @change="handleDateChange" ref="updateTime"></range-date>-->
    <!--</a-form-item>-->
    <!--</a-col>-->
    <!--</a-row>-->
    <!--</div>-->
    <!--<span style="float: right; margin-top: 3px;">-->
    <!--<a-button type="primary" @click="search">查询</a-button>-->
    <!--<a-button style="margin-left: 8px" @click="reset">重置</a-button>-->
    <!--</span>-->
    <!--</a-form>-->
    <!--</div>-->
    <div>
      <div class="operator">
        <select @change="selectByYear" style="width: 100px;height: 30px">
          <!--<option value="0">请选择年份</option>-->
          <!--<option value="2020">2020</option>-->
          <!--<option value="2019">2019</option>-->
          <!--<option value="2018">2018</option>-->
          <!--<option value="2017">2017</option>-->
          <!--<option value="2016">2016</option>-->
          <!--<option value="2015">2015</option>-->
          <!--<option value="2014">2014</option>-->
          <!--<option value="2013">2013</option>-->
          <!--<option value="2012">2012</option>-->
          <!--<option value="2011">2011</option>-->
          <!--<option value="2010">2010</option>-->
          <option v-for="option in options" v-bind:aria-valuemax="option.value">{{option.text}}</option>
          <option @change="setOptions">更多年份</option>
        </select>
        <select @change="selectByMonth" style="width: 100px;height: 30px">
          <option value="0">请选择月份</option>
          <option value="1">1</option>
          <option value="2">2</option>S
          <option value="3">3</option>
          <option value="4">4</option>
          <option value="5">5</option>
          <option value="6">6</option>
          <option value="7">7</option>
          <option value="8">8</option>
          <option value="9">9</option>
          <option value="10">10</option>
          <option value="11">11</option>
          <option value="12">12</option>
        </select>
        <a-dropdown v-hasPermission="'sell:excel'">
          <a-menu slot="overlay">
            <a-menu-item key="export-data" @click="exportExcel">导出Excel</a-menu-item>
          </a-menu>
          <a-button>
            更多操作
            <a-icon type="down"/>
          </a-button>
        </a-dropdown>
      </div>
      <!-- 表格区域 -->
      <a-table :columns="columns"
               :dataSource="dataSource"
               :pagination="pagination"
               :loading="loading"
               :scroll="{ x: 900 }"
               :rowSelection="{selectedRowKeys: selectedRowKeys, onChange: onSelectChange}"
               :rowKey="record => record.id"
               @change="handleTableChange">
        <template slot="operation" slot-scope="text, record">
          <a-icon v-hasPermission="'basic:update'" type="setting" theme="twoTone" twoToneColor="#4a9ff5"
                  @click="edit(record)" title="修改"></a-icon>
          <a-badge v-hasNoPermission="'basic:update'" status="warning" text="无权限"></a-badge>
        </template>
      </a-table>
    </div>
    <!-- 操作
    <basic-add
      @success="handleBasicAddSuccess"
      @close="handleBasicAddClose"
      :basicAddVisiable="basicAddVisiable">
    </basic-add>
    <basic-edit
      ref="basicEdit"
      @success="handleBasicEditSuccess"
      @close="handleBasicEditClose"
      :basicEditVisiable="basicEditVisiable">
    </basic-edit>
    -->
  </a-card>
</template>

<script>
// nowDate = new Date
export default {
  name: 'Sell',
  // components: {BasicAdd, BasicEdit, RangeDate},
  data () {
    return {
      year: null,
      month: null,
      advanced: false,
      dataSource: [],
      num: 0,
      selectedRowKeys: [],
      queryParams: {},
      filteredInfo: null,
      paginationInfo: null,
      sortedInfo: null,
      pagination: {
        pageSizeOptions: ['2', '4', '6', '8', '8'],
        defaultCurrent: 1,
        defaultPageSize: 10,
        showQuickJumper: true,
        showSizeChanger: true,
        showTotal: (total, range) => `显示 ${range[0]} ~ ${range[1]} 条记录，共 ${total} 条记录`
      },
      loading: false,
      basicAddVisiable: false,
      basicEditVisiable: false
    }
  },
  computed: {
    columns () {
      return [{
        title: '合同ID',
        dataIndex: 'id'
      }, {
        title: '合同名称',
        dataIndex: 'htName'
      }, {
        title: '合同金额',
        dataIndex: 'htSumMoney'
      }, {
        title: '签订时间',
        dataIndex: 'htDate'
        // customRender: (text) => {
        //   if (text) {
        //     return <a-tag color="red">是</a-tag>
        //   } else {
        //     return <a-tag color="cyan">否</a-tag>
        //   }
        // }
      }, {
        title: '采购费用',
        dataIndex: 'cgMoney'
        // sorter: true,
        // sortOrder: sortedInfo.columnKey === 'updateTime' && sortedInfo.order
      }, {
        title: '报销费用',
        dataIndex: 'bxMoney'
        // scopedSlots: { customRender: 'operation' },
        // fixed: 'right',
        // width: 120
      }, {
        title: '人员',
        dataIndex: 'htPerson'
        // scopedSlots: { customRender: 'operation' },
        // fixed: 'right',
        // width: 120
      }, {
        title: '已开票金额',
        dataIndex: 'htKpMoney'
        // scopedSlots: { customRender: 'operation' },
        // fixed: 'right',
        // width: 120
      }, {
        title: '税金',
        dataIndex: 'htSjMoney'
        // scopedSlots: { customRender: 'operation' },
        // fixed: 'right',
        // width: 120
      },
      {
        title: '费用总计',
        dataIndex: 'htFyMoney'
        // scopedSlots: { customRender: 'operation' },
        // fixed: 'right',
        // width: 120
      },
      {
        title: '应收账款',
        dataIndex: 'htYsMoney'
        // scopedSlots: { customRender: 'operation' },
        // fixed: 'right',
        // width: 120
      },
      {
        title: '逾期应收账款',
        dataIndex: 'htYqMoney'
        // scopedSlots: { customRender: 'operation' },
        // fixed: 'right',
        // width: 120
      }
      ]
    }
  },
  mounted () {
    this.Date = new Date()
    this.year = this.Date.getFullYear()
    this.month = this.Date.getMonth() + 1
    this.setOptions()
    this.fetch()
  },
  methods: {
    setOptions () {
      this.options = [
        { text: new Date().getFullYear() - (this.num++), value: new Date().getFullYear() - (this.num) },
        { text: new Date().getFullYear() - (this.num++), value: new Date().getFullYear() - (this.num) },
        { text: new Date().getFullYear() - (this.num++), value: new Date().getFullYear() - (this.num) },
        { text: new Date().getFullYear() - (this.num++), value: new Date().getFullYear() - (this.num) },
        { text: new Date().getFullYear() - (this.num++), value: new Date().getFullYear() - (this.num) },
        { text: new Date().getFullYear() - (this.num++), value: new Date().getFullYear() - (this.num) }
      ]
    },
    onSelectChange (selectedRowKeys) {
      this.selectedRowKeys = selectedRowKeys
    },
    // handleBasicAddClose () {
    //   this.BasicAddVisiable = false
    // },
    // handleBasicAddSuccess () {
    //   this.BasicAddVisiable = false
    //   this.$message.success('新增接口Basic成功')
    //   this.fetch()
    // },
    // add () {
    //   this.basicEditVisiable = true
    // },
    // handleBasicEditClose () {
    //   this.basicEditVisiable = false
    // },
    // handleBasicEditSuccess () {
    //   this.basicEditVisiable = false
    //   this.$message.success('修改接口成功')
    //   this.fetch()
    // },
    // edit (record) {
    //   this.basicEditVisiable = true
    //   this.$refs.deptEdit.setFormValues(record)
    // },

    exportExcel () {
      let {sortedInfo} = this
      let sortField, sortOrder
      // 获取当前列的排序和列的过滤规则
      if (sortedInfo) {
        sortField = sortedInfo.field
        sortOrder = sortedInfo.order
      }
      this.$export('crm/contract-order/excel', {
        sortField: sortField,
        sortOrder: sortOrder,
        year: this.year,
        month: this.month,
        pageSize: this.pagination.defaultPageSize,
        pageNum: this.pagination.defaultCurrent
        // ...this.queryParams
      })
    },
    selectByMonth (value) {
      this.month = value.target.value
      this.fetch()
    },
    selectByYear (value) {
      this.year = value.target.value
      this.fetch()
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
    // reset () {
    //   // 取消选中
    //   this.selectedRowKeys = []
    //   // 重置分页
    //   this.pagination.current = this.pagination.defaultCurrent
    //   if (this.paginationInfo) {
    //     this.paginationInfo.current = this.pagination.defaultCurrent
    //     this.paginationInfo.pageSize = this.pagination.defaultPageSize
    //   }
    //   // 重置列排序规则
    //   this.sortedInfo = null
    //   // 重置查询参数
    //   this.queryParams = {}
    //   // 清空时间选择
    //   this.$refs.updateTime.reset()
    //   this.fetch()
    // },
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
      // this.$get('crm/data/crmDataList', {
      //   ...params
      this.$get('crm/contract-order/List', {
        //  ...params
        month: this.month,
        year: this.year
      }).then((r) => {
        console.log(r)
        let data = r.data
        const pagination = {...this.pagination}
        pagination.total = data.total
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
