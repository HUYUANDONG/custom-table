<template>
  <div>
    <Table2Header ref="tableHeaderRef" v-on="$listeners" :headerList="tableConfig.headerList" @getMaxWidth="getMaxWidth" @clickCol="onClickCol" @mouseUpHandle="mouseUpHandle"></Table2Header>
    <Table2Body v-on="$listeners" :bodyList="tableConfig.bodyList" @getMaxWidth="getMaxWidth" @mouseUpHandle="mouseUpHandle"></Table2Body>
  </div>
</template>
<script>
import Table2Header from './Table2Header.vue'
import Table2Body from './Table2Body.vue'
export default {
  components: {
    Table2Header,
    Table2Body
  },
  provide() {
    return {
      colInfo: { col: this.tableConfig.col, cellHeight: this.tableConfig.cellHeight, cellWidth: this.tableConfig.cellWidth }

    }
  },
  data() {
    return {
      isShift: false,
      canMove: false,
      rowIndex: 0,
      clickParams: {},
      clientWidth: 0,
      maxWidth: 0,
      minWidth: 0,
      selectColArray: [],
      tableConfig: {
        cellHeight: 20,
        cellWidth: 20,
        col: 3,
        headerList: [
          {
            level: 0,
            height: 0,
            key: 1111,
            line: 1,
            layout: 'row',
            rowItem: [
              {
                prop: '1index', label: '道德与法治', index: 0, level: 1, width: 0, col: 1, isSelected: false, isMerged: false,
              },
              { prop: 'address', label: '学习水平', index: 1, level: 1, width: 0, col: 1, isSelected: false, isMerged: false, },
              { prop: 'address1', label: '过程表现', index: 2, level: 1, width: 0, col: 1, isSelected: false, isMerged: false, },
            ]
          }
        ],
        bodyList: [
          {
            height: 0,
            key: 1,
            line: 6,
            level: 0,
            layout: 'row',
            rowItem: [
              {
                col: 1, prop: 'index', label: '333', index: 0, level: 1, width: 0, isSelected: false
              },
              {
                col: 1,
                layout: 'column',
                index: 1,
                level: 1,
                width: 0,
                isSelected: false,
                rowItem: [
                  {
                    prop: '认知水平1', label: '', level: 2, index: 1
                  },
                  {
                    prop: '认知水平1', label: '', level: 2, index: 1
                  },
                  {
                    prop: '认知水平1', label: '', level: 2, index: 1
                  },
                  {
                    prop: '认知水平1', label: '', level: 2, index: 1
                  },
                  {
                    prop: '认知水平1', label: '', level: 2, index: 1
                  },
                  {

                    prop: '实践能力1', label: '', level: 2, index: 1

                  }
                ]
              },
              { col: 1, prop: 'index1', label: '', index: 2, level: 1, width: 0, isSelected: false },
            ]
          },
        ],
      }
    }
  },
  mounted() {
    this.keyDown()
  },

  methods: {

    //监听键盘是否按下shift
    keyDown() {
      // 键盘按下事件
      document.onkeydown = (e) => {
        // 取消默认事件
        e.preventDefault();
        //事件对象兼容
        let e1 = e || event || window.event || arguments.callee.caller.arguments[0]
        //键盘按键判断:左箭头-37;上箭头-38；右箭头-39;下箭头-40  回车：13   ctrl：17   shift：16
        switch (e1.keyCode) {
          case 16:
            this.isShift = true;  // 如果shift按下就让他按下的标识符变为true
            console.log('this.isShift: ', '按下');
            break;

        }
      }
      // 键盘抬起事件
      document.onkeyup = (e) => {
        // 取消默认事件
        e.preventDefault();
        //事件对象兼容
        let e1 = e || event || window.event || arguments.callee.caller.arguments[0]
        switch (e.keyCode) {
          case 16:
            this.isShift = false; // 如果shift抬起下就让他按下的标识符变为false
            console.log('this.isShift: ', '抬起');
            break;

        }
      }
    },
    //拉伸行、列
    mouseMoveHandle() {
      if (this.canMove) {
        if (this.clickParams.direction == 'x') {
          let colIndex = this.clickParams.colIndex
          let moveX = event.clientX - this.clickParams.startX
          //拖动有最小宽度和最大宽度
          let realMoveX = moveX
          if (this.clickParams.initWidth + moveX < this.minWidth) {
            realMoveX = this.minWidth - this.clickParams.initWidth
          }

          if (this.clickParams.initWidth + moveX >= this.maxWidth) {
            realMoveX = this.maxWidth - this.clickParams.initWidth
          }
          for (const item of this.tableConfig.headerList) {
            item.rowItem[colIndex].width = this.clickParams.initWidth + realMoveX
            item.rowItem[colIndex + 1].width = this.clickParams.nextWidth - realMoveX
          }

          for (const item of this.tableConfig.bodyList) {
            item.rowItem[colIndex].width = this.clickParams.initWidth + realMoveX
            item.rowItem[colIndex + 1].width = this.clickParams.nextWidth - realMoveX
            if (item.rowItem[colIndex].rowItem) {
              for (const colItem of item.rowItem[colIndex].rowItem) {
                if (colItem.initWidth) {
                  colItem.width = colItem.initWidth + realMoveX / item.rowItem[colIndex].rowItem.length
                }
              }
              if (item.rowItem[colIndex + 1].rowItem) {
                for (const colItem of item.rowItem[colIndex + 1].rowItem) {
                  if (colItem.initWidth) {
                    colItem.width = colItem.initWidth - realMoveX / item.rowItem[colIndex + 1].rowItem.length
                  }
                }
              }

            }
          }
        }
        if (this.clickParams.direction == 'y') {
          let rowIndex = this.clickParams.rowIndex
          let moveY = event.clientY - this.clickParams.startY
          //表头拖动 最小高度20
          if (this.clickParams.isHeader) {
            if (this.clickParams.initHeight + moveY >= this.tableConfig.cellHeight) {
              this.tableConfig.headerList[rowIndex].height = this.clickParams.initHeight + moveY
              this.$emit('onStretchDrag', moveY)
            } else {
              this.tableConfig.bodyList[rowIndex].height = this.tableConfig.cellHeight
              this.$emit('onStretchDrag', this.tableConfig.cellHeight - this.clickParams.initHeight)
            }
          } else {
            //表内容拖动
            if (this.clickParams.initHeight + moveY >= this.tableConfig.cellHeight) {
              this.tableConfig.bodyList[rowIndex].height = this.clickParams.initHeight + moveY
              this.$emit('onStretchDrag', moveY)
            } else {
              this.tableConfig.bodyList[rowIndex].height = this.tableConfig.cellHeight
              this.$emit('onStretchDrag', this.tableConfig.cellHeight - this.clickParams.initHeight)
            }
          }
        }
      }
    },
    //鼠标抬起
    mouseUpHandle(e) {
      this.canMove = false
      window.onmousemove = null;
    },
    getMaxWidth(params) {
      this.canMove = true
      window.onmousemove = this.mouseMoveHandle;
      //因为refs是通过push的方式加入的，所以会存在顺序问题，需要重新排序
      let moduleArrray = this.$refs.tableHeaderRef.$children
      moduleArrray.sort((a, b) => {
        return a.rowData.index - b.rowData.index
      })
      this.clickParams = params
      if (this.clickParams.direction == 'x') {
        this.clickParams.nextWidth = moduleArrray[params.colIndex + 1].$el.clientWidth
        //计算最小宽度
        this.minWidth = this.tableConfig.cellWidth
        if (params.colIndex < this.tableConfig.headerList[params.rowIndex].rowItem.length - 1) {
          //计算最大宽度
          this.maxWidth = params.initWidth + moduleArrray[params.colIndex + 1].$el.clientWidth - this.tableConfig.cellWidth
        }
        //记录列的初始宽度
        for (const row of this.tableConfig.bodyList) {
          if (row.rowItem) {
            row.rowItem[params.colIndex].initWidth = row.rowItem[params.colIndex].width
            row.rowItem[params.colIndex + 1].initWidth = row.rowItem[params.colIndex + 1].width
            if (row.rowItem[params.colIndex].rowItem) {
              for (const col of row.rowItem[params.colIndex].rowItem) {
                col.initWidth = col.width
              }
            }
            if (row.rowItem[params.colIndex + 1].rowItem) {
              for (const col of row.rowItem[params.colIndex + 1].rowItem) {
                col.initWidth = col.width
              }
            }
          }
        }
      }
      if (this.clickParams.direction == 'y') {
      }
    },
    //选中列
    onClickCol(params) {
      console.log('index: ', params);
      let rowIndex = params.rowIndex
      let colIndex = params.colIndex
      //如果一个都没有选中,选中第一个是单机
      if (!this.selectColArray.length) {
        this.tableConfig.headerList[rowIndex].rowItem[colIndex].isSelected = true
        this.selectColArray.push(colIndex)
      } else {
        //确保是连续的
        if (!this.selectColArray.includes(colIndex)) {
          //从未选中到选中，选中第二个要加shift，可以多选
          if (this.isShift) {
            let firstSelectIndex = this.selectColArray[0]
            let lastSelectIndex = this.selectColArray[this.selectColArray.length - 1]
            //选中的在前面
            if (colIndex < firstSelectIndex) {
              this.selectColArray = Array.from({ length: firstSelectIndex - colIndex + 1 }, (v, k) => k + colIndex);
            } else if (colIndex > lastSelectIndex) {
              //选中的在后面
              this.selectColArray = Array.from({ length: colIndex - lastSelectIndex + 1 }, (v, k) => k + this.selectColArray[0]);
            }
            for (const item of this.selectColArray) {
              this.tableConfig.headerList[rowIndex].rowItem[item].isSelected = true
            }
          } else {
            //把所有选中的改成未选中
            for (const item of this.selectColArray) {
              this.tableConfig.headerList[rowIndex].rowItem[item].isSelected = false
            }
            this.tableConfig.headerList[rowIndex].rowItem[colIndex].isSelected = true
            this.selectColArray = [colIndex]
          }

        } else {
          //从选中到未选中
          for (const item of this.selectColArray) {
            this.tableConfig.headerList[rowIndex].rowItem[item].isSelected = false
          }
          if (this.isShift) {
            let firstSelectIndex = this.selectColArray[0]
            this.selectColArray = Array.from({ length: colIndex - firstSelectIndex + 1 }, (v, k) => k + firstSelectIndex);
            for (const item of this.selectColArray) {
              this.tableConfig.headerList[rowIndex].rowItem[item].isSelected = true
            }
          } else {
            this.tableConfig.headerList[rowIndex].rowItem[colIndex].isSelected = true
            this.selectColArray = [colIndex]

          }

        }
      }
      this.selectColArray.sort()
      console.log('this.selectColArray: ', this.selectColArray);
    },
    //拆分列
    onSplitCol() {
        let colIndex = this.selectColArray[0]
        //判断是否可以拆分，单选+选中的该列之前合并过
      if (this.selectColArray.length == 1 && this.tableConfig.headerList[0].rowItem[colIndex].col>1) {
        let newTableConfig = JSON.parse(JSON.stringify(this.tableConfig))
        let indexArray = [] //记录需要重新选中的列
        //将合并的列中的子项拆出来
        for (const bodyItemIndex in newTableConfig.headerList) {
          let headerInsertCol = newTableConfig.headerList[bodyItemIndex].rowItem[colIndex].rowItem
          let bodyInsertCol = newTableConfig.bodyList[bodyItemIndex].rowItem[colIndex].rowItem
          let firstIndex = colIndex
          for (const colItemIndex in headerInsertCol) {
            headerInsertCol[colItemIndex].isMerged = false
            headerInsertCol[colItemIndex].width = bodyInsertCol[colItemIndex].width
            if (!indexArray.includes(firstIndex)) {
              indexArray.push(firstIndex)
            }
            firstIndex++
          }
          newTableConfig.headerList[bodyItemIndex].rowItem.splice(colIndex, 1, ...headerInsertCol)
          newTableConfig.bodyList[bodyItemIndex].rowItem.splice(colIndex, 1, ...bodyInsertCol)
        }
        this.selectColArray = indexArray
        newTableConfig.col = newTableConfig.headerList[0].rowItem.length
        this.computedRowCol(newTableConfig)
        this.tableConfig = newTableConfig
        console.log('this.splittableConfig: ', newTableConfig);

      }
    },
    //合并列
    onMergeCol() {
      if (this.selectColArray.length > 1) {
        let mergeWidth = 0
        for (const item of this.selectColArray) {
          for (const bodyItem of this.tableConfig.bodyList) {
            bodyItem.rowItem[item].width = this.$refs.tableHeaderRef.$children[item].$el.clientWidth
          }
          mergeWidth += this.$refs.tableHeaderRef.$children[item].$el.clientWidth
        }
        let newTableConfig = JSON.parse(JSON.stringify(this.tableConfig))
        let newColIndex = this.selectColArray[0]
        for (const item of newTableConfig.headerList) {
          let newCol = { index: newColIndex, col: 1, width: mergeWidth, isSelected: true, label: '', prop: '884747' + item.key } //合并完以后的列
          newCol.layout = 'row'
          newCol.index = newColIndex
          newCol.rowItem = []
          let mergeColIndex = 0
          for (const itemCol of this.selectColArray) {
            item.rowItem[itemCol].isSelected = false
            if (mergeColIndex) {
              item.rowItem[itemCol].isMerged = true
            }
            item.rowItem[itemCol].width = 0
            newCol.rowItem.push(item.rowItem[itemCol])
            mergeColIndex++
          }
          item.rowItem.splice(newColIndex, this.selectColArray.length, newCol)
        }
        for (const item of newTableConfig.bodyList) {
          let newCol = { index: newColIndex, col: 1, width: mergeWidth, isSelected: false, label: '', prop: '884747' + item.key } //合并完以后的列
          newCol.layout = 'row'
          newCol.index = newColIndex
          newCol.rowItem = []
          for (const itemCol of this.selectColArray) {
            item.rowItem[itemCol].isSelected = false
            newCol.rowItem.push(item.rowItem[itemCol])
          }
          item.rowItem.splice(newColIndex, this.selectColArray.length, newCol)
        }
        this.computedRowCol(newTableConfig)
        //合并完以后只选中第一条
        this.selectColArray = [this.selectColArray[0]]
        newTableConfig.col = newTableConfig.headerList[0].rowItem.length
        this.tableConfig = newTableConfig
        console.log('this.tableConfig: ', newTableConfig);
      }
    },
    //
    computedRowCol(data) {
      const countRow = (column, rowData, colData, lineCount = 1, colCount = 1, index) => {
        if (column.rowItem) {
          if (column.layout == 'column') {
            lineCount = lineCount + column.rowItem.length - 1
            if (lineCount > rowData.line) {
              rowData.line = lineCount
            }
          }
          if (column.layout == 'row') {
            colCount = colCount + column.rowItem.length - 1
            if (colCount > colData.col) {
              colData.col = colCount
            }
          }
          for (const item of column.rowItem) {
            item.level = column.level + 1
            item.index = colData.index
            countRow(item, rowData, colData, lineCount, colCount)
          }
        }
      }
      for (const item of data.headerList) {
        item.line = 1
        item.level = 0
        let index = 0
        for (const colItem of item.rowItem) {
          colItem.col = 1
          colItem.level = 1
          colItem.index = index
          countRow(colItem, item, colItem, 1, 1)
          index++
        }
      }
      for (const item of data.bodyList) {
        item.line = 1
        item.level = 0
        let index = 0
        for (const colItem of item.rowItem) {
          colItem.col = 1
          colItem.level = 1
          colItem.index = index
          countRow(colItem, item, colItem, 1, 1)
          index++

        }
      }
    }
  }
}
</script>
