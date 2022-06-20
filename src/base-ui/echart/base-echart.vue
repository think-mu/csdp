<template>
  <div class="echart">
    <div ref="echartDivRef" :style="{ width: width, height: height }" id="myChart"></div>
  </div>
</template>

<script>
import * as echarts from 'echarts'
import gzMap from '@/common/guangzhou.json'
export default {
  name: '',
  props: {
    width: {
      type: String,
      default: '100%'
    },
    height: {
      type: String,
      default: '300px'
    },
    options: {
      type: Object
    }
  },
  data() {
    return {
      echartInstance: null
    }
  },
  mounted() {
    this.$nextTick(() => {
      this.draw()
    })
    
  },
  activated () {
    // this.$refs.echartDivRef.refresh()
    // this.draw()
    // if(this.echartInstance) {
    //   setTimeout(() => {
    //     this.echartInstance.resize()
    //   },200)
    // }
  },
  methods: {
    draw() {
      let that = this;
      echarts.registerMap('gz', gzMap)
      // let echartInstance
      const echartDivRef = this.$refs.echartDivRef
      // var myChart =document.getElementById("myChart")
      // myChart.echartInstance.removeAttribute("_echarts_instance_");
      if (this.echartInstance != null && this.echartInstance != "" && this.echartInstance != undefined){
          this.echartInstance.dispose();
      }

       this.echartInstance = echarts.init(echartDivRef)
      //  this.echartInstance = echartInstance
      // echartInstance.dispose();
      // echartInstance.off("click").on('click', 'series.bar', function (params) {
      //   that.$emit('barClick',params)
      // });
      this.echartInstance.off('click')
      // echartInstance.getZr().off('click');
      this.echartInstance.on('click', 'series', function (params) {
        
      // const pointInPixel = [params.offsetX, params.offsetY];
      //   if (this.echartInstance.containPixel("grid", pointInPixel)) {
      //       let pointInGrid = this.echartInstance.convertFromPixel({
			// 			seriesIndex: 0
			// 		}, pointInPixel);
			// 		let xIndex = pointInGrid[0]; //索引
			// 		let handleIndex = Number(xIndex);
			// 		let seriesObj = this.echartInstance.getOption(); //图表object对象
      //      that.$emit('pieClick',seriesObj,handleIndex)
      //  } 
        that.$emit('pieClick',params)

      });
      // echartInstance.getZr().on("click", (params) => {
      //     that.$emit('pieClick',params)
      //     /* const pointInPixel = [params.offsetX, params.offsetY];
      //     if (echartInstance.containPixel("grid", pointInPixel)) {
      //         let xIndex = echartInstance.convertFromPixel({ seriesIndex: 0 }, [params.offsetX,params.offsetY])[0];//柱形的下标 ，此处取[0]
      //     } */
      // });
      // echartInstance.clear()
      
      // echartInstance.setOption(this.options,true)

      setTimeout(() => {
        try {
            this.echartInstance.setOption(that.options,true)
        } catch (error) {}
      }, 1000) //加延时,先获取数据在加载图表.

      window.addEventListener('resize', () => {
        this.echartInstance.resize()
      })
      
    }
  },
  watch: {
    options: { 
      deep:true,
      handler(newVal, oldVal) {
        this.draw()
        // this.$nextTick(() => {
        //   this.draw()
        // })
      },
      

    }
  },
}
</script>

<style scoped>

</style>
