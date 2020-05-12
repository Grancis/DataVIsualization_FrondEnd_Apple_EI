<template>
  <div class="app-container">
    <OptionBar ref="option" 
    @dataChanged="updateData" 
   />
    <title-of-project
      v-show="showVendorProjectTitle"
      :cellQualProjName="cellQual.projName"
      :cellQualVendorName="cellQual.vendorName"
    />
    <div ref="chartDv" style="width: 100%;height: 600px"></div>
  </div>
</template>

<script>
import OptionBar from "../components/OptionBar.vue";
import TitleOfProject from "../components/TItleOfProject.vue";
import echarts from "echarts";
import "echarts/dist/extension/dataTool";
//test plot
import * as dvApi from "@/api/ei/dv";
export default {
  name: "historgramChart",
  components: {
    OptionBar,
    TitleOfProject
  },
  data() {
    return {
      showVendorProjectTitle: false,
      cellQual: {
        projName: "",
        vendorName: ""
      },
      plotDataAll: []
    };
  },
  // mounted() {
  //     this.plotDataAll = dvApi.testData_json();
  //     this.initPlot();
  // },
  methods: {
    //将option组件request到的一些data传递到父组件使用
    updateData(param) {
      let option = this.$refs.option;
      this.cellQual.projName = option.cellQualProjName;
      this.cellQual.vendorName = option.cellqualVenderName;
      //目前只筛选dimNo
      let dimNo = option.searchMoreConditions.dimNo;
      if (dimNo != 0) {
        this.plotDataAll = option.totalData.filter(function(item) {
          return !(dimNo.indexOf(item.dim_no) == -1);
        });
      } else {
        this.plotDataAll = option.totalData;
      }
    },
 
    initPlot() {
      let chartDv = this.$refs.chartDv;
      let echart = echarts.init(chartDv);
      let plotdata = [];
      let xAxisData = [];
      for (var item of this.plotDataAll) {
        plotdata.push(item.projected_yeild);
        xAxisData.push(item.dim_no);
      }
    //   //use dataTool to format data
    //   let data = echarts.dataTool.prepareBoxplotData(plotdata);
    //   for (var i = 0; i < plotdata.length; i++) {
    //     data.boxData[i].push(this.plotDataAll[i].tol_max);
    //     data.boxData[i].push(this.plotDataAll[i].tol_min);
    //     data.boxData[i].push(this.plotDataAll[i].tol_sip_max);
    //     data.boxData[i].push(this.plotDataAll[i].tol_sip_min);
    //   }
      //set option to draw
      let option = {
        title: [
          {
            text: "Projected Yields",
            left: "5%",
            top: "2%",
            textStyle: {
              fontSize: 20,
              color: "black"
            }
          },
          {
            text: "Dim. No",
            //borderColor: '#999',
            //borderWidth: 1,
            textStyle: {
              fontSize: 14
            },
            left: "center",
            top: "90%"
          }
          // {
          // 	text: 'upper: Q3 + 1.5 * IRQ \nlower: Q1 - 1.5 * IRQ',
          // 	borderColor: '#999',
          // 	borderWidth: 1,
          // 	textStyle: {
          // 		fontSize: 14
          // 	},
          // 	left: '10%',
          // 	top: '90%'
          // }
        ],
        tooltip: {
          trigger: "item", //触发类型,数据项图形触发，主要在散点图，饼图等无类目轴的图表中使用。
          axisPointer: {
            //指示器类型。
            type: "shadow"
          }
        },
        grid: {
          //直角坐标系网格。
          //show: true,//default: false
          left: "10%",
          right: "10%",
          bottom: "15%"
          //borderWidth: 1,
          //borderColor: '#000',
        },
          toolbox: {
            show: true,
            feature: {
              dataZoom: {
                show: true
              },
              // dataView: { readOnly: false },
              // magicType: { type: ["line", "bar"] },
              restore: {show: true},
              saveAsImage: {show: true}
            }
          },       
        xAxis: {
          //X轴
          type: "category", //'category' 类目轴，适用于离散的类目数据，为该类型时必须通过 data 设置类目数据。
          //data: data.axisData,
          data: xAxisData,
          boundaryGap: true, //类目轴中 boundaryGap 可以配置为 true 和 false。默认为 true，这时候刻度只是作为分隔线，标签和数据点都会在两个刻度之间的带(band)中间。
          nameGap: 30, //坐标轴名称与轴线之间的距离。
          splitArea: {
            //坐标轴在 grid 区域中的分隔区域，默认不显示。
            //show: true, //是否显示分隔区域
            //interval: 'auto', //坐标轴分隔区域的显示间隔，在类目轴中有效
          },
          axisLabel: {
            //坐标轴刻度标签的相关设置。
            //formatter: 'expr {value}',  // 使用字符串模板，模板变量为刻度默认标签 {value}
            show: false, //是否显示刻度标签。
            //interval: 'auto', //坐标轴刻度标签的显示间隔，在类目轴中有效。
            color: "black"
          },
          splitLine: {
            //坐标轴在 grid 区域中的分隔线。
            show: true, //是否显示分隔线。默认数值轴显示，类目轴不显示。
            lineStyle: {
              //分隔线样式
              type: "dashed" //分隔线线的类型。
            }
          },
          axisLine: {
            //坐标轴轴线相关设置。
            show: true, //是否显示坐标轴轴线。
            //onZero:false,//X 轴或者 Y 轴的轴线是否在另一个轴的 0 刻度上，只有在另一个轴为数值轴且包含 0 刻度时有效。
            //symbol:'arrow', //轴线两边的箭头, 默认不显示箭头，即 'none'
            lineStyle: {
              //轴线样式
              width: 2,
              color: "black"
              //opacity: 1, //图形透明度。支持从 0 到 1 的数字，为 0 时不绘制该图形。
            }
          },
          axisTick: {
            //坐标轴刻度相关设置。
            show: true //是否显示坐标轴刻度。
            //alignWithLabel: true,//类目轴中在 boundaryGap 为 true 的时候有效，可以保证刻度线和标签对齐,default: false
          }
        },
        yAxis: {
          //y轴
          type: "value",
          splitArea: {
            //坐标轴在 grid 区域中的分隔区域，默认不显示。
            //show: true
          },
          axisLabel: {
            //坐标轴刻度标签的相关设置。
            //formatter: 'expr {value}',  // 使用字符串模板，模板变量为刻度默认标签 {value}
            show: true, //是否显示刻度标签。
            //interval: 'auto', //坐标轴刻度标签的显示间隔，在类目轴中有效。
            color: "black"
          },
          splitLine: {
            show: true,
            lineStyle: {
              type: "dashed"
            }
          },
          axisLine: {
            show: true, //是否显示坐标轴轴线。
            //onZero:false,//X 轴或者 Y 轴的轴线是否在另一个轴的 0 刻度上，只有在另一个轴为数值轴且包含 0 刻度时有效。
            //symbol:'arrow', //轴线两边的箭头
            lineStyle: {
              width: 2,
              color: "black"
            }
          }
        },
         dataZoom: [
            {
                show: true,
                start: 72,
                end: 100
            },
            {
                type: 'inside',
                start: 72,
                end: 100
            },
            {
                show: true,
                yAxisIndex: 0,
                filterMode: 'empty',
                width: 30,
                height: '80%',
                showDataShadow: false,
                left: '93%'
            }
        ],
        series: [
          {
            name: "projectected yield", //箱形图
            type: "bar",
            barGap:'200%',
            barCategoryGap:'60%',
            //legendHoverLink: true, //是否启用图例 hover 时的联动高亮。
            //hoverAnimation: false, //是否开启 hover 在 box 上的动画效果。

            itemStyle: {
                          normal: {
              color: function(params) {
                 var index_color = params.value;
                    if(index_color>=0.995){
                      return '#008000';
                        }
                    else if (index_color<0.995&&index_color>0.900){
                      return '#FFFF00';
                                    }
                    else{
                        return '#FF0000';
                    }
				},
				},
 
            },
            data: plotdata,
          },
        ]
      };
      echart.setOption(option);
    }
  },
  watch: {
    plotDataAll: function(newValue, oldValue) {
      this.initPlot();
    }
  }
};
</script>
