<template>
<div v-if="rowData.layout &&(rowData.layout=='row' || rowData.layout=='column')" :style="{position: 'relative',flex:rowData.width?'0 0 auto':1,display:'flex'}">
  <div v-if="rowData.level==1 && rowData.index != colInfo.col-1" class="border-line right-line" @mousedown="mouseDownHandle($event,'rowRef','x')" >
    <div class="inner-line"></div>
  </div>
  <div ref="rowRef" :style="{height:'100%',backgroundColor:rowData.isSelected?'#ECF6FF':'transparent',width:rowData.width ? `${rowData.width}px`:'100%',display:'flex','flex-direction':rowData.layout=='row'?'':'column',overflow:'hidden'}" >
      <template v-for="item in rowData.rowItem">
        <table-row v-on="$listeners" :rowData="item"  :rowIndex="rowIndex" ></table-row>
      </template>
  </div>
  </div>
  <div v-else-if="!rowData.isMerged" :style="{ backgroundColor:rowData.isSelected?'#ECF6FF':'transparent', minHeight:`${colInfo.cellHeight}px`,minWidth:`${colInfo.cellWidth}px`,width:rowData.width ? `${rowData.width}px`:'100%',flex: rowData.width?'0 0 auto':1,position:'relative'}" ref="rowRef">
    <div  v-if="rowData.level==1 && rowData.index != colInfo.col-1" class="border-line right-line" @mousedown="mouseDownHandle($event,'rowRef','x')"><div class="inner-line"></div></div>
    <div  class="pre-center" @click="onHandleClick" style="border:.5px solid #333;margin:0;height:100%;boxSizing:border-box;overflow: hidden;">{{rowData.label}}</div>
  </div>

</template>
<script>

export default {
  name: 'TableRow',
  props: ['rowData', 'rowIndex'],
  inject: ['colInfo'],
  data() {
    return {
      canMove: false,
    }
  },
  created() {
  },

  methods: {
    mouseDownHandle(e, targetDom, direction) {
      if (targetDom && !this.canMove) {
        this.canMove = true
        this.$emit('getMaxWidth', { colIndex: this.rowData.index, rowIndex: this.rowIndex, initWidth: this.$refs.rowRef.clientWidth, startX: event.clientX, direction: direction })
        window.onmouseup = this.mouseUpHandle;
      }

    },
    mouseUpHandle() {
      this.canMove = false
      this.$emit('mouseUpHandle')
      window.onmouseup = null
    },

    onHandleClick() {
      this.$emit('clickCol', { colIndex: this.rowData.index, rowIndex: this.rowIndex, })
    },
  }
}
</script>
<style scoped>
.border-line{
  position: absolute;
}
.right-line{

  right: -3px;
  width: 6px;
  height: 100%;
  z-index: 10;
  background-color: transparent;


}
.right-line:hover{
    cursor: e-resize;
  }
  .inner-line{
     width: 1px;
  background-color: #333;
  height: 100%;
  margin: 0 auto;


  }
  .pre-center{
    display: flex;
    text-align: center;
    align-items: center;
    word-break: break-all;
    justify-content: center;
    font-size: 12px;
  }
</style>
