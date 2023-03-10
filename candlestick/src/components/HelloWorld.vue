<template>
  <div id="main">

  </div>
</template>

<script>
import * as echarts from 'echarts';
import axios from "axios";
export default {
  name: 'HelloWorld',
  data(){
    return{
      rawData: [],
    }
  },
  mounted() {
    this.loadData()
  },
  methods: {
    loadData() {
      const options = {
        method: 'GET',
        url: 'https://twelve-data1.p.rapidapi.com/time_series??&start_date=2021-03-08&end_date=2023-03-08',
        params: {symbol: 'GSPC', interval: '1day', format: 'json'},
        headers: {
          'X-RapidAPI-Key': '6955f24879msh45ac4a7b39ccb29p18d9d9jsnd092a25c8c21',
          'X-RapidAPI-Host': 'twelve-data1.p.rapidapi.com'
        }
      };
      axios.request(options).then((response)=> {
        this.rawData = response.data.values;
        let data = this.splitData(response.data.values);
        this.drawGraph(data);
      }).catch(function (error) {
        console.error(error);
      });
    },
    splitData(raw){
      let categoryData = [];
      let values = [];
      let volumes = [];
      console.log(raw);
      for (let i = raw.length-1; i >=0; i--) {
        const date = raw[i].datetime
        const open = Number.parseFloat(raw[i].open).toFixed(3)
        const high = Number.parseFloat(raw[i].high).toFixed(3)
        const low = Number.parseFloat(raw[i].low).toFixed(3)
        const close = Number.parseFloat(raw[i].close).toFixed(3)
        const volume = Number.parseFloat(raw[i].volume).toFixed(3)
        categoryData.push(date);
        const temp = [open,close,low,high]
        values.push(temp);
        volumes.push([i, volume, open > close ? 1 : -1]);
      }
      return {
        categoryData: categoryData,
        values: values,
        volumes: volumes
      };
    },
    calculateMA(dayCount, data) {
      const result = [];
      for (let i = 0, len = data.values.length; i < len; i++) {
        if (i < dayCount) {
          result.push('-');
          continue;
        }

        let sum = 0;
        for (let j = 0; j < dayCount; j++) {
          let number = Number.parseInt(data.values[i - j][1])
          sum += number
        }
        result.push(+(sum / dayCount).toFixed(3));
      }
      return result;
    },
    drawGraph(data){
      const chartDom = document.getElementById('main');
      const myChart = echarts.init(chartDom);
      let option;

      const upColor = '#00da3c';
      const downColor = '#ec0000';
      option = {
        animation : false,
        legend: {
          bottom: 10,
          left: 'center',
          data: ['S&P index','MA50','MA200']
        },
        tooltip: {
          trigger: 'axis',
          axisPointer: {
            type: 'cross'
          },
          borderWidth: 1,
          borderColor: '#ccc',
          padding: 10,
          textStyle: {
            color: '#000'
          },
          position: function (pos, params, el, elRect, size) {
            const obj = {
              top: 10
            };
            obj[['left', 'right'][+(pos[0] < size.viewSize[0] / 2)]] = 30;
            return obj;
          }
          // extraCssText: 'width: 170px'
        },
        axisPointer: {
          link: [
            {
              xAxisIndex: 'all'
            }
          ],
          label: {
            backgroundColor: '#777'
          }
        },
        toolbox: {
          feature: {
            dataZoom: {
              yAxisIndex: false
            },
            brush: {
              type: ['lineX', 'clear']
            }
          }
        },
        brush: {
          xAxisIndex: 'all',
          brushLink: 'all',
          outOfBrush: {
            colorAlpha: 0.1
          }
        },
        visualMap: {
          show: false,
          seriesIndex: 5,
          dimension: 2,
          pieces: [
            {
              value: 1,
              color: downColor
            },
            {
              value: -1,
              color: upColor
            }
          ]
        },
        grid: [
          {
            left: '10%',
            right: '8%',
            height: '50%'
          },
          {
            left: '10%',
            right: '8%',
            top: '63%',
            height: '16%'
          }
        ],
        xAxis: [
          {
            type: 'category',
            data: data.categoryData,
            boundaryGap: false,
            axisLine: { onZero: false },
            splitLine: { show: false },
            min: 'dataMin',
            max: 'dataMax',
            axisPointer: {
              z: 100
            }
          },
          {
            type: 'category',
            gridIndex: 1,
            data: data.categoryData,
            boundaryGap: false,
            axisLine: { onZero: false },
            axisTick: { show: false },
            splitLine: { show: false },
            axisLabel: { show: false },
            min: 'dataMin',
            max: 'dataMax'
          }
        ],
        yAxis: [
          {
            scale: true,
            splitArea: {
              show: true
            }
          },
          {
            scale: true,
            gridIndex: 1,
            splitNumber: 2,
            axisLabel: { show: false },
            axisLine: { show: false },
            axisTick: { show: false },
            splitLine: { show: false }
          }
        ],
        dataZoom: [
          {
            type: 'inside',
            xAxisIndex: [0, 1],
            start: 98,
            end: 100
          },
          {
            show: true,
            xAxisIndex: [0, 1],
            type: 'slider',
            top: '85%',
            start: 98,
            end: 100
          }
        ],
        series: [
          {
            name: 'S&P index',
            type: 'candlestick',
            data: data.values,
            itemStyle: {
              color: upColor,
              color0: downColor,
              borderColor: undefined,
              borderColor0: undefined
            },
            tooltip: {
              formatter: function (param) {
                param = param[0];
                return [
                  'Date: ' + param.name + '<hr size=1 style="margin: 3px 0">',
                  'Open: ' + param.data[0] + '<br/>',
                  'Close: ' + param.data[1] + '<br/>',
                  'Lowest: ' + param.data[2] + '<br/>',
                  'Highest: ' + param.data[3] + '<br/>'
                ].join('');
              }
            }
          },
          {
            name: 'MA50',
            type: 'line',
            data: this.calculateMA(50, data),
            smooth: true,
            lineStyle: {
              opacity: 0.5
            }
          },
          {
            name: 'MA200',
            type: 'line',
            data: this.calculateMA(200, data),
            smooth: true,
            lineStyle: {
              opacity: 0.5
            }
          },
          {
            name: 'Volume',
            type: 'bar',
            xAxisIndex: 1,
            yAxisIndex: 1,
            data: data.volumes
          }
        ]
      };
      myChart.setOption(option,true);
      myChart.dispatchAction({
        type: 'brush',
        areas: [
          {
            brushType: 'lineX',
            coordRange: ['2023-03-01', '2023-03-08'],
            xAxisIndex: 0
          }
        ]
      });

    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
#main{
  height: 500px;
  width: 100%;
}
</style>
