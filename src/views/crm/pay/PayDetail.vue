<template>

  <a-modal
    v-model="show"
    :centered="true"
    :keyboard="false"
    :footer="null"
    :width="1200"
    @cancel="handleCancleClick"
    title="开支详情">

    <a-card :bordered="false" class="card-area">
      <div :class="advanced ? 'search' : null">
        <!-- 搜索区域 -->
        <a-form layout="horizontal">
          <a-row >
            <div :class="advanced ? null: 'fold'">

              <a-col :md="12" :sm="24" >
                <a-form-item
                  label="开支类别"
                  :labelCol="{span: 4}"
                  :wrapperCol="{span: 18, offset: 2}">
                  <a-select v-model="choose">
                    <a-select-option value="1">
                      采购支出
                    </a-select-option>
                    <a-select-option value="2">
                      开支流水
                    </a-select-option>
                    <a-select-option value="3">
                      费用报销
                    </a-select-option>
                  </a-select>
                </a-form-item>
              </a-col>

            </div>
            <span style="float: right; margin-top: 3px;">
            <a-button type="primary" @click="search">查询</a-button>

          </span>
          </a-row>
        </a-form>
      </div>
      <div>
        <!-- 表格区域 -->
        <!-- 表格区域 -->
        <a-table :columns="columns"
                 :dataSource="dataSource"
                 :pagination="pagination"
                 :loading="loading"
                 :scroll="{ x: 900 }"
                 :rowKey="record => record.id"
        >
        </a-table>
      </div>

    </a-card>

  </a-modal>
</template>
<script>
export default {
  name: 'PayDetail',
  props: {
    payDetailVisiable: {
      require: true,
      default: false
    },
    payDetailData: {
      require: true
    }
  },
  data () {
    return {
      payDetail: {
        visiable: false,
        data: {}
      },
      choose: '',
      date: '',
      advanced: false,
      dataSource: [],
      queryParams: new Date().getFullYear() + '-03-01',
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
    show: {
      get: function () {
        return this.payDetailVisiable
      },
      set: function () {
      }
    },
    columns () {
      return [{
        title: '支出日期',
        dataIndex: '支出日期'
      }, {
        title: '对应客户/供应商',
        dataIndex: '对应客户'
      }, {
        title: '备注',
        dataIndex: '备注'
      }, {
        title: '开支人员',
        dataIndex: '开支人员'

      }, {
        title: '开支金额',
        dataIndex: '开支金额'

      }]
    }
  },

  methods: {
    handleCancleClick () {
      this.$emit('close')
    },
    search () {
      this.$get('crm/purchase/payDetail', {
        'date': this.date,
        'choose': this.choose
      }).then((r) => {
        let data = r.data
        this.dataSource = data
        this.loading = false
      })
    }
  },
  watch: {
    payDetailData (data) {
      this.choose = ''
      this.date = data.month1
      this.$get('crm/purchase/payDetail', {
        'date': data.month1
      }).then((r) => {
        let data = r.data
        this.dataSource = data
        this.loading = false
      })

      // let billParentid = this.collect.billParentid
      // if (this.ticketInfoVisiable) {
      //   this.$get(`/crm/contract-order/findContractOrderById/${billParentid}`).then((res) => {
      //     this.contractData = res.data
      //   })
      // }
    }
  }
}
</script>
<style lang="less" scoped>
@import "PayDetail";
</style>
