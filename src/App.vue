<template>
  <div id="app">
    <el-dialog
  :title="title"
  :visible.sync="dialogVisible"
  width="20%"
  center>
  <span slot="footer" class="dialog-footer">
    <el-button type="primary" @click="againGame">再来一局</el-button>
  </span>
</el-dialog>
    <div class="container">
        <div class="main">
          <table class="mine-table">
            <tr v-for="(rowItem,rIndex) of rows" :key="'row_'+rIndex" >
              <!-- <td v-for="(colItem,cIndex) of cols" :key="'cols_'+cIndex" :style="{width: cellWidth+'px',height: cellHeight+'px'}"></td> -->
              <cell @clearBoom="clearBoom" v-for="(colItem,cIndex) of cols" :key="'cols_'+cIndex" :size="{w:cellWidth,h:cellHeight}" :cellData="cellArray[rIndex * cols + cIndex]"></cell>
            </tr>
          </table>
        </div>
        <div class="aside">
          <div>
            <p>难易程度</p>
            <el-radio-group @change="changeLevel" v-model="level">
              <el-radio-button label="简单"></el-radio-button>
              <el-radio-button label="中级"></el-radio-button>
              <el-radio-button label="高级"></el-radio-button>
            </el-radio-group>
          </div>
          <div class="block">
            <div class="block">
            <el-switch
              style="display: block"
              v-model="isRowCol"
              active-color="#13ce66"
              inactive-color="#ff4949"
              active-text="是否同步列数以及行数">
            </el-switch>
          </div>
            <span class="demonstration">列数: {{cols}}</span>
            <el-slider v-model="cols" :min=5 :max=100></el-slider>
          </div>
          <div class="block">
              <span class="demonstration">行数: {{rows}}</span>
              <el-slider v-model="rows" :min=5 :max=100></el-slider>
          </div>
          <div class="block">
            <el-switch
              style="display: block"
              v-model="isCellWHSync"
              active-color="#13ce66"
              inactive-color="#ff4949"
              active-text="是否同步宽高">
            </el-switch>
          </div>
          <div class="block">
              <span class="demonstration">行高: {{cellWidth}}</span>
              <el-slider v-model="cellWidth" :min=20></el-slider>
          </div>
          <div class="block">
              <span class="demonstration">列高: {{cellHeight}}</span>
              <el-slider v-model="cellHeight" :min=20></el-slider>
          </div>
          <div class="block">
            <el-button @click="initCellData" type="danger">重置</el-button>
          </div>
        </div>
      </div>
      <playsound srcSound="/click.mp3"></playsound>
      <timer></timer>
      <audio src="/boom.mp3" ref="boom"></audio>
    </div>
</template>

<script>
import "element-ui/lib/theme-chalk/index.css";
import Cell from "./components/Cell.vue"
import playsound from "./components/playSound.vue"
import timer from "./components/Timer"
import eventBus from "./eventBus"

