<template>
<div>
  <div @click="mergeCol">合并</div>
  <div @click="splitCol">拆分</div>
   <div style="height:1500px;user-select: none; -moz-user-select: none; -webkit-user-select: none;"
    >
    <div class="center" style="height:1500px">
        <VueDragResizeGorks
            v-for="item in dragList"
            class="table-drag"
            :key="item.key"
            :parent="true"
            :active="item.activated"
            :snap="true"
            :snapTolerance="0"
            :x="item.x"
            :y="item.y"
            :w="item.w"
            :h="item.h"
            :resizable="false"
            @activated="onActivated(item)"
            @deactivated="onDeactivated(item)"
            @mousedown.native="onHandleClick(item)"
            :drag-handle="'.drag-handle'">
            <div class="drag-icon drag-handle" >
              拖
            </div>
            <div style="border:0.5px solid #333;width:100%">
                <Table2   :tableKey="item.key"  :ref="`dragItemRef${item.key}`"   @onStretchDrag="onStretchDrag"></Table2>
            </div>
        </VueDragResizeGorks>
    </div>
  </div>
</div>

</template>
<script>
import VueDragResizeGorks from 'vue-draggable-resizable-gorkys';
import Table2 from './table2/Table2.vue'
export default {
  components: {
    Table2,
    VueDragResizeGorks
  },
  created() {
  },
  destroyed() {
  },
  data() {
    return {
      dragParams: {},
      initDragHeight: 0,
      dragList: [
        {
          active: true,
          key: 1,
          x: 50,
          y: 150,
          w: 800,
          h: 140
        },
      ],
    }
  },
  methods: {
    onActivated(item) {
      item.active = true
    },
    onDeactivated(item) {
      item.active = false
    },
    onHandleClick(item) {
      this.dragParams = { key: item.key }
      this.initDragHeight = item.h
      item.active = !item.active
    },

    //合并列
    mergeCol() {
      this.$refs[`dragItemRef${this.dragParams.key}`][0].onMergeCol && this.$refs[`dragItemRef${this.dragParams.key}`][0].onMergeCol()
    },
    //拆分列
    splitCol() {
      this.$refs[`dragItemRef${this.dragParams.key}`][0].onSplitCol && this.$refs[`dragItemRef${this.dragParams.key}`][0].onSplitCol()
    },
    //拉伸drag
    onStretchDrag(moveY) {
      for (const item of this.dragList) {
        if (item.key == this.dragParams.key) {
          item.h = this.initDragHeight + moveY
          break
        }
      }
    },
  }
}
</script>
<style>
.drag-icon{
  position: absolute;
  top:-30px;
  left:-30px;
  width: 30px;
  height: 30px;
  line-height: 30px;
  border:1px solid #333;
  text-align: center;
  cursor: move;
}
.table-drag{
  border:0
}

</style>
