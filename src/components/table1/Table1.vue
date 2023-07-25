<template>
  <table
    class="infinite-split-table"
    cellpadding="0"
    cellspacing="0"
    border="1"
    style="width:100%"
    >
     <!-- <colgroup>
        <col v-for="colItem in [1,2,3,4,5]"></col>
     </colgroup> -->
      <Table1Header></Table1Header>
  </table>
</template>
<script>
import Table1Header from './Table1Header.vue'
export default {
  name: 'TablePage',
  components:{
    Table1Header
  },
  data() {
    return {
      columns: [
        { prop: 'index', label: '', width: 3, label: '日期' },
        {
          label: '其他信息',
          layout:'column',
          children: [
            {
              prop: 'age', label: '年龄',
            },
            {
              label: '地址',
              children: [
                {
                  label: '省份',
                  prop: 'street'
                },
                {
                  label: '区域',
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
