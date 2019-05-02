<template>
  <div class="mine-con" @contextmenu.prevent="flagSelect(-1,-1)">
    <div v-if="row>0" class="game-tags clearfix">
      <!--左侧剩余雷的数量-->
      <div class="game-tag-images">
        <h2 class="ft-red">{{pageCount}}</h2>
      </div>
      <!--笑脸 哭脸-->
      <div class="game-face">
        <div :class="isOver?'gameFaceCY':'gameFaceSM'" @click="reRunGame"></div>
      </div>
      <!--右侧时间-->
      <div class="game-time-images">
        <h2 class="ft-red">{{pageSecond}}</h2>
      </div>
    </div>
    <table>
      <tr v-if="row>0" v-for="i in row" :key="i">
        <td class="mine-grid" v-for="j in column" :key="j">
          <div @contextmenu.prevent="flagSelect(i-1,j-1)"
               @click="openGrid(i-1,j-1)"
               class="img-block"
               :class="calcClass(i-1,j-1)"></div>
        </td>
      </tr>
    </table>
  </div>
</template>

<script>
  export default {
    name: 'HelloWorld',
    data() {
      return {
        row:0,
        column:0,
        isOver:false,//是否游戏结束
        mSecond:0, //秒数
        mRow:16, //行数
        mColumn:32, //列数
        mInterval:null,
        mineCount:10, //雷的数量

        mines:null,//雷
        flags:null,//标记的旗子
        open:null,//打开的格子
        numbers:null, //每个格子 雷的数量
      }
    },

    /**
     * 初始化雷区
     */
    mounted(){
      this.row = 16;
      this.column = 32;
      this._initData();
      //生成雷区
      this.generateMines();

      //计算格子的雷数量
      this.calcMines();
      //开启计时
      this.recordTime();

    },
    computed:{
      /**
       * 页面显示时间
       */
      pageSecond(){
        let {mSecond} = this;
        if (mSecond<10) {
          mSecond = `00${mSecond}`;
        }else if(mSecond<100){
          mSecond = `0${mSecond}`;
        }
        return mSecond;
      },
      /**
       * 页面显示剩余雷的数量
       */
      pageCount(){
        let selectCount = 0;
        let flags = this.flags;
        for (let i = 0; i < flags.length ; i++) {
          for (let j = 0; j < flags[i].length; j++) {
            //如果这个格子已经打开了  需要忽略不计
            if (flags[i][j] && !this.open[i][j]){
              selectCount++;
            }
          }
        }
        let result = this.mineCount - selectCount;
        //是否完成游戏
        let flag =  true;
        if(result===0){
          //判断是否完成游戏
          for (let i = 0; i < flags.length ; i++) {
            for (let j = 0; j < flags[i].length; j++) {
              if (flags[i][j] && !this.mines[i][j]){
                flag = false;
                break;
              }
            }
          }
          if (flag){
            alert("恭喜你，完成游戏!");
          }
        }
        if (result<0) {
          result = `${result}`;
        } else if (result<10) {
          result = `00${result}`;
        }else if(result<100){
          result = `0${result}`;
        }
        return result;
      }
    },
    methods:{
      /**
       * 重新开始游戏
       */
      reRunGame(){
        this.restData();
        clearInterval(this.mInterval);

        this.isOver = false;
        this.row = 16;
        this.column = 32;
        this._initData();
        //生成雷区
        this.generateMines();

        //计算格子的雷数量
        this.calcMines();
        //开启计时
        this.recordTime();
      },
      /**
       * 右键事件
       */
      flagSelect(x,y){
        if (this.isOver) {
          return;
        }
        //检查是否越界
        if (!this.contain(x,y)){
          return;
        }
        this.flags[x][y] = !this.flags[x][y];
        let mFlags = this.flags;
        //重置数据 从新渲染 否则不生效
        this.row  = 0;
        this.flags = null;
        this.flags = mFlags;
        this.row  = this.mRow;
      },
      /**
       * 打开格子
       */
      openGrid(x,y){
        if (this.isOver) {
          return;
        }
        //判断是否点到雷  点到雷就Game Over
        if (this.mines[x][y]) {
          this.isOver = true;
          //打开所有  有雷的 格子
          let {mines,open} = this;
          for (let i = 0; i < mines.length; i++) {
            for (let j = 0; j < mines[i].length; j++) {
              if (mines[i][j]){
                this.open[i][j] = true;
              }
            }
          }
          clearInterval(this.mInterval);
          this.row  = 0;
          this.open = null;
          this.open = open;
          this.row  = this.mRow;
        }else{
          let tOpen = this.open;
          let tNumbers = this.numbers;

          this.openArea(x,y);
          this.row  = 0;
          this.open = null;
          this.numbers = null;
          this.open = tOpen;
          this.numbers = tNumbers;
          this.row  = this.mRow;
        }
      },
      /**
       * 打开一个区域
       * 如果是数字就只打开一个格子
       * 如果是空白 就FloodFill打开一个区域
       */
      openArea(x,y){
        if (!this.contain(x,y)){
          return;
        }
        if (this.open[x][y]){
          return;
        }
        //标记为访问过
        this.open[x][y] = true;

        //数字是边界
        if (this.numbers[x][y]==0){
          //遍历
          for (let i = x-1; i <= x+1 ; i++) {
            for (let j = y-1; j <= y+1; j++) {
              if (this.contain(i,j) && !this.open[i][j] && !this.mines[i][j]){
                this.openArea(i,j);
              }
            }
          }
        }
      },
      /**
       * 重置数据
       */
      restData(){
        this.row  = 0;
        this.mSecond = 0;
        this.mines = null;
        this.flags = null;
        this.open = null;
        this.numbers = null;
      },
      /**
       * 计算需要显示的类名
       */
      calcClass(x,y){
        let {mines,flags,open,numbers} = this;

        //判断格子是否打开了
        if (open[x][y]){
          if (mines[x][y]) {
            return 'mine';
          }else{
            //显示数字
            return `number-${numbers[x][y]}`;
          }
        } else {
          //是否被旗子 标记
          if (flags[x][y]) {
            //显示旗子
            return 'flag';
          }else{
            //显示格子
            return 'block';
          }
        }
      },
      /**
       * 根据雷的数量 随机分布雷
       */
      generateMines(){
        let {mineCount,row,column,mines} = this;

        //首先顺序把雷排列起来
        for (let i = 0; i < mineCount; i++) {
          let x = parseInt( i / row);
          let y = i % column;

          mines[x][y] = true;
        }
        let N = row;
        let M = column;

        //2 打乱位置
        for (let i = N*M -1; i >=0; i--) {
          let x = parseInt(i/N);
          let y = i%N;

          //放到另一个随机的位置
          let randX = parseInt((Math.random()*(i+1)));
          //转换 x y
          let newX = parseInt(randX/N);
          let newY = randX%N;
          this.swap(y, x,newY, newX);
        }
        this.mines = mines;
      },

      /**
       * 计算格子里面的雷数量
       */
      calcMines() {
        let N = this.row;
        let M = this.column;

        for (let i = 0; i < N; i++) {
          for (let j = 0; j < M; j++) {
            let temp = 0;
            //计算8个方向的雷总和
            for (let ii = i-1; ii <i+1; ii++) {
              for (let jj = j-1; jj <j+1; jj++) {
                if (this.contain(ii,jj) && this.mines[ii][jj]){
                  temp++;
                }
              }
            }
            this.numbers[i][j] = temp;
          }
        }
      },
      /**
       * 交换位置的数据
       */
      swap( x1, y1, x2, y2){
        let t = this.mines[x1][y1];
        this.mines[x1][y1] = this.mines[x2][y2];
        this.mines[x2][y2] = t;
      },
      /**
       * 计时
       */
      recordTime(){
        this.mInterval = setInterval(()=>{
          if(this.mSecond<=999){
            this.mSecond++;
          }else{
            this.mSecond=999;
            clearInterval(this.mInterval);
          }
        },1000);
      },
      /**
       * 这个坐标是否包含在区域内
       */
      contain(x,y){
        let N = this.row;
        let M = this.column;

        if (x>=0 && x<N && y>=0 && y<M){
          return true;
        }
        return false;
      },

      /**
       * 初始数组数据
       * @private
       */
      _initData(){
        let {row,column,mines,flags,open,numbers} = this;

        //初始化4个区域 雷区 标记区 数字区
        mines = new Array();
        flags = new Array();
        open = new Array();
        numbers = new Array();

        for (let i=0;i<row;i++){
          mines[i] = new Array();
          flags[i] = new Array();
          open[i] = new Array();
          numbers[i] = new Array();

          //添加二维
          for (let j=0;j<column;j++){
            //默认每个空格0
            numbers[i][j] = 0;
            //默认不是雷
            mines[i][j] = false;

            //默认没有打开和标记
            flags[i][j] = false;
            open[i][j] = false;
          }
        }
        this.mines = mines;
        this.flags = flags;
        this.open = open;
        this.numbers = numbers;
      },
    }
  }
