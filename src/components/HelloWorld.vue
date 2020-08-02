<template> 
<div class="map">
  <header>
    <h1>疫情地图展示</h1>
  </header>
  <section class="map_content">
    <div class="map_tab">
      <div class="map_tab_top" :class="!flag?'checked':''">确诊病例</div>
      <div class="map_tab_top" :class="flag?'checked':''">疑似病例</div>
      <div class="map_tab_top" :class="flag?'checked':''">治愈病例</div>
    </div>
    <div class="map_main" ref="mapbox"></div>  
  </section>
</div>
</template>

<script>
import echarts from 'echarts'
import 'echarts/map/js/china.js'
import jsonp from 'jsonp'
window.dataList = [{
        name: "南海诸岛",
        value: 0
    },
    {
        name: '北京',
        value: 54
    },
    {
        name: '天津',
        value: 13
    },
    {
        name: '上海',
        value: 40
    },
    {
        name: '重庆',
        value: 75
    },
    {
        name: '河北',
        value: 13
    },
    {
        name: '河南',
        value: 83
    },
    {
        name: '云南',
        value: 11
    },
    {
        name: '辽宁',
        value: 19
    },
    {
        name: '黑龙江',
        value: 15
    },
    {
        name: '湖南',
        value: 69
    },
    {
        name: '安徽',
        value: 60
    },
    {
        name: '山东',
        value: 39
    },
    {
        name: '新疆',
        value: 4
    },
    {
        name: '江苏',
        value: 31
    },
    {
        name: '浙江',
        value: 104
    },
    {
        name: '江西',
        value: 36
    },
    {
        name: '湖北',
        value: 1052
    },
    {
        name: '广西',
        value: 33
    },
    {
        name: '甘肃',
        value: 7
    },
    {
        name: '山西',
        value: 9
    },
    {
        name: '内蒙古',
        value: 7
    },
    {
        name: '陕西',
        value: 22
    },
    {
        name: '吉林',
        value: 4
    },
    {
        name: '福建',
        value: 18
    },
    {
        name: '贵州',
        value: 5
    },
    {
        name: '广东',
        value: 98
    },
    {
        name: '青海',
        value: 1
    },
    {
        name: '西藏',
        value: 0
    },
    {
        name: '四川',
        value: 44
    },
    {
        name: '宁夏',
        value: 4
    },
    {
        name: '海南',
        value: 22
    },
    {
        name: '台湾',
        value: 3
    },
    {
        name: '香港',
        value: 5
    },
    {
        name: '澳门',
        value: 5
    }
];
export default {
  name: 'HelloWorld',
  props: {//组件的属性
    msg: String
  },
  data () {
    return {
      flag: false,
      time: new Date(),
      option : {
        tooltip: {
            triggerOn: "click",
            // formatter: function(e, t, n) {
            //     return .5 == e.value ? e.name + "：有疑似病例" : e.seriesName + "<br />" + e.name + "：" + e.value
            // }
        },
        visualMap: {
            min: 0,
            max: 1000,
            left: 26,
            bottom: 40,
            showLabel: !0,
            text: ["高", "低"],
            pieces: [{
                gt: 10000,
                label: "> 10000",
                color: "#660208"
            }, {
                gte: 1000,
                lte: 9999,
                label: "1000 - 9999",
                color: "#8C0D0D"
            },{
                gte: 100,
                lte: 999,
                label: "100 - 999",
                color: "#CC2929"
            }, {
                gte: 10,
                lt: 99,
                label: "10 - 99",
                color: "#FF7B69"
            }, {
                gt: 1,
                lt: 9,
                label: "1 - 9",
                color: "#FFAA85"
            }, {
                value: 0,
                color: "#ffffff"
            }],
            itemWidth:15,
            show: !0
        },
        geo: {
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
                    show: !0,
                    fontSize: "14",
                    color: "rgba(0,0,0,0.7)"
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
            name: "确诊病例",
            type: "map",
            geoIndex: 0,
            data: window.dataList
        }]
      }
    }
  },
  mounted () {
    this.getData()
    let myChart = echarts.init(this.$refs.mapbox);
    myChart.setOption(this.option);
  },
  methods:{
    getData () {
      jsonp('https://voice.baidu.com/newpneumonia/get?target=trend&isCaseIn=0&stage=publish&callback=jsonp_1596355876712_14463', {}, (err, data) => {
        if (!err) {
          console.log(data)
          var Y = this.time.getFullYear();
          var M = this.time.getMonth() + 1;
          var D = this.time.getDate();
          var day = (new Date(Y,M,D).getTime()-new Date(2020,1,26).getTime())/(1000*60*60*24);
          let arr = [];
          // var data = data.data
          for (var item of data.data) {
            // console.log(day)
            // console.log(item.trend.list[0].data[188])
            console.log(item.trend.list[0].data.length)
            arr.push({
              name: item.name,
              value: item.trend.list[0].data[day-1]
            });
          }
          console.log(arr)
          // let list = data.data.map(item => ({name: item.name, value: item.value}))
          // option.series[0].data = list
          // this.mycharts.setOption(option)
          // echarts初始化的前提是dom渲染完成
        }
      })
    }
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
