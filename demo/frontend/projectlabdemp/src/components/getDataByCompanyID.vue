<template>
  <div>
    <span>{{ serverResponse }} </span>
    <button @click="getData(); expand=!expand">GET DATA</button>
    <!-- <section v-if="expand">
      <figure>
        <v-chart
          :option="bar"
          ref="bar"
          theme="ovilia-green"
        />
      </figure>
    </section> -->
    <div id='test' :style="{width: '100%', height: '600px'}"></div>
  </div>

</template>

<script>
import axios from 'axios';
import * as echarts from 'echarts';
import ECharts from 'vue-echarts';

import { CanvasRenderer } from 'echarts/renderers';
import { BarChart } from 'echarts/charts';

const { use } = echarts;


// use([
//   CanvasRenderer,
//   BarChart
// ])

export default {
  name: 'bar-chart-test',
  components: {
    'v-chart': ECharts
  },
  data: function() {
    return {
      expand: false,
      // bar: this.getBar(),
      serverResponse: 'TEST',
      chartData: []
    }
  },
  methods: {
    testDraw() {
      var chartDom = document.getElementById('test');
      var myChart = echarts.init(chartDom)
      var option;
      var that = this;

      option = {
          title: {
            text: 'Company ID VS VaR and SVaR'
          },
          legend: {
            data:['VaR', 'SVaR']
          },
          xAxis: {
            type: 'category',
            data: that.chartData[0]
          },
          yAxis: {
              type: 'value'
          },
          tooltip: {
            trigger:'axis'
          },
          series: [
          {
              name: 'VaR',
              data: that.chartData[1],
              type: 'bar'
          },
          {
              name: 'SVaR',
              data: that.chartData[2],
              type: 'bar'
          }
          ]
      }
      myChart.setOption(option);
    },
    getData() {
      var that = this;
      that.chartData = [];
      // 对应 Python 提供的接口，这里的地址填写下面服务器运行的地址，本地则为127.0.0.1，外网则为 your_ip_address
      const path = "http://127.0.0.1:5000/getDataByCompanyID/" + this.$route.params.id;
      
      axios
        .get(path)
        .then(function(response) {
          // 这里服务器返回的 response 为一个 json object，可通过如下方法需要转成 json 字符串
          // 可以直接通过 response.data 取key-value
          // 坑一：这里不能直接使用 this 指针，不然找不到对象
          // var msg = response.data.msg;
          // 坑二：这里直接按类型解析，若再通过 JSON.stringify(msg) 转，会得到带双引号的字串
          // that.serverResponse = msg;
          // that.serverResponse = "Name of Company: " + response.data.name + "; X data:" + response.data.xdata + "; Ydata:" + response.data.ydata;
          let itemNames = response.data.item
          let data = response.data.data
          console.log("TESTTESTTEST", data)
          let chartData = []
          let x_data = []
          let y1_data = []
          let y2_data = []
          chartData.push(itemNames)
          for (let key in data) {
            let company = []
            if (data.hasOwnProperty(key)) {
              x_data.push(key)
              y1_data.push(data[key][0][1])
              y2_data.push(data[key][1][1])
              company.push(key)
              console.log(data[key])
              for (const item of data[key]) {
                console.log(item)
                company.push(item[1])
              }
            }
            chartData.push(company)
          }
          console.log(chartData)
          that.chartData.push(x_data,y1_data,y2_data)
          console.log(that.chartData)
          that.testDraw()
          return chartData
        })
        .catch(function(error) {
          alert('Error ' + error);
        });
    },
    getBar() {
      return {
        title: {
          text: "Test",
          left: "center"
        },
        dataset: {
          source: this.getData()
        },
        xAxis: { type: 'category' },
        yAxis: {},
        series: [
          { type: 'bar' },
          { type: 'bar' },
          { type: 'bar' }
        ]
      }
    }
  }
};
</script>

<style scoped>

figure {
  display: inline-block;
  position: relative;
  margin: 2em auto;
  border: 1px solid rgba(0, 0, 0, 0.1);
  border-radius: 8px;
  box-shadow: 0 0 45px rgba(0, 0, 0, 0.2);
  padding: 1.5em 2em;
  min-width: calc(40vw + 4em);
}

.echarts {
  width: 100%;
  width: 40vw;
  min-width: 400px;
  height: 400px;
}

.chart {
  height: 400px;
}

</style>
