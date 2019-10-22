<template>
  <div class="select-content">
    <!-- {{linkageSelectData}} -->
    <!-- {{optionsList}} -->
    <span v-for="(item, key) in labelKeyList" :key="key">
      <span class="label-text">{{item.label}}</span>
      <el-select
        v-model="item.value"
        @change="changeOptions(key)"
        class="select-margin"
        size="small"
      >
        <el-option
          v-for="(subItem, subKey) in optionsList[key]"
          :key="subKey"
          :value="subItem[value]"
          :label="subItem[label]"
        ></el-option>
      </el-select>
    </span>
  </div>
</template>

<script>
/**
 * @labelKeyList 下拉框的信息，包括显示，值和字段key，例如  this.labelKeyList = [{  label: 'DB类型',  key: 'DbType',  value: ''},{  label: '版本',  key: 'version',  value: ''},{  label: '测试字段',  key: 'test',  value: ''}]
 * @linkageSelectData 所有下拉框的选项列表 例如 this.linkageSelectData = {label: 'c12',value: 'c12',id: 12,pId: 1,level: 1,children: [  {    label: 'c121',    value: 'c121',    id: 121,    pId: 11,    level: 2,    children: []  },  {    label: 'c122',    value: 'c122',    id: 22,    pId: 11,    level: 2,    children: []  }]}
 */
export default {
  props: {
    // 下拉框的label和key
    labelKeyList: {
      type: Array,
      default () {
        return []
      }
    },
    // 所有数据(后台请求回来的，包括类似label, value, children, level, id, pId, 后面3个字段非必要)
    linkageSelectData: {
      type: Array,
      default () {
        return []
      }
    },
    // 指定数据的哪个字段作为下拉框选项的显示
    label: {
      type: String,
      default () {
        return 'label'
      }
    },
    // 指定数据的哪个字段作为下拉框选项的值
    value: {
      type: String,
      default () {
        return 'value'
      }
    }
  },
  data () {
    return {
      // 所有下拉框的选项
      optionsList: []
    }
  },
  beforeMount () {
    this.initOptionsList()
  },
  methods: {
    // 初始化各个下拉框的选项列表
    initOptionsList () {
      if (this.linkageSelectData.length) {
        let self = this
        // this.optionsList.length = this.labelKeyList.length // 注释的这2步检测不到optionsList的变化，原因暂时不明
        // this.optionsList.fill([])
        this.optionsList = Array(self.labelKeyList.length).fill([])
        this.optionsList[0] = this.linkageSelectData
        for (let i = 1; i < this.optionsList.length; i++) {
          // 如果前面的下拉框为空，则无法决定当前下拉框的选项列表
          if (self.labelKeyList[i - 1].value === '') {
            return
          }
          let index = this.optionsList[i - 1].findIndex(obj => obj[self.value] === self.labelKeyList[i - 1].value)
          self.optionsList[i] = self.optionsList[i - 1][index].children
        }
      }
    },
    // 切换选项时改变后面的下拉框选项
    changeOptions (key) {
      let self = this
      if (key < this.optionsList.length - 1) {
        let index = self.optionsList[key].findIndex(obj => obj[self.value] === self.labelKeyList[key].value)
        self.optionsList[key + 1] = self.optionsList[key][index].children
        // 清空后面的选项
        self.optionsList.fill([], key + 2)
        for (let k = key + 1; k < this.labelKeyList.length; k++) {
          self.labelKeyList[k].value = ''
        }
      }
      this.$emit('on-change', self.labelKeyList[key])
    }
  },
  watch: {
    linkageSelectData (val) {
      this.initOptionsList()
    }
  }
}
</script>

<style lang="scss" scoped>
.select-content {
  display: inline-block;
}
.select-margin {
  margin-right: 15px;
}
.label-text {
  line-height: 34px;
  text-align: right;
  padding-right: 5px;
}
</style>
