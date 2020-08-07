<template> 
<div class="map">
  <header>
    <h1>疫情地图展示</h1>
  </header>
  <section class="map_content">
    <div class="map_tab">
      <div class="map_tab_top" :class="flagCon?'checked':''" @click="confirmed()">确诊病例</div>
      <div class="map_tab_top" :class="flagSus?'checked':''" @click="suspected()">死亡病例</div>
      <div class="map_tab_top" :class="flagCur?'checked':''" @click="cured()">新增确诊</div>
    </div>
    <div class="map_main" ref="mapbox"></div>  
  </section>
</div>
</template>

<script>
import echarts from 'echarts'
import 'echarts/map/js/china.js'
import jsonp from 'jsonp'
import * as R from 'ramda'
import dataGeo from '../../static/dataGeo'
import '../../static/dark'
import '../../static/macarons'
import '../../static/vintage'




export default {
  name: 'HelloWorld',
  props: {//组件的属性
    msg: String
  },
  data () {
    return {
    //   flag: false,
      flagCon: true,
      flagSus: false,
      flagCur: false,
      time: new Date(),
      timer: null, //定时器
      arr: [],
      option : {
        tooltip: {
            triggerOn: "click",
            formatter: "{b}:{c}"
        },
        visualMap: {
            min: 0,
            max: 1000,
            left: 26,
            bottom: 40,
            showLabel: !0,
            seriesIndex: [1],
            text: ["高", "低"],
            pieces: [{
                gte: 10000,
                label: "> 10000",
                color: "#b80909"
            }, {
                gte: 1000,
                lte: 9999,
                label: "1000 - 9999",
                color: "#e64546"
            },{
                gte: 100,
                lte: 999,
                label: "100 - 999",
                color: "#f57567"
            }, {
                gte: 10,
                lte: 99,
                label: "10 - 99",
                color: "#ff9985"
            }, {
                gte: 1,
                lte: 9,
                label: "1 - 9",
                color: "#ffe5db"
            }, {
                value: 0,
                color: "#ffffff"
            }],
            itemWidth:15,
            show: !0
        },
        geo: {
            show: true,
            map: "china",
            roam: !1,
            scaleLimit: {
                min: 1,
                max: 2
            },
            zoom: 1.23,
            top: 120,
            label: {
                normal: {
                    show: true,//显示地区名称
                    fontSize: "14",
                    color: "rgba(0,0,0,0.7)",
                    // formatter: '{b}\n{c}'
                },
                emphasis:{
                  show: true//高亮显示地区名称
                }
            },
            itemStyle: {
                normal: {
                    //shadowBlur: 50,
                    // shadowColor: 'rgba(0, 0, 0, 0.2)',
                    borderColor: "rgba(0, 0, 0, 0.2)"
                },
                emphasis: {
                    areaColor: "#f2d5ad",
                    shadowOffsetX: 0,
                    shadowOffsetY: 0,
                    borderWidth: 0
                }
            }
        },
        series: [{
            name: '散点',
            type: 'scatter',
            coordinateSystem: 'geo',
            data: [],
            symbolSize: 15,
            // symbolOffset:[0,'50%'],
            label: {
                normal: {
                    formatter: '{b}',
                    position: 'right',
                    show: false
                },
                emphasis: {
                    show: true
                }
            },
            itemStyle: {
                color: '#fff'
            }
        },{
            name: "确诊病例",
            type: "map",
            map: "china",
            geoIndex: 0,
            data: []
        },
        {
            name: '点',
            type: 'scatter',
            coordinateSystem: 'geo',
            zlevel: 6,
        },
        {
            name: '危险地区',
            type: 'effectScatter',
            coordinateSystem: 'geo',
            data: [],
            symbolSize: 15,
            // symbolOffset:[0,'50%'],
            showEffectOn: 'render',
            rippleEffect: {
                brushType: 'stroke'
            },
            hoverAnimation: true,
            label: {
                normal: {
                    formatter: '{b}',
                    position: 'left',
                    show: true
                }
            },
            itemStyle: {
                normal:{
                  color: 'yellow',
                  shadowBlur: 10,
                  shadowColor: 'yellow'
                }
            },
            zlevel: 1
        },]
      }
    }
  },
  mounted () {
    this.myChart = echarts.init(this.$refs.mapbox, 'macarons');
    this.getData();
    this.timer = setInterval(() => {//定时刷新
       setTimeout(this.getData, 0)
    }, 1000*60*60)
  },
  methods:{
    getData () {
      jsonp('https://voice.baidu.com/newpneumonia/get?target=trend&isCaseIn=0&stage=publish&callback=jsonp_1596355876712_14463', {}, (err, data) => {
        if (!err) {
        //   var Y = this.time.getFullYear();
          var M = R.inc(this.time.getMonth());
          var D = R.dec(this.time.getDate());
          var md = R.add(M, ".", D);  //与数据格式一致的当前日期
          // let arr = [];
            let  finalData;
            this.arr = [];
          for (var item of data.data) {
            var arr_date = item.trend.updateDate;
            var returnIndex = R.indexOf(md)(arr_date);
            var index = R.equals(returnIndex, -1)?R.dec(R.length(arr_date)):returnIndex;//当前日期的索引
            if(this.flagCon){
                finalData = item.trend.list[0];
                // console.log(finalData)
            }else if(this.flagSus) {
                finalData = item.trend.list[2];
                // console.log(finalData)
            }else if(this.flagCur) {
                finalData = item.trend.list[3];
                // console.log(finalData)
            }
            this.arr.push({
              name: item.name,
              value: R.nth(index, finalData.data)//当前日期对应的数据
              // value: finalData.data[index]
            });
          }
          // console.log(this.arr)
          // this.option.series[0].data = this.arr;
          
          this.option.series[0].data = this.getWarningData(this.arr);
          this.option.series[1].data = this.arr;
          this.option.series[3].data = this.getWarningData(this.arr);
          this.myChart.setOption(this.option);
          // echarts初始化的前提是dom渲染完成
          
          // console.log(this.arr)
          // console.log(this.flagCon)

        }
      })
    },
    getWarningData (data) {
      // console.log(this.arr)
      var res = [];
      for (var i = 0; i < data.length; i++) {
          var geoCoord = R.nth(i, dataGeo).value;// 地区经纬度
          if (R.gt(data[i].value, 1000)) {//超过1000人，设置警告
              res.push({
                  name: R.nth(i, data).name,
                  // name: data[i].name,
                  value: R.concat(geoCoord, [data[i].value]),
              });
          }
      }
      return res;
    },
    
    confirmed() {
        this.flagCon = true;
        this.flagSus = false;
        this.flagCur = false;
        this.getData();
    },
    suspected() {
        this.flagCon = false;
        this.flagSus = true;
        this.flagCur = false;
        this.getData();
    },
    cured() {
        this.flagCon = false;
        this.flagSus = false;
        this.flagCur = true;
        this.getData();
    }
  },
    beforeDestroy(){
        clearInterval(this.timer);        
        this.timer = null;
    }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.map{
  display: flex;
  flex-direction:column;
  justify-content: center;
  height: 100%;
}
header h1{
  padding: 20px 0;
  color: #fff;
  text-align: center;
}

.map_content{
  height: 850px;
  width: 100%;
  display: flex;
  flex-direction: column;
  justify-content: center;
  background-color: #f8f9fa;
  border-radius: 10px;
}
.map_content .map_tab{
  width: 95%;
  height: 40px;
  display: flex;
  flex-direction: row;
  border-radius: 10px;
  background-color: #ddd;
  margin: 16px auto;
  
}
.map_tab_top{
  flex: 1;
  line-height: 40px;
  border-radius: 10px;
  font-size: 16px;
  text-align: center;
  background-color: #ddd;
  cursor:pointer;
}
.map_main{
  flex: 1;
  width: 780px;
  height: 800px;
  background-color: #f8f9fa;
}
.map_tab .checked{
  background-color: #fff;
  font-weight: bold;
  border: 2px solid #ddd;
}
</style>
