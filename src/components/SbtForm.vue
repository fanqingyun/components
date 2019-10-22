<template>
  <div>
    <el-form
      :model="form"
      :rules="rules"
      ref="form"
      :label-width="labelWidth"
      size="small"
      :disabled="disabledState"
    >
      <slot name="header"></slot>
      <template v-for="(item, index) in formInfo">
        <el-form-item
          :key="index"
          :label="item.label"
          :required="item.required"
          v-if="!item.showCondition || (form[item.showCondition.name] === item.showCondition.value)"
        >
          <template v-for="(subItem, key) in item.components">
            <el-col
              :span="subItem.spanLength?subItem.spanLength:24"
              :key="key"
              v-if="!subItem.showCondition || (form[subItem.showCondition.name] === subItem.showCondition.value)"
            >
              <el-form-item
                :prop="''+subItem.name"
                :rule="subItem.rule"
                v-if="subItem.type!=='others'"
              >
                <!-- 单选框组 -->
                <el-radio-group
                  v-model="form[subItem.name]"
                  v-if="subItem.type==='radio'"
                  @change="radioGroupChange(subItem)"
                >
                  <el-radio
                    v-for="(option, key) in subItem.options"
                    :key="key"
                    :label="option.value?option.value:option.value"
                    :disabled="option.disabled"
                  >{{option.label}}</el-radio>
                </el-radio-group>
                <el-radio-group
                  v-model="form[subItem.name]"
                  v-if="subItem.type==='radiobutton'"
                  @change="radioBtnGroupChange(subItem)"
                >
                  <el-radio-button
                    v-for="(option, key) in subItem.options"
                    :key="key"
                    :label="option.value"
                    :disabled="option.disabled"
                  >{{option.label}}</el-radio-button>
                </el-radio-group>
                <!-- 复选框组 -->
                <el-checkbox-group
                  v-model="form[subItem.name]"
                  v-if="subItem.type==='checkbox' && subItem.options.length>1"
                  @change="checkboxGroupChange(subItem)"
                >
                  <el-checkbox
                    v-for="(option, key) in subItem.options"
                    :key="key"
                    :label="option.value"
                    :disabled="option.disabled"
                    :checked="option.checked"
                  ></el-checkbox>
                </el-checkbox-group>
                <el-checkbox
                  v-model="form[subItem.name]"
                  :disabled="subItem.options[0].disabled"
                  :checked="subItem.options[0].checked"
                  v-if="subItem.type==='checkbox' && subItem.options.length===1"
                  @change="checkboxChange(subItem)"
                >{{subItem.options[0].label}}</el-checkbox>
                <el-checkbox-group
                  v-model="form[subItem.name]"
                  v-if="subItem.type==='checkboxbutton'"
                  @change="checkboxBtnGroupChange(subItem)"
                >
                  <el-checkbox-button
                    v-for="(option, key) in subItem.options"
                    :key="key"
                    :label="option.value"
                    :disabled="option.disabled"
                    :checked="option.checked"
                  ></el-checkbox-button>
                </el-checkbox-group>
                <!-- 下拉框 -->
                <el-select
                  v-model="form[subItem.name]"
                  :placeholder="'请选择'+ (subItem.desc=subItem.desc? subItem.desc:item.label)"
                  v-if="subItem.type==='select'"
                  :multiple="subItem.multiple"
                  :filterable="subItem.filterable"
                  clearable
                  :disabled="subItem.disabled"
                  @change="changeSelect(subItem)"
                  @click.native="selectClick(subItem)"
                  style="width: 100%;"
                >
                  <el-option
                    v-for="(option, key) in subItem.options"
                    :key="key"
                    :label="option.label"
                    :value="option.value"
                  ></el-option>
                </el-select>
                <!-- input：密码框，文件，普通文本输入框，按钮等 -->
                <!-- 文本输入框 -->
                <el-input
                  type="textarea"
                  v-model="form[subItem.name]"
                  :placeholder="'请输入'+(subItem.desc=subItem.desc? subItem.desc:item.label)"
                  v-if="subItem.type==='textarea'"
                  clearable
                  :disabled="subItem.disabled"
                ></el-input>
                <!-- 文件 -->
                <el-input type="file" v-model="form[subItem.name]" v-if="subItem.type==='file'"></el-input>
                <!-- 普通输入框 -->
                <el-input
                  type="text"
                  v-model="form[subItem.name]"
                  :placeholder="'请输入'+(subItem.desc=subItem.desc? subItem.desc:item.label)"
                  v-if="subItem.type==='text'"
                  clearable
                  :disabled="subItem.disabled"
                >
                  <template slot="prepend" v-if="subItem.prepend">{{subItem.prepend}}</template>
                  <template slot="append" v-if="subItem.append">{{subItem.append}}</template>
                </el-input>
                <!-- 密码输入框 -->
                <el-input
                  type="password"
                  v-model="form[subItem.name]"
                  :placeholder="'请输入'+(subItem.desc=subItem.desc? subItem.desc:item.label)"
                  v-if="subItem.type==='password'"
                  clearable
                  :disabled="subItem.disabled"
                ></el-input>
                <!-- 按钮 由于name是唯一的。通过name来识别不同按钮的事件,若通过按钮来控制某些字段的显示，由于按钮的值在父组件赋予，所以showCondition的value值应为基本类型，避免引用造成不等-->
                <el-button
                  :icon="subItem.icon"
                  @click="buttonClick(subItem)"
                  v-if="subItem.type==='button'"
                  :class="!subItem.class?'primary':subItem.class"
                  :style="!subItem.style?'border:0px solid #fff; color: #409eff;':subItem.style"
                >{{subItem.desc}}</el-button>
                <!-- 数字计数器 -->
                <el-input-number
                  v-model="form[subItem.name]"
                  :placeholder="'请输入'+(subItem.desc=subItem.desc? subItem.desc:item.label)"
                  v-if="subItem.type==='number'"
                  controls-position="right"
                  :min="subItem.min?subItem.min:-Infinity"
                  :max="subItem.max?subItem.max:+Infinity"
                  :step="subItem.step?subItem.step:1"
                  :size="subItem.size?subItem.size:'small'"
                  :disabled="subItem.disabled"
                ></el-input-number>
                <!-- 级联 -->
                <el-cascader
                  v-model="form[subItem.name]"
                  v-if="subItem.type==='cascader'"
                  :options="subItem.options"
                  :show-all-levels="subItem.showAllLevel"
                  :filterable="subItem.filterable"
                  clearable
                  :disabled="subItem.disabled"
                  :change-on-select="subItem.changeOnSelect"
                ></el-cascader>
                <!-- 开关转换器 -->
                <el-switch
                  v-model="form[subItem.name]"
                  v-if="subItem.type==='switch'"
                  :active-color="subItem.activeColor?subItem.activeColor:'#13ce66'"
                  :inactive-color="subItem.inactiveColor?subItem.inactiveColor:'#ff4949'"
                  :active-text="subItem.activeText?subItem.activeText:''"
                  :inactive-text="subItem.inactiveText?subItem.inactiveText:''"
                  :active-value="subItem.activeValue?subItem.activeValue:true"
                  :inactive-value="subItem.inactiveValue?subItem.inactiveValue:false"
                ></el-switch>
                <!-- 滑块 -->
                <el-slider
                  v-model="form[subItem.name]"
                  v-if="subItem.type==='slider'"
                  :show-input="subItem.showInput"
                  :range="subItem.range"
                  :min="subItem.min?subItem.min:0"
                  :max="subItem.max?subItem.max:100"
                  :step="subItem.step?subItem.step:1"
                  :disabled="subItem.disabled"
                  :show-stops="subItem.showStops"
                ></el-slider>
                <!-- 日期/时间日期 -->
                <el-date-picker
                  type="date"
                  :placeholder="'选择' + (subItem.desc=subItem.desc? subItem.desc:item.label)"
                  v-model="form[subItem.name]"
                  v-if="subItem.type==='date'"
                  :format="subItem.format ? subItem.format : 'yyyy-MM-dd'"
                  :disabled="subItem.disabled"
                ></el-date-picker>
                <el-date-picker
                  type="datetime"
                  :placeholder="'选择' + (subItem.desc=subItem.desc? subItem.desc:item.label)"
                  v-model="form[subItem.name]"
                  v-if="subItem.type==='datetime'"
                  :format="subItem.format ? subItem.format:'yyyy-MM-dd HH:mm:ss'"
                  :disabled="subItem.disabled"
                ></el-date-picker>
                <el-date-picker
                  type="daterange"
                  start-placeholder="开始日期"
                  end-placeholder="结束日期"
                  v-model="form[subItem.name]"
                  v-if="subItem.type==='daterange'"
                  :format="subItem.format?subItem.format:'yyyy-MM-dd'"
                  :disabled="subItem.disabled"
                ></el-date-picker>
                <el-date-picker
                  type="datetimerange"
                  start-placeholder="开始日期"
                  end-placeholder="结束日期"
                  v-model="form[subItem.name]"
                  v-if="subItem.type==='datetimerange'"
                  :format="subItem.format?subItem.format:'yyyy-MM-dd HH:mm:ss'"
                  :disabled="subItem.disabled"
                ></el-date-picker>
                <!-- 时间: 此处应添加公司自己封装的时间控件（待完善） -->
                <el-time-picker
                  type="time"
                  :placeholder="'选择' + (subItem.desc=subItem.desc? subItem.desc:item.label)"
                  v-model="form[subItem.name]"
                  :picker-options="subItem.options"
                  v-if="subItem.type==='time'"
                  :disabled="subItem.disabled"
                  :is-range="subItem.isRange"
                  start-placeholder="开始时间"
                  end-placeholder="结束时间"
                ></el-time-picker>
                <!-- 评分 -->
                <el-rate
                  v-model="form[subItem.name]"
                  :colors="['#99A9BF', '#F7BA2A', '#FF9900']"
                  :show-text="subItem.showText"
                  v-if="subItem.type==='rate'"
                  :disabled="subItem.disabled"
                  :allow-half="subItem.allowHalf"
                ></el-rate>
                <!-- 颜色选择器 -->
                <el-color-picker
                  v-model="form[subItem.name]"
                  v-if="subItem.type==='color'"
                  show-alpha
                  :size="subItem.size?subItem.size:'small'"
                ></el-color-picker>
                <!-- 穿梭框 -->
                <el-transfer
                  v-model="form[subItem.name]"
                  :data="subItem.options"
                  v-if="subItem.type==='transfer'"
                  :filterable="subItem.filterable"
                  :titles="subItem.titles?subItem.titles:['列表一', '列表二']"
                ></el-transfer>
                <!-- 树形单元 -->
                <!-- <el-tree v-if="subItem.type==='tree'"></el-tree> -->
                <!-- 文件上传 -->
                <!-- <el-upload v-if="subItem.type==='upload'"></el-upload> -->
              </el-form-item>
              <!-- 非表单控件非弹框 -->
              <span v-if="subItem.type==='others'" class="othersClass" v-html="subItem.desc"></span>
              <!-- 弹框说明 -->
              <el-popover
                v-if="subItem.type==='popover'"
                placement="right-start"
                title
                width="400"
                popper-class="popper-class"
                trigger="click"
              >
                <p v-html="subItem.desc"></p>
                <el-button
                  slot="reference"
                  style="border:0px solid #fff; color: #409eff;"
                >{{subItem.btnDesc}}</el-button>
              </el-popover>
              <!--<component :is='subItem.desc' v-if="subItem.type==='others'"></component> -->
              <!-- <template v-if="subItem.type==='others'">{{subItem.desc}}</template>-->
            </el-col>
          </template>
          <p
            class="comment"
            v-if="item.comment"
            style="font-size: 10px; line-height: 15px; padding-top: 5px; padding-bottom: 5px;float: left;"
            v-html="item.comment"
          ></p>
        </el-form-item>
      </template>
      <slot name="footer"></slot>
      <!-- <el-form-item style="margin-left: 60%">
    <el-button type="primary" @click="submitForm">保存</el-button>
    <el-button @click="resetForm">取消</el-button>
      </el-form-item>-->
    </el-form>
    <div class="form-btn">
      <el-button type="primary" @click="submitForm" size="small" v-if="!disabledState">保存</el-button>
      <el-button @click="resetForm" size="small">取消</el-button>
    </div>
  </div>
