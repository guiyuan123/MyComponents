<template>
  <ListLayout>
    <el-table :data="tableData" :row-style="showRow" class="table-new-primary" stripe border v-bind="$attrs" v-on="$listeners" >
      <slot name="selection" />
      <slot name="pre-column" />
      <el-table-column
        v-for="item in columns"
        :label="item.label"
        :key="item.key"
        :width="item.width"
        :min-width="item.minWidth"
        :align="item.align||'left'"
        :render-header="headerLabel"
        :header-align="item.headerAlign">
        <template slot-scope="scope">
          <slot :scope="scope" :name="item.key">
            <div class="tree-table-cell">
              <template v-if="item.expand">
                <!-- <span :style="{'padding-left':+scope.row._level*indent + 'px'} "/> -->
                <span v-show="showSperadIcon(scope.row)" :style="{'padding-left':+scope.row._level*indent + 'px'}" class="tree-ctrl"  @click="toggleExpanded(scope.$index)">
                  <i v-if="!scope.row._expand" class="el-icon-arrow-right" />
                  <i v-else class="el-icon-arrow-down" />
                </span>
                <span v-show="!showSperadIcon(scope.row)" :style="{'padding-left':+scope.row._level*indent + 'px'}" class="tree-ctrl no-pointer">
                  <!-- <i class="el-icon-minus" /> -->
                </span>
              </template>
              <!-- 可编辑 -->
              <template v-if="item.isEdit">
                <span v-if="item.type === 'text' || !item.type">
                  <span v-show="checkList.id!==scope.row.id">{{ scope.row[item.key] }}</span>
                  <el-input
                    v-show="checkList.id===scope.row.id"
                    v-model="checkList[item.key]"
                  ></el-input>
                </span>
                <span v-if="item.type === 'selected'">
                  <span v-show="checkList.id!==scope.row.id">{{ formatValue(item.selectList ,scope.row[item.key]) }}</span>
                  <el-select v-show="checkList.id===scope.row.id" v-model="checkList.treecode" placeholder="请选择" >
                    <el-option
                      v-for="sItem in item.selectList"
                      :key="sItem.id"
                      :label="sItem.name"
                      :value="sItem.id"
                    ></el-option>
                  </el-select>
                </span>
              </template>
              <template v-if="item.checkbox">
                <el-checkbox
                  v-if="scope.row[defaultChildren]&&scope.row[defaultChildren].length>0"
                  :style="{'padding-left':+scope.row._level*indent + 'px'} "
                  :indeterminate="scope.row._select"
                  v-model="scope.row._select"
                  @change="handleCheckAllChange(scope.row)" />
                <el-checkbox
                  v-else
                  :style="{'padding-left':+scope.row._level*indent + 'px'} "
                  v-model="scope.row._select"
                  @change="handleCheckAllChange(scope.row)" />
              </template>
              <template v-if="!item.isEdit">
              {{ scope.row[item.key] }}
              </template>
            </div>
          </slot>
        </template>
      </el-table-column>
    </el-table>
  </ListLayout>
</template>

<script>
import treeToArray, { addAttrs } from './eval.js'
import { ListLayout } from '@/components/UI'

export default {
  name: 'TreeTable',
  components: {
    ListLayout
  },
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
    },
    defaultExpandAll: {
      type: Boolean,
      default: false
    },
    defaultChildren: {
      type: String,
      default: 'children'
    },
    indent: {
      type: Number,
      default: 30
    }
  },
  data () {
    return {
      guard: 1,
      expandList: []
    }
  },
  computed: {
    children () {
      return this.defaultChildren
    },
    tableData () {
      const data = this.data
      if (this.data.length === 0) {
        return []
      }
      addAttrs(data, {
        expand: this.defaultExpandAll,
        children: this.defaultChildren
      })

      const retval = treeToArray(data, this.defaultChildren)
      return retval
    }
  },
  methods: {
    headerLabel (h, { column, $index }) {
      const status = this.columns[$index].isEdit
      if (status) {
        return (
          <span>
            <span>{column.label}</span>
            <i class="icon-edit iconfont qzd_icon" style="font-size:12px;color:#66b1ff;margin-left: 5px;"></i>
          </span>
        )
      } else {
        return (<span>{column.label}</span>)
      }
    },
    formatValue (list, key) {
      for (let item in list) {
        if (list[item].id === key) {
          return list[item].name
        }
      }
    },
    // addBrother (row, data) {
    //   if (row._parent) {
    //     row._parent.children.push(data)
    //   } else {
    //     this.data.push(data)
    //   }
    // },
    // addChild (row, data) {
    //   if (!row.children) {
    //     this.$set(row, 'children', [])
    //   }
    //   row.children.push(data)
    // },
    // delete (row) {
    //   const { _index, _parent } = row
    //   if (_parent) {
    //     _parent.children.splice(_index, 1)
    //   } else {
    //     this.data.splice(_index, 1)
    //   }
    // },
    // getData () {
    //   return this.tableData
    // },
    showRow: function ({ row }) {
      const parent = row._parent
      const show = parent ? parent._expand && parent._show : true
      row._show = show
      return show
        ? 'animation:treeTableShow 1s;-webkit-animation:treeTableShow 1s;'
        : 'display:none;'
    },
    showSperadIcon (record) {
      return record[this.children] && record[this.children].length > 0
    },
    toggleExpanded (trIndex) {
      const record = this.tableData[trIndex]
      const expand = !record._expand
      record._expand = expand
      if (expand) {
        this.expandList.push(record.id)
      } else {
        let ins = this.expandList.indexOf(record.id)
        this.expandList.splice(ins, 1)
      }
    },
    handleCheckAllChange (row) {
      this.selcetRecursion(row, row._select, this.defaultChildren)
      this.isIndeterminate = row._select
    },
    selcetRecursion (row, select, children = 'children') {
      if (select) {
        this.$set(row, '_expand', true)
        this.$set(row, '_show', true)
      }
      const subItem = row[children]
      if (subItem && subItem.length > 0) {
        subItem.map(child => {
          child._select = select
          this.selcetRecursion(child, select, children)
        })
      }
    },
    dataExpandInit () {
      this.tableData.forEach(item => {
        const ind = this.expandList.some(sItem => { return sItem === item.id })
        item.expand = ind
        item._expand = ind
      })
    }
  },
  watch: {
    // tableData: {
    //   handler (v, p) {
    //   },
    //   deep: true,
    //   immediate: true
    // }
    data: {
      handler () {
        this.dataExpandInit()
      }
    }
  }
}
</script>

<style>
@keyframes treeTableShow {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}
@-webkit-keyframes treeTableShow {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}

.tree-ctrl {
  position: relative;
  cursor: pointer;
  color: #2196f3;
  font-size: 16px;
}
.tree-ctrl.no-pointer{
  cursor: initial;
}
.tree-table-cell{
  display: flex;
}
</style>