</script>

<style scoped>
  *{
    border: none;
    margin: 0;
    padding: 0;
    border: 0;
  }
  .ft-red{
    color: red;
  }
  .mine-con{
    width: 707px;
    margin: 0 auto;
    background: #c0c0c0;
    padding: 7px;
    border-left: 2px solid #fff;
    border-top: 2px solid #fff;
  }
  .gameFaceSM{
    width: 100%;
    height: 100%;
    background-size: 100%;
    background-image: url("./imgs/face_normal.png");
  }
  .gameFaceCY{
    width: 100%;
    height: 100%;
    background-size: 100%;
    background-image: url("./imgs/face_fail.png");
  }
  .game-tags{
    border: 2px solid #808080;
    border-right-color: #fff;
    border-bottom-color: #fff;
    height: 34px;
    padding: 5px 6px;
    text-align: center;
    position: relative;
    margin-bottom: 8px;
  }
  .game-tag-images{
    float: left;
    width: 39px;
    height: 23px;
    text-align: center;
  }
  .game-face{
    border: 2px solid #808080;
    border-left-color: #fff;
    border-top-color: #fff;
    height: 20px;
    width: 20px;
    position: absolute;
    left: 50%;
    top: 50%;
    margin-left: -12px;
    margin-top: -12px;
  }
  .game-time-images{
    float: right;
    width: 39px;
    height: 23px;
  }
  /**每个格子大小*/
  .mine-grid{
    width: 20px;
    height: 20px;
  }
  .img-block{
    width: 100%;
    height: 100%;
    background-size: 100%;
  }
  /**雷*/
  .mine{
    background-image: url("./imgs/mine.png");
  }
  /**旗*/
  .flag{
    background-image: url("./imgs/flag.png");
  }
  /**格子*/
  .block{
    background-image: url("./imgs/block.png");
  }
  /**数量 0 - 8 */
  .number-0{
    background-image: url("./imgs/0.png");
  }
  .number-1{
    background-image: url("./imgs/1.png");
  }
  .number-2{
    background-image: url("./imgs/2.png");
  }
  .number-3{
    background-image: url("./imgs/3.png");
  }
  .number-4{
    background-image: url("./imgs/4.png");
  }
  .number-5{
    background-image: url("./imgs/5.png");
  }
  .number-6{
    background-image: url("./imgs/6.png");
  }
  .number-7{
    background-image: url("./imgs/7.png");
  }
  .number-8{
    background-image: url("./imgs/8.png");
  }
</style>
