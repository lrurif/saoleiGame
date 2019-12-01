<template>
  <td :class="{'mine-clear':cellData.isClear,'marked':cellData.isMarked, 'boom':cellData.isShowBoom}" @mousedown="cellClick" class="cell" :style="{width: size.w+'px',height: size.h+'px'}">
      <template v-if="cellData.isClear">
          <span :style="{'color':colors[cellData.data]}">{{cellData.data||""}}</span>
      </template>
      
  </td>
</template>

<script>
import eventBus from "../eventBus.js"
export default {
    name:"cell",
    props: {
        size: {
             type: Object,
             required: false,
             default: function() {
                 return {
                     w:30,
                     h:30
                 }
             }
        },
        cellData: {
            type: Object,
            required: true,
        }
    },
    data() {
        return {
            colors:["#F0FFFF","#7FFFD4","#6495ED","	#FF00FF","#FF6A6A","#EE6363","#CD5555","#8B3A3A"]
        }
    },
    methods: {
        cellClick(e) {
            if(this.cellData.isClear) {
                return;
            }
            // this.$emit("app-cell-click");
            if(e.button === 0) {
                if(this.cellData.isBoom&& !this.cellData.isMarked) {
                //发送游戏结束标志
                eventBus.$emit("boom-end");
            }else {
                if(this.cellData.isMarked) {
                    return;
                }
                eventBus.$emit('click-cell');
                this.$emit('clearBoom',this.cellData.cellIndex);
            }
            }else if(e.button === 2) {
                eventBus.$emit('click-cell');
                this.cellData.isMarked = !this.cellData.isMarked;
                // this.$set(this.cellData, "isMarked", !this.celldata.isMarked);//有未知问题
                return;
            }
            
        }
    }
}
</script>

<style lang="scss">
    td {
        text-align: center;
    }
    .mine-clear {
        background: #363636!important;
    }
    .marked {
        background-image: url("../../public/flag.png")!important;
        background-size: cover!important;
    }
    .boom {
        background-image: url("../../public/bomb.png")!important;
        background-size: cover!important;
    }
</style>