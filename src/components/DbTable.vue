<template>
  <div :class="{'file-choice': show }">
    <div class="flex-item" v-if="show">
      <span class="flex-item-top">
        <span class="step-title" v-if="show" style="width: 40%">{{leftTitle}}</span>
        <span class="search-part">
          <el-input
            v-model="keyword"
            size="small"
            style="width: 75%; margin-right: 15px;"
            @keyup.enter.native="search"
          ></el-input>
          <el-button type="primary" size="small" @click="search">搜索</el-button>
        </span>
      </span>
      <!-- <sbt-tree-grid
                :columns="allRowCol"
                :tree-structure="true"
                :data-source="allList"
                :default-expand-all="defaultExpandAll"
                :multi-selection="true"
                @multi-select="handleSelectedAllRows"
                max-height="490"
      ></sbt-tree-grid>-->
      <sbt-element-table
        :data="allList"
        :columns="allCol"
        expand-all="false"
        :multi-selection="multiSelection"
        @row-click="handleSelectedAllRow"
        @multi-select="handleSelectedAllRows"
        max-height="490"
        layout="total, sizes, prev, pager, next"
        ref="selectedAllRows"
        :tablePageBottomShow="false"
        :page-size="10000"
      ></sbt-element-table>
    </div>
    <div class="file-btn" v-if="show">
      <el-button
        size="small"
        @click="addRow"
        :disabled="!selectedAllRows.length || addDisabled"
        type="primary"
      >添加</el-button>
      <el-button
        size="small"
        @click="removeRow"
        :disabled="!selectedRows.length || delDisabled"
        type="primary"
      >删除</el-button>
    </div>
    <el-form-item
      :class="{'flex-item' : show}"
      :label="showLabel ? rightTitle : ''"
      :label-width=" showLabel ? '40%' : '0%'"
    >
      <span class="step-title" v-if="show">{{rightTitle}}</span>
      <sbt-element-table
        :data="selectedList"
        :columns="selectedCol"
        expand-all="false"
        :multi-selection="multiSelection"
        @multi-select="handleSelectedRows"
        @row-click="handleSelectedRow"
        max-height="490"
        layout="total, sizes, prev, pager, next"
        :style="show ? 'margin-top:10px;' : 'margin-top:10px;width: 50%;'"
        :tablePageBottomShow="false"
        :page-size="10000"
        ref="selectedRows"
      ></sbt-element-table>
    </el-form-item>
  </div>
</template>

