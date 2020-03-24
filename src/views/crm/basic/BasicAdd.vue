<template>
  <a-drawer
    title="修改按钮"
    :maskClosable="false"
    width=650
    placement="right"
    :closable="false"
    @close="onClose"
    :visible="basicEditVisiable"
    style="height: calc(100% - 55px);overflow: auto;padding-bottom: 53px;">
    <a-form
      :form="form">
      <a-form-item label='接口名称' v-bind="formItemLayout">
        <a-input v-decorator="['dataName',
                   {rules: [
                    { required: true, message: '接口名称不能为空'},
                    { max: 20, message: '长度不能超过20个字符'}
                  ]}]"/>
      </a-form-item>

      <a-form-item label='对应实体' v-bind="formItemLayout">
        <a-input v-decorator="['bean']"/>
      </a-form-item>

      <a-form-item label='需要更新' v-bind="formItemLayout">
        <a-radio-group
          v-decorator="[
            'isUpdate',
            {rules: [{ required: true, message: '请选择状态' }]}
          ]">
          <a-radio value="1">是</a-radio>
          <a-radio value="0">否</a-radio>
        </a-radio-group>
      </a-form-item>

    </a-form>
    <div class="drawer-bootom-button">
      <a-popconfirm title="确定放弃编辑？" @confirm="onClose" okText="确定" cancelText="取消">
        <a-button style="margin-right: .8rem">取消</a-button>
      </a-popconfirm>
      <a-button @click="handleSubmit" type="primary" :loading="loading">提交</a-button>
    </div>
  </a-drawer>
</template>

<script>
export default {
  name: 'BasicAdd'
}
</script>

<style lang="less" scoped>
  @import "../../../../static/less/Common";
</style>
