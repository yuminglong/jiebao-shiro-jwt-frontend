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
                    { required: true, message: '接口名称不能为空'}
                  ]}]"/>
      </a-form-item>

      <a-form-item label='对应实体' v-bind="formItemLayout">
        <a-input v-decorator="['bean',{rules: [{ required: true, message: '对应实体不能为空'}]}]"/>
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
const formItemLayout = {
  labelCol: {span: 3},
  wrapperCol: {span: 18}
}
export default {
  name: 'BasicEdit',
  props: {
    basicEditVisiable: {
      default: false
    }
  },
  data () {
    return {
      id: '',
      dataTypeId: '',
      loading: false,
      formItemLayout,
      form: this.$form.createForm(this)
    }
  },
  methods: {

    setFormValues ({...basic}) {
      this.dataTypeId = basic.dataTypeId
      this.id = basic.id
      let fields = ['dataName', 'bean', 'isUpdate']
      Object.keys(basic).forEach((key) => {
        if (fields.indexOf(key) !== -1) {
          this.form.getFieldDecorator(key)
          let obj = {}
          obj[key] = basic[key]
          this.form.setFieldsValue(obj)
        }
      })
      if (basic.isUpdate) {
        this.form.getFieldDecorator('isUpdate')
        this.form.setFieldsValue({'isUpdate': '1'})
      } else {
        this.form.getFieldDecorator('isUpdate')
        this.form.setFieldsValue({'isUpdate': '0'})
      }
    },
    reset () {
      this.loading = false
      this.form.resetFields()
    },
    onClose () {
      this.reset()
      this.$emit('close')
    },
    handleSubmit () {
      this.form.validateFields((err, values) => {
        if (!err) {
          this.loading = true
          let basic = this.form.getFieldsValue()
          basic.dataTypeId = this.dataTypeId
          basic.id = this.id
          this.$put('crm/data', {
            ...basic
          }).then((r) => {
            this.reset()
            this.$emit('success')
          }).catch(() => {
            this.loading = false
          })
        }
      })
    }

  }
}
</script>