</template>

<script>
// import SbtTimePicker from '@/components/sbtTimePicker/SbtTimePicker'
export default {
  /**
   * @param formIfo 表单配置信息 {
      label: 显示在前端的名称 string 必填
      components:[] 表单控件格式 array 必填
      require: （可选）显示必填*号 boolean， 只是显示*号，同时规则里也应设置required为true
      comment:（可选）显示在控件底下的文字说明 string
      showCondition （可选） 控件显示的条件 根据表单某个字段的值是否进行显示 可选 object { name: '', value: ''}
      }
      components里每个控件的name的值，即为form的属性
   * @param rules 表单验证规则 Object
   * @param labelWidth 表单域标签的宽度 String 可选
   * @param form 接收值的表单，以传给父组件, 可进行初始化 Object
   * slot为插槽，针对一些带有事件的或比较复杂的表单事件
   */
  props: {
    formInfo: {
      type: Array,
      default: function () {
        return []
      }
    },
    rules: {
      type: Object,
      default: function () {
        return {}
      }
    },
    labelWidth: {
      type: [Number, String],
      default: '25%'
    },
    form: {
      type: Object,
      default: function () {
        return {}
      }
    },
    disabledState: {
      type: Boolean,
      default: function () {
        return false
      }
    }
  },
  data () {
    return {
      initForm: {}
    }
  },
  methods: {
    submitForm () {
      let self = this
      self.$refs['form'].validate((valid) => {
        if (valid) {
          this.$emit('submit-form', this.form)
        } else {
          self.$message({
            message: '请完整并正确填写表单',
            type: 'warning'
          })
        }
      })
    },
    resetForm () {
      this.$refs['form'].resetFields()
      this.$emit('reset-form', this.form)
    },
    radioGroupChange (subItem) {
      this.$emit('radio-group-change', subItem)
    },
    radioBtnGroupChange (subItem) {
      this.$emit('radio-btn-group-change', subItem)
    },
    checkboxGroupChange (subItem) {
      this.$emit('checkbox-group-change', subItem)
    },
    checkboxChange (subItem) {
      this.$emit('checkbox-change', subItem)
    },
    checkboxBtnGroupChange (subItem) {
      this.$emit('checkbox-btn-group-change', subItem)
    },
    buttonClick (subItem) {
      this.$emit('button-click', subItem)
    },
    changeSelect (subItem) {
      if (subItem.change) {
        this.$emit('change-select', subItem)
      }
    },
    selectClick (subItem) {
      if (subItem.click) {
        this.$emit('select-click', subItem)
      }
    }
  }
  // mounted () {
  //   // this.initForm = this.$objectOperationUtil.clone(this.form)
  // },
  // watch: {
  //   formInfo (val) { // 动态字段初始化
  //     let self = this
  //     if (val.length) {
  //       val.forEach(item => {
  //         item.components.forEach(subItem => {
  //           if (subItem.name && !self.form.hasOwnProperty(subItem.name)) {
  //             if (subItem.type === 'checkbox') {
  //               self.$set(self.form, subItem.name, false)
  //               // self.form[subItem.name] = false
  //             } else if (subItem.type === 'select') {
  //               self.$set(self.form, subItem.name, subItem.options.length ? subItem.options[0].value : '')
  //               // self.form[subItem.name] = subItem.options.length ? subItem.options[0].value : ''
  //             } else {
  //               self.$set(self.form, subItem.name, '')
  //               // self.form[subItem.name] = ''
  //             }
  //           }
  //         })
  //       })
  //     }
  //   }
  // }
}
</script>
<style scoped>
.othersClass >>> label {
  background-color: #e4e7ed;
}
.form-btn {
  display: flex;
  justify-content: flex-end;
  margin-bottom: 10px;
}
.comment::before {
  clear: both;
}
</style>
