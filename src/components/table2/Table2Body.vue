<template>
    <div>
        <div v-for="(bodyItem,rowIndex) in bodyList" :ref="`row${rowIndex}`" class="row-box" :style="{display:'flex',width:'100%',height:bodyItem.height?bodyItem.height+'px':bodyItem.line * colInfo.cellHeight + 'px'}">
          <div class="bottom-line border-line" @mousedown="mouseDownHandle($event,'row'+rowIndex,rowIndex)">
              <div class="inner-line"></div>
          </div>
          <TableRow  v-on="$listeners" v-for="item in bodyItem.rowItem" :key="item.prop" :rowData="item"  :rowIndex="rowIndex"></TableRow>
        </div>
    </div>
</template>
<script>
import TableRow from './TableRow.vue'
export default {
  name: 'Table2Body',
  props: ['bodyList'],
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
        this.$emit('getMaxWidth', { rowIndex: rowIndex, initHeight, startY: event.clientY, direction: 'y', isHeader: false })
        window.onmouseup = this.mouseUpHandle;
      }
    },

    mouseUpHandle(e) {
      console.log('111')
      this.canMove = false;
      this.$emit('mouseUpHandle')
       window.onmouseup =null

    }
  }
}
</script>
<style scoped>

.row-box{
  position: relative;
}

.bottom-line{
  bottom:-3px;
  width: 100%;
  height: 6px;
  z-index: 9;
  background-color: transparent;
}
.border-line{
  position: absolute;

}
.bottom-line:hover{
    cursor: s-resize;
  }
.inner-line{
     width: 100%;
     height: 1px;
  background-color: #333;
  margin: 0 auto;
  margin-top: 2.5px;


  }
</style>
