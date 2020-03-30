<template>
  <a-modal
    v-model="show"
    :centered="true"
    :keyboard="false"
    :footer="null"
    :width="750"
    :visible="ticketInfoVisiable"
    @cancel="handleCancleClick"
    title="合同详情">
    <a-layout class="ticket-info" v-if="info">
      <a-layout-content class="ticket-content-one">
        <p><a-icon type="star"/>项目名称：{{contractData.htTitle ? contractData.htTitle : '无项目名称'}}</p>
        <p><a-icon type="user"/>对应客户：{{contractData.htCustomerid ? contractData.htCustomerid : '无对应客户'}}</p>
        <p><a-icon type="phone" />联系方式：{{contractData.htLxrinfo ? contractData.htLxrinfo : '暂无联系方式'}}</p>
        <p><a-icon type="clock-circle"/>签单日期：{{contractData.htDate ? contractData.htDate : '无对应客户'}}</p>
        <p><a-icon type="login"/>所有者：{{contractData.htPreside ? contractData.htPreside : '暂无所有者'}}</p>
        <p><a-icon type="bank"/>订单总金额：{{contractData.htSummoney ? contractData.htSummoney : '暂无对应总金额'}}</p>
        <p><a-icon type="carry-out"/>已收款金额：{{contractData.afterMoney ? contractData.afterMoney : '暂未收款'}}</p>
      </a-layout-content>
      <a-layout-content class="ticket-content-two">
        <p><a-icon type="carry-out"/>待收款金额：{{contractData.waitMoney ? contractData.waitMoney : '已全部收款'}}</p>
        <p><a-icon type="account-book"/>优惠抹零金额：{{contractData.htKjmoney ? contractData.htKjmoney : '无优惠'}}</p>
      </a-layout-content>
    </a-layout>
  </a-modal>
</template>
<script>
export default {
  name: 'ticketInfo',
  props: {
    data () {
      return {
        collect: {
          billParentid: ''
        }
      }
    },
    ticketInfoVisiable: {
      require: true,
      default: false
    },
    ticketInfoData: {
      require: true
    }
  },
  data () {
    return {
      contractData: {
      },
      info: true
    }
  },
  computed: {
    show: {
      get: function () {
        return this.ticketInfoVisiable
      },
      set: function () {
      }
    }
  },
  methods: {
    handleCancleClick () {
      this.$emit('close')
    }
  },
  watch: {
    ticketInfoData (data) {
      this.collect = data
      let billParentid = this.collect.billParentid
      if (this.ticketInfoVisiable) {
        this.$get(`/crm/contract-order/findContractOrderById/${billParentid}`).then((res) => {
          this.contractData = res.data
        })
      }
    }
  }
}
</script>
<style lang="less" scoped>
@import "TicketInfo";
</style>
