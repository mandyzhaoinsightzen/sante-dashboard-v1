<template>
<div class="container">
  <div class="main"> 
    <el-row class="pt-chart">
       <el-col :span="24" >
                <el-col :span="6">
                <h3 class="title-size">{{$t("homecontent.patientstats")}}</h3>
            </el-col>
            <el-col :span="18" class="align-right margin-right30">
                <span class="pt-title">{{$t("homecontent.totalnumberpt")}}</span>
                <!-- <span class="title-size mg-right50">{{allPatientsNumber}}</span> -->
                <span class="title-size mg-right50">345345345</span>
                <span class="pt-title">{{$t("homecontent.totalnumberast")}}</span>
                <!-- <span class="title-size">{{allPatientsAssessments}}</span> -->
                <span class="title-size">23423</span>
            </el-col>
        </el-col>
       <el-col :span="6">
        <h4>{{$t("homecontent.genderdistribtn")}}</h4>
        <div id="gender-pie-chart" class="chart-w-h"></div>
      </el-col>
      <el-col :span="6">
        <h4>{{$t("homecontent.agedistribtn")}}</h4>
        <div id="age-pie-chart" class="chart-w-h"></div>
      </el-col>
      
    </el-row>
  </div>
</div>

</template>
<script>
// 导入组件
import echarts from "echarts";

export default {
  name: 'index',
  data() {
    return {
      showtype: false
    }
  },
  // 注册组件
  components: {
     },
  methods: {
    drawGenderChart(data) {
      this.chPie = echarts.init(document.getElementById("gender-pie-chart"));
      var nameData = ["Males", "Females"];
      var option = {
        animation: true,
        title: {
          x: "center",
          y: "center",
          textStyle: {
            color: "red",
            fontSize: 60,
            fontWeight: "normal",
            align: "center",
            width: "200px"
          },
          subtextStyle: {
            color: "red",
            fontSize: 20,
            fontWeight: "normal",
            align: "center"
          }
        },
        legend: {
          width: "50%",
          left: "center",
          textStyle: {
            color: "#9B9B9B",
            fontSize: 16
          },
          icon: "circle",
          right: "0",
          bottom: "0",
          top:"75%",
          itemGap: 20,
          data: nameData
        },
        series: [
          {
            type: "pie",
            center: ["50%", "35%"],
            radius: ["30%", "40%"],
            color: ["#3A86D9", "#DD0C75"],
            startAngle: 135,
            labelLine: {
              normal: {
                length: 25
              }
            },
            label: {
              normal: {
                // formatter: "{c}\n{d}%",
                formatter: "\n{d}%",
                backgroundColor: "rgba(255, 147, 38, 0)",
                borderColor: "transparent",
                borderRadius: 4,
                textStyle: {
                  color: "#4a4a4a",
                  fontSize: 16
                }
              }
            },
            // data: data
            data:[
                {value:32, name:''},
                {value:23, name:''},
            ]
          }
        ]
      };
      this.chPie.setOption(option);
    },
    drawAgeChart(data1) {
      var agePie = echarts.init(document.getElementById("age-pie-chart"));
      var nameData = ["Under 20", "20-40", "40-60", "Above 60"];
      var option = {
        animation: true,
        title: {
          x: "center",
          y: "center",
          textStyle: {
            color: "red",
            fontSize: 60,
            fontWeight: "normal",
            align: "center",
            width: "200px"
          },
          subtextStyle: {
            color: "red",
            fontSize: 20,
            fontWeight: "normal",
            align: "center"
          }
        },
        legend: {
          width: "70%",
          left: "center",
          textStyle: {
            color: "#9B9B9B",
            fontSize: 16
          },
          icon: "circle",
          right: "0",
          bottom: "0",
          top:"75%",
          itemGap: 30,
          data: nameData
        },
        series: [
          {
            type: "pie",
            center: ["50%", "32%"],
            radius: ["30%", "40%"],
            color: ["#999DC5", "#8EC3B8", "#E19A4F", "#2F4186"],
            startAngle: 135,
            labelLine: {
              normal: {
                length: 25
              }
            },
            label: {
              normal: {
                formatter: "\n{d}%",
                backgroundColor: "rgba(255, 147, 38, 0)",
                borderColor: "transparent",
                borderRadius: 4,
                textStyle: {
                  color: "#4a4a4a",
                  fontSize: 16
                }
              }
            },
            // data: data1
            data:[
                {value:32, name:''},
                {value:23, name:''},
                {value:75, name:''},
                {value:91, name:''},
            ]
          }
        ]
      };
      agePie.setOption(option);
    },
  },
  mounted() {
    debugger;
    //  var data = res.response;
    //  var ageDistribution = data.ageDistribution;
    //  var assessmentDistribution = data.assessmentDistribution;
    // this.drawGenderChart(genderDistribution);
    // this.drawAgeChart(data.ageDistribution);
    this.drawGenderChart();
    this.drawAgeChart();
  },
 
}
</script>
<style scoped>
.container{
    text-align: center;
    overflow: hidden;
    margin: 0 auto;
    width: 80%;
    display: block;
}
.container .main{
    position: absolute;
    padding-top: 110px;
}
.pt-chart {
  background: #fff;
  
}
.pt-chart h4 {
    color: rgb(151, 151, 151);
    text-align: center;
  }
  .title-size {
    font-size: 24px;
    font-weight: bold;
    color: rgb(74, 74, 74);
    margin: 0px;
  }
  .pt-title {
    font-size: 16px;
    color: rgb(151, 151, 151);
  }
.chart-w-h {
    width: 100%;
    height: 350px;
  }
</style>

