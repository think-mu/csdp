<template>
  <div class="enforce">
    <el-row :gutter="20">
      <el-col :span="11">
        <s-card title="监督执法统计" class="map">
          <template v-slot:select>
            <el-date-picker
              v-model="cYear"
              type="year"
              :clearable="false"
              popper-class="yselect"
              :picker-options="pickerOptions"
              :popper-append-to-body="false"
              @change="changeYear"
              placeholder="选择年份">
            </el-date-picker>
          </template>  
          <map-echart
            height="750px"
            :mapData="mapData"
            mapName="enforce"
          ></map-echart>
          <p class="down-title" @click="downLoad">导出Excel文档</p>
        </s-card>
      </el-col>
      <el-col :span="13" class="content-right">
        <s-card title="化妆品违法案件情况" class="content-right-pie">
          <template v-slot:select>
            <el-select
              v-model="value"
              @change="pieSelect"
              class="content-select"
              :popper-append-to-body="false"
            >
              <el-option
                v-for="item in options"
                :key="item.value"
                :label="item.label"
                :value="item.value"
              >
              </el-option>
            </el-select>
          </template>
          <pie-echart
            style="margin-left:10%;"
            height="335px"
            width="800px"
            legendWidth="480"
            :pieData="pieAllData"
            :pieStyleData ='pieStyle'
            :unit='pUnit'
          ></pie-echart>
        </s-card>
        <s-card title="化妆品违法案件情况" class="content-right-bar">
          <template v-slot:select>
            <el-select
              v-model="valueBar"
              @change="barSelect"
              :popper-append-to-body="false"
            >
              <el-option
                v-for="item in optionsBar"
                :key="item.value"
                :label="item.label"
                :value="item.value"
              >
              </el-option>
            </el-select>
          </template>
          <div class="content-right-bar-total">
            <span class="content-right-bar-total-label">{{ aLabel }}</span>
            <span class="content-right-bar-total-num">{{ aTotal }}</span>
          </div>
          <!-- <bar-echart
            height="280px"
            :xData="xData"
            :yData="yData"
            @barClick="barClick"
            v-show="!isShowIcon"
          ></bar-echart>-->
          <bar-echart height="325px" 
          :xData="xData" 
          :yData="yData"
          :unit='bUnit'
          >
          </bar-echart>
        </s-card>
      </el-col>
    </el-row>
  </div>
</template>

<script>
import qs from 'qs'
import SCard from '@/base-ui/card/card'
import PieEchart from '@/components/page-echart/src/pie-echart'
import BarEchart from '@/components/page-echart/src/bar-echart'
import MapEchart from '@/components/page-echart/src/map-echart'
import { mainInfo, exportByIds } from 'service/main/main'
import { levelData, convertData } from '@/utils/convert-data'

export default {
  name: '',
  components: {
    SCard,
    PieEchart,
    BarEchart,
    MapEchart
  },
  data() {
    return {
      mapData: [],
      pieAllData: [],

      xData: [], //柱形图x轴数据
      xData1: [],
      yData: [], //柱形图y轴数据
      yData1: [],
      isShowIcon: false, //是否现实柱形图返回按钮
      mainData: [],
      pUnit: '件',
      bUnit: '件 ',
      options: [
        {
          value: 'LANUM',
          label: '立案数量'
        },
        {
          value: 'YSNUM',
          label: '移送数量'
        },
        {
          value: 'MSJENUM',
          label: '没收金额'
        },
        {
          value: 'FKJENUM',
          label: '罚款金额'
        },
        {
          value: 'CFNUM',
          label: '处罚数量'
        },
        {
          value: 'JANUM',
          label: '结案数量'
        }
      ],
      optionsBar: [
        {
          value: 'LANUM',
          label: '立案数量'
        },
        {
          value: 'YSNUM',
          label: '移送数量'
        },
        {
          value: 'MSJENUM',
          label: '没收金额'
        },
        {
          value: 'FKJENUM',
          label: '罚款金额'
        },
        {
          value: 'CFNUM',
          label: '处罚数量'
        },
        {
          value: 'JANUM',
          label: '结案数量'
        }
      ],
      value: 'LANUM',
      valueBar: 'LANUM',
      aTotal: 0,
      aLabel: "案件总数（件）",//柱形图数量
      cYear: new Date(),
      txtYear: new Date().getFullYear(),
      pickerOptions: {
        disabledDate(time) {
          return (  //Date.now()
            time.getTime() > Date.now() ||
            time.getTime() < new Date('2012')
          )
        },
      },
      pieStyle: {  //饼形图样式
        imgPosition: '29%',
        line2: '75',
        piePosition: '44%'
      }
    }
  },
  created() {
    this.getMapInfo()

    // this.getLevelInfo()
    // this.getPieAraeData("A")
  },
  mounted() {},
  methods: {
    //下载文档
    downLoad({ vYear = this.txtYear } = {}) {
      const data = {
        action: 'punish',
        year: vYear,
        type: 'T02',
        level: 3
      }
      exportByIds(qs.stringify(data)).then((response) => {
        const blob = new Blob([response])
        const downloadElement = document.createElement('a')
        const href = window.URL.createObjectURL(blob) // 创建下载的链接
        downloadElement.href = href
        downloadElement.download = '监督执法列表.xlsx' // 下载后文件名
        document.body.appendChild(downloadElement)
        downloadElement.click() // 点击下载
        document.body.removeChild(downloadElement) // 下载完成移除元素
        window.URL.revokeObjectURL(href) // 释放掉blob对象
      })

    },
    /* 数据获取 start */
    //获取地图数据
    getMapInfo({ vYear = new Date().getFullYear() } = {}) {
      const data = {
        region: '',
        action: 'punish',
        year: vYear,
        type: 'T02',
        level: 1
      }
      mainInfo(qs.stringify(data)).then((res) => {
        this.mapData = res.data.map((item) => {
          return {
            name: item.REGIONNAME,
            value: item
          }
        })
        this.mainData = res.data
                
        this.getPieInfo('LANUM')
        this.getBarInfo('LANUM')
      })
    },
    //获取饼图数量
    getPieInfo(val) {
      if(val=="MSJENUM" || val =="FKJENUM") {
        // this.aLabel = "金额总数（元）"
        this.pUnit = '元'
      }else{
        // this.aLabel = "案件总数（件）"
        this.pUnit = '件'
      }
      this.pieAllData = this.mainData.map((item) => {
        return {
          name: item.REGIONNAME,
          value: item[val]
        }
      })
    },
    getBarInfo(val) {
      if(val=="MSJENUM" || val =="FKJENUM") {
        this.aLabel = "金额总数（元）"
        this.bUnit = '元'
      }else{
        this.aLabel = "案件总数（件）"
        this.bUnit = '件'
      }
      this.xData = this.mainData.map((item) => {
        return {
          value: item.REGIONNAME
        }
      })
      this.yData = this.mainData.map((item) => {
        return {
          value: item[val]
        }
      })
      let formatData = this.mainData.map((item) => {
        return item[val]
      }).reduce((prev,curr) => {
          return this.formatNum(Number(prev),Number(curr),"+",2)
      })
      this.aTotal = formatData
    
    },
    formatNum(add, reduce, s, num) { //消除加和小数点误差
         let m = Math.pow(10, num); // num是10的次幂
         let res = s == '+' ? (add * m + reduce * m) / m : (add * m - reduce * m) / m;
         return (res * m) / m;
     },

    /* 数据获取 end */
    /* 饼形图事件 start*/
    //全市分级饼形图--点击事件
    pieSelect(param) {
      this.getPieInfo(param)
    },
    /* 饼形图事件 end*/

    /* 柱形图事件 start*/
    barSelect(param) {
      this.getBarInfo(param)
    },
    changeShowIcon(val) {
      this.isShowIcon = val
    },
    /* 柱形图事件 end*/
    changeYear(param) {
      this.getMapInfo({ vYear: param.getFullYear() })
      this.txtYear = param.getFullYear()
      this.value = 'LANUM'
      this.valueBar = 'LANUM'
    }
  }
}
</script>