export default {
  name: "app",
  components: {
    Cell,
    playsound,
    timer
  },
  data() {
    return {
      level: "简单",
      cols: 10,
      rows: 10,
      isCellWHSync: true,
      cellWidth:30,
      cellHeight:30,
      isRowCol: true,
      //{isBoom:false, data:2, isMarked:false, isClear:false}
      cellArray: [],
      isReset: true, //是否重置
      dialogVisible: false,
      openCell: new Set(),
      title: ""
    }
  },
  watch: {
    cellWidth(newVal) {
      this.isCellWHSync && (this.cellHeight = newVal);
    },
    cellHeight(newVal) {
      this.isCellWHSync && (this.cellWidth = newVal);
    },
    cols(newVal) {
      this.isRowCol && (this.rows = newVal);
      this.initCellData();
    },
    rows(newVal) {
      this.isRowCol && (this.cols = newVal);
      this.initCellData();
    }
    
  },
  created() {
    this.initCellData();
    document.oncontextmenu = ()=>{
      return false;
    }
    eventBus.$on("click-cell",()=> {
      this.isReset && eventBus.$emit("start-time");
      this.isReset = false;
    })
    let _this =this;
    eventBus.$on("boom-end", () => {
      _this.boomEnd();
    })
  },
  methods: {
    againGame() {
      this.dialogVisible = false;
      this.initCellData();
    },
    boomEnd() {
      this.$refs.boom.play();
      //显示所有雷区
      this.cellArray.forEach(item => {
        if(item.isBoom) {
          item.isShowBoom = true;
        }else {
          item.isClear = true;
        }
      })
      this.title = "抱歉，猜到地雷了"
      this.dialogVisible = true;
    },
    clearBoom(index) {
      const innerClear = cIndex=> {
        if(cIndex>=0 && cIndex < this.cellArray.length) {
          let cell = this.cellArray[cIndex];
          if(cell.isMarked || cell.isBoom || cell.isClear){
            return;
          }
            this.$set(this.cellArray[cIndex],'isClear',true);
            this.clearBoom(cIndex);
          }
      }
      let cell = this.cellArray[index];
      if(cell.data !== 0) {
        this.$set(cell,"isClear",true);
      }else {
        for(let i =-1;i<=1;i++) {
          let startIndex = index -i * this.cols -1;
          if(index % this.cols >0) {
            innerClear(startIndex);
          }
            innerClear(startIndex+1);
          if(index % this.cols <this.cols-1 ) {
            innerClear(startIndex+2);
          }
          
      }
      }
      this.openCell.add(index);
        if(this.openCell.size == Math.ceil((this.cols * this.rows)-Math.ceil(this.getLevelNum() * 0.15 * (this.cols * this.rows)))) {
          this.title = "恭喜你完成游戏！";
          this.dialogVisible = true;
        }
    },
    initCellData() {
      this.openCell = new Set();
      eventBus.$emit("init-time");
      this.isReset = true,
      this.cellArray = [];
      //初始化地雷数组
    let sum = this.cols * this.rows;
    let randomBoom = this.getLevelNum() * 0.15 * sum;
    let randomIndexSet = new Set();
    while(randomIndexSet.size < Math.ceil(randomBoom)) {
      randomIndexSet.add(Math.ceil(Math.random() * sum));
    }
    for (let i = 0;i<sum;i++) {
      let isBoom = randomIndexSet.has(i);
      let data = this.getBoomNum(i,randomIndexSet);
      this.cellArray.push({
        isBoom,
        isShowBoom:false,
        data,
        isMarked: false,
        isClear: false,
        cellIndex: i
      })
    }
    },
    changeLevel() {
      this.initCellData();
    },
    getBoomNum(index,boomSet) {
      let count = 0;
      for(let i =-1;i<=1;i++) {
        let startIndex = index -i * this.cols -1;
        if(index % this.cols >0) {
          count += boomSet.has(startIndex);
        }
        
        count += boomSet.has(startIndex+1);
        if(index % this.cols < this.cols -1) {
          count += boomSet.has(startIndex+2);
        }
        
      }
      return count;
    },
    getLevelNum() {
      if(this.level == '简单') {
        return 1;
      }else if (this.level == '中级') {
        return 2;
      }else {
        return 3;
      }
    }
  }
};
</script>

<style lang="scss">
#app {
  .container {
    display: flex;
  &> div {
    border: 1px solid #ccc;
  }
  .main {
    flex: 5 0 500px;
    min-height: 300px;
  }
  .aside {
    padding: 20px;
    flex: 2 0 200px;
    min-height: 300px;
    .block {
      // border-top: 1px solid blue;
      margin-top: 20px;
      // width: 80%;
    }
  }
  }
  .mine-table {
    border-collapse: collapse;
    td {
      width: 30px;
      height: 30px;
      border: 1px solid #c09;
      background: aliceblue;
    }
  }
}
</style>
