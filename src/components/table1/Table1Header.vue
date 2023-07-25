<template>
    <thead>
        <tr v-for="item in columns" :key="item.prop">
          <th  :key="th.prop" v-for="th in item" :colspan="th.colSpan" :rowspan="th.rowSpan">{{th.label}}</th>
       </tr>
    </thead>
</template>
<script>
import Draggable from 'vuedraggable';

export default {
  name: 'Table1Header',
  components:{
      Draggable
  },
  data() {
    return {
      columns: [
        { prop: 'index', label: '', label: '日期' },
        {
          label: '其他信息',
           prop:'other',
          children: [
            {
              prop: 'age', label: '年龄',
            },
            {
              label: '地址',
              prop:'address',
              children: [
                {
                  label: '省份',
                  prop: 'street'
                },
                {
                  label: '区域',
                   prop:'address1',
                  children: [
                    {
                      label: '区域1',
                      prop: 'building',
                    },
                    {
                      label: '区域2',
                      prop: 'number',
                    },
                  ],
                },
              ]
            }
          ]
        },
      ],
    }
  },
  created() {
    this.columns = this.convertToRows(this.columns)
    console.log('columns: ', this.columns);
  },
  methods: {
    //计算单元格的colSpan
    /**
     * 计算单元格的colsapn，用于合并列
     * 计算单元格所属的层级
     * @param {*} originColumns，是指第一层级的元素
     */
    convertToRows(originColumns) {
      let maxLevel = 1;
      const traverse = (column, parent) => {
        if (parent) {
          //计算当前元素属于第几个层级
          column.level = parent.level + 1;
          if (maxLevel < column.level) {
            //计算最大层级
            maxLevel = column.level;
          }
        }
        if (column.children) {
          let colSpan = 0;
          column.children.forEach((subColumn) => {
            traverse(subColumn, column);
            colSpan += subColumn.colSpan;
          });
          column.colSpan = colSpan;
        } else {
          column.colSpan = 1;
        }
      };
      //设置第一层级的leave = 1； 和第一层级的colspan
      originColumns.forEach((column) => {
        column.level = 1;
        traverse(column);
      });
      //计算有几个tr；得到rows的数组[];
      const rows = [];
      for (let i = 0; i < maxLevel; i++) {
        rows.push([]);
      }
      //获得所有的th项， 展开树形数据
      const allColumns = this.getAllColumns(originColumns);
      //计算单元格的rowspan;
      //得到循环的数组
      allColumns.forEach((column) => {
        if (!column.children) {
          column.rowSpan = maxLevel - column.level + 1;
        } else {
          column.rowSpan = 1;
        }
        rows[column.level - 1].push(column);
      });

      return rows;
    },
    /**
     * 展开树形数据结构
     * @param {*} columns :为树形结构的对象
     */
    getAllColumns(columns) {
      const result = [];
      columns.forEach((column) => {
        if (column.children) {
          result.push(column);
          result.push.apply(result, this.getAllColumns(column.children));
        } else {
          result.push(column);
        }
      });
      return result;
    },
    renderHeader() {

    }
  }
}
</script>
<style scoped>

</style>
