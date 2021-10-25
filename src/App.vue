<template>
  <div id="app">
    <div class="game">
      <div class="game-box">
        <div class="title-box">
          <span class="title">扫雷大战</span>
          <button>重置</button>
          <span>步数:{{clickNumber}}</span>
          <span>计时:{{time}}</span>
        </div>
        <div class="box">
          <ul>
            <li
              v-for=" (item,index) in number"
              :key="index"
              :value="index"
              @click="clickEvent(item.row,item.column)"
              @mouseup="mouseUpEvent($event,item.row,item.column)"
              @mousedown="mouseDownEvent(item.row,item.column)"
              @contextmenu.prevent="rightClickEvent()"
              @dblclick="dblClick(item.row,item.column)"
              :class = "{
                red:item.isBomb,
                canSee:item.canSee,
                isShow:item.isShow||item.canSee,
                baseColor:true}"
            >{{item.aroundNumber}}{{(item.flag)?'F':''}}</li>
          </ul>
        </div>
      </div>
    </div>
  </div>
</template>

<script>

export default {
  name: 'App',
  data () {
    return {
      number: [],
      bombNumber: 80,
      bombIndex: [],
      clickNumber: 0,
      time: '00:00',
      min: 0,
      max: 450,
      column: 30,
      row: 0
    }
  },
  created () {
    this.init()
  },
  methods: {
    init () {
      const n = 450
      this.row = Math.floor(n / this.column)
      for (let i = 0; i < n; i++) {
        let quotient = Math.floor(i / this.column)
        let remainder = i - (this.column * quotient)
        this.number.push({flag: false, canSee: false, isShow: false, isBomb: false, aroundNumber: null, column: (remainder + 1), row: quotient + 1})
      }
      // 部署炸弹
      const bombIndex = this.random(this.max, this.min, this.bombNumber)
      for (let i = 0; i < bombIndex.length; i++) {
        this.number[bombIndex[i]].isBomb = true
      }
      // 计算该点九宫格内有多少炸弹
    },
    clickEvent (row, column) {
      console.log('click')
      const currentPoint = this.getCurrentPoint(row, column)
      if (currentPoint.isBomb) {
        alert('失败！')
        return false
      }
      // 要判断是否点击过了
      if (currentPoint.isShow) return
      this.aroundCircle(row, column)
      if (this.clickNumber === 0) {
        // 第一次点击事件  即便是炸弹也不爆炸？
        this.clickNumber++
      } else {
        // 当前点击对象
        // const clickTarget = e.target
        this.clickNumber++
      }
      if (this.number.filter(item => {
        return item.aroundNumber === null
      }).length === this.bombNumber) {
        alert('胜利')
      }
      // let allFlagNumber=0
      // let allHideNumber=0
      // this.number.forEach(item=>{
      //   if(item.flag===true){allFlagNumber++}
      //   if(item.show===false){allHideNumber++}
      // })
      // console.log(allHideNumber,allFlagNumber)
      // if(allFlagNumber!=0||allHideNumber!=0){
      //   console.log('执行判断')
      //   if(allFlagNumber===allHideNumber){
      //     const win = this.number.every(item=>{
      //       item.flag==true&&item.show===false
      //     })
      //     if(win){
      //       alert('恭喜你！胜利！')
      //     }
      //   }
      // }
    },
    mouseDownEvent (row, column) {
      console.log('mouseDown')
      const currentPoint = this.getCurrentPoint(row, column)
      if (!currentPoint.isShow) {
        // 未揭示的点，不触发事件，只将这个点背景变色
        currentPoint.canSee = true
      } else {
        // 已揭示的点，统统变为背景色
        let flagPoints = this.around(row, column).flagPoints
        flagPoints.forEach(item => {
          item.canSee = true
        })
      }
    },
    mouseUpEvent (e, row, column) {
      const currentPoint = this.getCurrentPoint(row, column)
      if (e.button === 2) {
        console.log('右键')
        // 右键抬起触发
        // 如果已经被点击了，就走下面，如果没有，加棋子
        if (!currentPoint.isShow) {
          currentPoint.flag = !currentPoint.flag
          currentPoint.canSee = false
        }
      } else {
        // 没有被揭示的点触发点击事件
        if (!currentPoint.isShow) {
          // 未揭示的点，不触发事件，转为点击事件
          this.clickEvent(row, column)
        } else {
          // 还原颜色
          let aroundPoints = this.around(row, column).aroundPoints
          aroundPoints.forEach(item => {
            item.canSee = false
          })
        }
      }
    },
    rightClickEvent (row, column) {
      console.log(row, column)
      // const currentPoint = this.getCurrentPoint(row, column)
      // currentPoint.flag = !currentPoint.flag
    },
    dblClick (row, column) {
      console.log('dbclick')
      const count = this.around(row, column).count
      const aroundPoints = this.around(row, column).aroundPoints
      const flagPoints = this.around(row, column).flagPoints
      // 判断旗子是否等于count
      let flagCount = 0
      aroundPoints.forEach(item => {
        if (item.flag) {
          flagCount++
        }
      })
      if (count === flagCount) {
        // 对周围点执行点击
        const that = this
        flagPoints.forEach(item => {
          that.clickEvent(item.row, item.column)
        })
      }
    },
    getCurrentPoint (row, column) {
      return this.number[(row - 1) * this.column + column - 1]
    },
    around (row, column) {
      // 失败
      const currentPoint = this.getCurrentPoint(row, column)
      // 寻找周围的点
      let rowTmp = []
      let columnTmp = []
      if (row === 1) {
        rowTmp = [1, 2]
      } else if (row === this.row) {
        rowTmp = [row - 1, row]
      } else {
        rowTmp = [row + 1, row, row - 1]
      }
      if (column === 1) {
        columnTmp = [1, 2]
      } else if (column === this.column) {
        columnTmp = [column - 1, column]
      } else {
        columnTmp = [column + 1, column, column - 1]
      }
      let aroundPoints = []
      let flagPoints = []
      rowTmp.forEach((i) => {
        columnTmp.forEach(j => {
          if (!(row === i && column === j)) {
            // 排除已经标记的点
            let thisPoint = this.number[(i - 1) * this.column + j - 1]
            if (!thisPoint.flag) {
              flagPoints.push(this.number[(i - 1) * this.column + j - 1])
            }
            aroundPoints.push(this.number[(i - 1) * this.column + j - 1])
          }
        })
      })
      // 寻找炸弹
      const count = aroundPoints.filter(item => {
        return item.isBomb
      }).length
      currentPoint.aroundNumber = count
      return {count, aroundPoints, flagPoints}
    },
    aroundCircle (row, column) {
      const currentPoint = this.getCurrentPoint(row, column)
      const countAndPoints = this.around(row, column)
      console.log(row, column, 'aroundCircle')
      if (countAndPoints.count !== 0) {
        // 非零的点不扩散查询 展示一个点
        currentPoint.isShow = true
        return
      }
      currentPoint.isShow = true
      // 零点扩散查询
      countAndPoints.aroundPoints.forEach(item => {
        if (item.aroundNumber !== null) return
        let currentPoint = this.getCurrentPoint(item.row, item.column)
        currentPoint.isShow = true
        this.aroundCircle(item.row, item.column)
      })
    },
    random (max, min, bombNumber) {
      let arr = []
      for (let i = 0; i < bombNumber; i++) {
        min = Math.ceil(min)
        max = Math.floor(max)
        const n = Math.floor(Math.random() * (max - min)) + min
        if (arr.indexOf(n) === -1) {
          arr.push(n) // 不含最大值，含最小值
        } else {
          i--
        }
      }
      return arr
      // 随机数
    },
    countBomb () {

    }
  }
}
</script>

<style >
/* 禁止选中文字 */
*{
  -webkit-touch-callout: none;
  user-select: none;
}
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  margin-top: 60px;
  line-height: 30px;
}
.title-box{
  display: flex;
  justify-content: space-between;
}
button{
  width: 100px;
  height: 30px;
  background-color: cadetblue;
  border-radius: 4px;
  border:0;
  color:#fff;
}
.title{
  font-weight: bold;
  font-size: 16px;
  margin-bottom: 20px;
}
.game{
  display: flex;
  justify-content: center;
}
.box{
  width: 980px;
  height:500px;
}
ul{
  padding: 0;
  margin:0;
}
li{
  list-style: none;
  display:block;
  float: left;
  border:1px solid #ccc;
  width: 30px;
  height: 30px;
  padding: 0;
  margin:0;
  cursor: pointer;
  text-align: center;
  line-height: 30px;

}
li.red{
  background-color: red !important;
}
li.isShow{
  background-color: #fff !important;
}
li.baseColor{
  background-color: cadetblue;
}
</style>
