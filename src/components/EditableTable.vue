<template>
  <div>
    <el-table size="medium" :data="data" layout="total, sizes">
      <el-table-column
        v-for="(item, key) in columns"
        :key="key"
        :label="item.label"
        :prop="item.prop"
      >
        <template slot-scope="scope">
          <span v-if="item.isEditable || (isEditable && !('isEditable' in item))">
            <el-input size="small" placeholder="请输入内容" v-model="scope.row[item.prop]" @blur="getEditContent(scope.row[item.prop])" type="textarea"></el-input>
          </span>
          <span v-else>{{scope.row[item.prop]}}</span>
        </template>
      </el-table-column>
    </el-table>
  </div>
</template>

<script>
export default {
  props: {
    data: {
      default: function () {
        return []
      },
      type: Array
    },
    columns: {
      default: function () {
        return []
      },
      type: Array
    },
    isEditable: {
      default: false,
      type: Boolean
    }
  },
  methods: {
    getEditContent (val) {
      this.$emit('get-edit-content', val)
    }
  }
}
</script>

<style>
.el-table__header-wrapper > table > thead > tr > th {
  padding: 6px 0;
  height: 45px;
  background-color: #f0f2f5!important;
}
</style>
