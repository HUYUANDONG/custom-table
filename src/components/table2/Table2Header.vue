<template>
<div>
  <div v-for="(item,rowIndex) in headerList" :style="{display:'flex',width:'100%',position: 'relative',height:item.height?item.height+'px':item.line * colInfo.cellHeight + 'px'}" :ref="`headerRef${rowIndex}`">
      <div class="bottom-line border-line" @mousedown="mouseDownHandle($event,`headerRef${rowIndex}`,rowIndex)">
            <div class="inner-line"></div>
      </div>
      <TableRow v-on="$listeners" v-for="headerItem in item.rowItem"   :key="headerItem.prop" :rowData="headerItem"  :rowIndex="rowIndex" ></TableRow>
  </div>
</div>

</template>
<script>
import TableRow from './TableRow.vue'
export default {
  name: 'Table2Header',
  props: ['col', 'headerList'],
  components: {
    TableRow
  },
    inject: ['colInfo'],

  data() {
    return {
      canMove: false,
    }
  },
  created() {

  },
  methods: {
    mouseDownHandle(e, targetDom, rowIndex) {
      if (targetDom && !this.canMove) {
        let initHeight = this.$refs[targetDom][0].clientHeight
        this.$emit('getMaxWidth', { rowIndex: rowIndex, initHeight, startY: event.clientY, direction: 'y', isHeader: true })
        window.onmouseup = this.mouseUpHandle;
      }
    },
    mouseUpHandle() {
      this.canMove = false;
      console.log('this.canMove: ', this.canMove);
      this.$emit('mouseUpHandle')
    },
  }
}
</script>
<style>
.border-line{
  position: absolute;

}
.bottom-line{
  bottom:0;
  width: 100%;
  height: 6px;
  background-color: transparent;
  z-index: 9;
}
.bottom-line:hover{
    cursor: s-resize;
  }
  inner-line{
     width: 100%;
     height: 1px;
  background-color: #333;
  margin: 0 auto;
  margin-top: 2.5px;


  }
</style>