<script>
import SbtElementTable from '@/components/sbtTables/sbtElementTable/SbtElementTable'
export default {
  components: {
    SbtElementTable
  },
  props: {
    param: { // 指定对比哪个值
      default: '',
      type: String
    },
    // 是否显示左边表格
    show: {
      default: false,
      type: Boolean
    },
    // 是否多选
    multiSelection: {
      default: true,
      type: Boolean
    },
    // 左边列表标题
    leftTitle: {
      default: '',
      type: String
    },
    // 右边列表标题
    rightTitle: {
      default: '',
      type: String
    },
    // 控制添加按钮的禁启用
    addDisabled: {
      default: false,
      type: Boolean
    },
    // 控制删除按钮的禁启用
    delDisabled: {
      default: false,
      type: Boolean
    },
    // 是否以显示表单的Label
    showLabel: {
      default: false,
      type: Boolean
    },
    allList: {
      default: function () {
        return []
      },
      type: Array
    },
    allCol: {
      default: function () {
        return []
      },
      type: Array
    },
    selectedList: {
      default: function () {
        return []
      },
      type: Array
    },
    selectedCol: {
      default: function () {
        return []
        // console.log(this)
        // return this.allCol
      },
      type: Array
    }
  },
  data () {
    return {
      keyword: '',
      selectedAllRows: [],
      selectedRows: []
    }
  },
  methods: {
    search () {
      this.$emit('search', this.keyword)
    },
    handleSelectedAllRows (rows) {
      this.selectedAllRows = rows
      this.$emit('multi-select-all', rows)
    },
    // 从已选的文件目录选择
    handleSelectedRows (rows) {
      this.selectedRows = rows
      this.$emit('multi-select', rows)
    },
    handleSelectedAllRow (row) {
      if (this.multiSelection) {
        this.$refs['selectedAllRows'].$children[0].toggleRowSelection(row)
      } else {
        this.selectedAllRows = [row]
      }
    },
    handleSelectedRow (row) {
      if (this.multiSelection) {
        this.$refs['selectedRows'].$children[0].toggleRowSelection(row)
      } else {
        this.selectedRows = [row]
      }
      // let self = this
      // let index = this.selectedRows.findIndex((item) => {
      //   return item[self.param] === row[self.param]
      // })
      // if (index > -1) {
      //   this.selectedRows.splice(index, 1)
      // } else {
      //   this.selectedRows.push(row)
      // }
      // this.$emit('multi-select', this.selectedRows)
    },
    addRow () {
      let exitMsg = ''
      let repeatMsg = ''
      for (let i = 0; i < this.selectedAllRows.length; i++) {
        let j = 0
        for (; j < this.selectedList.length; j++) {
          // 选中的路径已经包含在恢复列表中，则不添加
          // if ((this.selectedAllRows[i][this.param].indexOf(this.selectedList[j][this.param]) > -1 && this.selectedList[j][this.param].split(/\//).length !== this.selectedAllRows[i][this.param].split(/\//).length) || this.selectedAllRows[i][this.param] === this.selectedList[j][this.param]) {
          if ((this.selectedAllRows[i][this.param].indexOf(this.selectedList[j][this.param]) === 0 && this.selectedList[j][this.param].lastIndexOf('/') === this.selectedList[j][this.param].length - 1) || this.selectedAllRows[i][this.param] === this.selectedList[j][this.param]) {
            exitMsg = `${exitMsg}${this.selectedAllRows[i][this.param]}已经包含在${this.selectedList[j][this.param]}中<br/>`
            break
          }
          // 恢复列表中的路径包含在选中的路径中，则移除恢复列表中的行，添加选中的路径到恢复列表，如果移除的行是属于搜索结果的项，还需要添加到搜索结果，否则直接移除掉
          // if (this.selectedList[j][this.param].indexOf(this.selectedAllRows[i][this.param]) > -1 && this.selectedList[j][this.param].split(/\//).length !== this.selectedAllRows[i][this.param].split(/\//).length) {
          if (this.selectedList[j][this.param].indexOf(this.selectedAllRows[i][this.param]) === 0 && this.selectedAllRows[i][this.param].lastIndexOf('/') === this.selectedAllRows[i][this.param].length - 1) {
            repeatMsg = '已移除恢复列表中重复的行'
            if (this.selectedList[j].isResult) {
              // this.selectedList[j].isResult = false
              this.allList.push(this.selectedList[j])
            }
            this.selectedList.splice(j, 1)
            j-- // 从移除的地方开始计数
            continue
          }
        }
        if (j >= this.selectedList.length) {
          this.selectedList.push(this.selectedAllRows[i])
          this.allList.splice(this.allList.indexOf(this.selectedAllRows[i]), 1)
        }
      }
      this.selectedAllRows = []
      // 数组去重
      // this.selectedList = Array.from(new Set(this.selectedList))
      if (exitMsg || repeatMsg) {
        this.$message({
          type: 'warning',
          dangerouslyUseHTMLString: true,
          message: `${exitMsg}${exitMsg && repeatMsg ? `且${repeatMsg}` : repeatMsg}`
        })
      }
      this.$emit('add-row', { all: this.allList, selected: this.selectedList })
    },
    removeRow () {
      this.selectedRows.forEach(item => {
        console.log(item)
        // 如果是属于搜索结果的，则添加到未选择的列表中，否则直接移除
        if (item.isResult) {
          // item.isResult = false
          this.allList.push(item)
        }
        this.selectedList.splice(this.selectedList.indexOf(item), 1)
      })
      this.selectedRows = []
      this.$emit('remove-row', { all: this.allList, selected: this.selectedList })
    }
  }
}
</script>

<style scoped>
.file-choice {
  display: flex;
  justify-content: center;
}
.flex-item {
  margin: 0 20px;
  width: 40%;
}
.flex-item-top {
  display: flex;
  align-items: center;
  justify-content: space-between;
}
.step-title {
  font-size: 14px;
  font-weight: 600;
  line-height: 30px;
  display: block;
  margin: 15px 0;
}
.file-btn {
  margin-top: 80px;
  /* flex-shrink: 1; */
  width: 10%;
  /* align-self: center; */
  /* display: flex;
  flex-direction: column; */
  align-items: center;
}
.file-btn > button {
  margin: 20px auto !important;
  display: block;
}
.search-part {
  text-align: right;
  display: inline-block;
  width: 60%;
}
</style>
