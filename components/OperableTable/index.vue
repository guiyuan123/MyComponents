<template>
  <div>
    <el-table :data="data" v-bind="$attrs" v-on="$listeners" stripe border class="table-new-primary">
      <slot name="selection"/>
      <slot name="pre-column"/>
      <slot name="index"/>
      <el-table-column
        v-for="item in columns"
        :label="item.label"
        :key="item.key"
        :width="item.width"
        :min-width="item.minWidth"
        :align="item.align||'left'"
        :header-align="item.headerAlign"
        :show-overflow-tooltip="true"
      >
        <template slot="header" v-if="item.isEdit && !item.sort">
          <span>
            <span>{{item.label}}</span>
            <i class="icon-edit iconfont qzd_icon" style="font-size:12px;color:#66b1ff;margin-left: 5px;"></i>
          </span>
        </template>
        <!-- 排序回调 -->
        <template slot="header" v-if="!item.isEdit && item.sort">
          <span class="is-sort-col" @click="callBackSort(item.key)">
            <span>{{item.label}}</span>
            <i class="el-icon-arrow-down is-sort-icon"></i>
          </span>
        </template>
        <template slot="header" v-if="!item.isEdit && !item.sort">
          <span >
            <span>{{item.label}}</span>
          </span>
        </template>
        <template slot-scope="scope">
          <slot :scope="scope" :name="item.key">
            <!-- 可编辑 -->
            <div v-if="item.isEdit">
              <span v-if="item.type === 'text' || !item.type">
                <span v-show="checkList.id!==scope.row.id">{{ scope.row[item.key] }}</span>
                <el-input
                  :maxlength="item.maxlength"
                  v-show="checkList.id===scope.row.id"
                  v-model.trim="checkList[item.key]"
                ></el-input>
              </span>
              <span v-if="item.type === 'selected'">
                <span v-show="checkList.id!==scope.row.id" v-html="item.definedFormat ? item.definedFormat(item.selectList ,scope.row[item.key]) : formatValue(item.selectList ,scope.row[item.key])"></span>
                <el-select v-show="checkList.id===scope.row.id" v-model="checkList[item.key]" placeholder="请选择" >
                  <el-option
                    v-for="sItem in item.selectList"
                    :key="sItem.id"
                    :label="sItem.name"
                    :value="sItem.id"
                  ></el-option>
                </el-select>
              </span>
              <span v-if="item.type === 'date'">
                <span v-show="checkList.id!==scope.row.id">{{ scope.row[item.key] | parseDate }}</span>
                <el-date-picker
                  v-show="checkList.id===scope.row.id"
                  type="date"
                  v-model="checkList[item.key]"
                  placeholder="选择日期">
                </el-date-picker>
              </span>
            </div>
            <!-- 不可编辑 -->
            <div v-else>
              <span v-if="item.type === 'date'">
                {{ scope.row[item.key] | parseTime }}
              </span>
              <span v-else>
                {{ scope.row[item.key] }}
              </span>
            </div>
          </slot>
        </template>
      </el-table-column>
    </el-table>
  </div>
</template>

<script>
export default {
  props: {
    data: {
      type: Array,
      required: true,
      default: () => []
    },
    columns: {
      type: Array,
      default: () => []
    },
    checkList: {
      type: Object,
      default: () => {}
    }
  },
  methods: {
    formatValue (list, key) {
      for (let item in list) {
        if (list[item].id === key) {
          return list[item].name
        }
      }
    },
    callBackSort (key) {
      this.$emit('onSort', key)
    },
    headerLabel (h, { column, $index }) {
      const status = this.columns[$index].isEdit
      const sort = this.columns[$index].type
      if (status && sort !== 'sort') {
        return (
          <span>
            <span>{column.label}</span>
            <i class="icon-edit iconfont qzd_icon" style="font-size:12px;color:#66b1ff;margin-left: 5px;"></i>
          </span>
        )
      } else if (!status && sort !== 'sort') {
        return (
          <span>
            <span>{column.label}</span>
          </span>
        )
      } else if (!status && sort === 'sort') {
        return (
          <span>
            <span>{column.label}</span>
            <i class="el-icon-arrow-down"></i>
          </span>
        )
      }
    }
  }
}
</script>

<style lang="scss" scoped>
  .is-sort-col{
    cursor: pointer;
  }
  .is-sort-icon{
    font-size: 14px;
    color: #385C98;
    vertical-align: middle;
  }
</style>