<style scoped lang="less">
.map {
  position: relative;
  .down-title {
    position:absolute;
    bottom: 15px;
    left: 20px;
    color: #FFAE00;
    font-size: 16px;
  }
}
::v-deep {
  .el-input__inner {
    background-color: rgba(166, 255, 0, 0) !important;
    border: 1px transparent solid;
    border-image: linear-gradient(
        to right,
        rgba(4, 127, 156, 1),
        #e9eceb,
        #01cfff
      )
      1 10;
    box-shadow: 0px 0px 3px 1px #01cfff;
    z-index: 10;
    color: #fff;
    font-size: 16px;
    width: 180px;
    text-align: left;
  }
  .el-select-dropdown {
    background-color: rgba(20, 74, 116, 0);
    .el-scrollbar {
      background-color: rgba(20, 74, 116, 0.705);
      color: #fff !important;
      border: 1px transparent solid;
      border-image: linear-gradient(
          to right,
          rgba(4, 127, 156, 1),
          #e9eceb,
          #01cfff
        )
        1 10;
      box-shadow: 0px 0px 3px 1px #01cfff;
      .el-scrollbar__view {
        color: #fff !important;
        .el-select-dropdown__item {
          font-size: 16px;
          text-align: left;
          color: #fff !important;
        }
        .hover {
          background-image: linear-gradient(
            to right,
            rgba(2, 157, 193, 1),
            rgb(3, 64, 110)
          ) !important;
        }
        .selected {
          font-weight: bold;
          color: #00efff !important;
        }
      }
    }
  }
}
// /deep/ .el-scrollbar{
//   background-color: rgba(20, 74, 116,.8);
//   color: #fff !important;
//   border:1px transparent solid;
//   border-image:linear-gradient(to right,rgba(4,127,156,1),#e9eceb,#01cfff) 1 10;
//   box-shadow: 0px 0px 3px 1px #01cfff;
// }

.enforce {
  // height: 100%;
  .el-row,
  .el-col {
    height: 100% !important;
  }

  .content-right {
    height: 100%;
    display: flex;
    flex-direction: column;

    .content-select {
      // background-color: rgb(255, 0, 0) !important;
    }
    &-pie {
      margin-bottom: 15px;
    }
    &-bar {
      position: relative;
      &-total {
        position: absolute;
        top: 70px;
        left: 56px;
        width: 312px;
        height: 73px;
        background: url('../../../assets/img/card/total-bg.png') no-repeat;
        background-size: 100% 100% !important;
        display: flex;
        align-items: center;
        font-size: 14px;
        text-align: center;
        justify-content: center;
        &-label {
          background-image:-webkit-linear-gradient(bottom,#6EDBFF,#FFFFFF);
          font-weight: bold;
          -webkit-background-clip:text;
          -webkit-text-fill-color:transparent;
        }
        &-num {
          font-weight: bold;
          text-shadow: 2px 2px 22px #52d5ec;
          // text-shadow: 0px 2px 0px #fff, 0 0 2px #fff, 0 0 0px #fff, 0 0 4px #52d5ec, 0 0 4px #52d5ec;
          font-size: 22px;
          color: #FFFFFF;
        }
      }
    }
  }
}
</style>
