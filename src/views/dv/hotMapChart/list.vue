<template>
  <div class="app-container">
    <OptionBar ref="option" @dataChanged="updateData" chartType="hotmap"/>
    <title-of-project
      v-show="showVendorProjectTitle"
      :cellQualProjName="cellQual.projName"
      :cellQualVendorName="cellQual.vendorName"
      :cellQualDate="updateDate"
    > Hotmap </title-of-project>
    <el-row v-if="show"> 
      <el-col :span="4"><p class="ruleName" style="margin: 8px 0px">Rules:</p></el-col>
      <el-col :span="20">
        <!-- <div style="margin-top: 8px"> -->
          <!-- <el-radio-group v-model="radio" size="small" style="margin-top: 8px"> -->
            <el-radio-group v-model="radio" size="small">
            <el-radio-button label="Dim Risk Level" ></el-radio-button>
            <el-radio-button label="Dim Outlier Detection"></el-radio-button>
            <el-radio-button label="Dim Dispersion Detection"></el-radio-button>
            <el-radio-button label="Dim Deviation Detection"></el-radio-button>
          </el-radio-group>
          <el-button style="margin: 0px 16px" type="primary" @click="clickUserCheckButton()">User Check</el-button>
          <el-radio-group size="small" v-model="showOriginOrUpdate">
            <el-radio-button label="Origin"></el-radio-button>
            <el-radio-button label="Update"></el-radio-button>
          </el-radio-group>
        <!-- </div> -->
      </el-col>
    </el-row>
    <div ref="chartDv" style="width: 100%;height: 1000px;">
    </div>
    <el-dialog
      :title="title"
      :visible.sync="dialogVisible"
      :close-on-click-modal="false"
    ><!-- close-on-click-modal="false"做到在弹出dialog时，外部呈锁定状态，而不是点击外部导致dialog直接关闭 -->
      <el-row>
        <el-col :span="24">
          <el-card :body-style="{padding:'2px'}">
          <!-- <div> -->
            <div slot="header" class="clearfix">
              <span style="font-weight:bold;">{{radio}} User Check Table</span>
              <el-button
                type="text"
                style="float: right;padding: 3px 0"
                @click="popRiskSuggestDialog()"
              >New Record</el-button>
            </div>
            <el-table :data="userCheckTableData" border style="width: 100%">
              <el-table-column prop="machine" label="Machine"></el-table-column>
              <el-table-column prop="faiNo" label="FAI No"></el-table-column>
              <el-table-column prop="riskLevel" label="Risk Level"></el-table-column>
              <el-table-column prop="check" label="Check"></el-table-column>
            </el-table>
          <!-- </div> -->
          </el-card>
        </el-col>
      </el-row>
    </el-dialog>
    <el-dialog
    title="New Risk Record"
    :fullscreen="false"
    :modal="false"
    :visible.sync="showRiskInput"
    @close='closeDialog'
    style="right:0px;"
    >
      <el-row style='text-align:center;'> 
        <el-col :span="6">
          <p>Machine</p>
        </el-col>
        <el-col :span="6">
          <p>FAI No</p>
        </el-col>
        <el-col :span="6">
          <p>Risk Level</p>
        </el-col>
        <el-col :span="6">
          <p>Check</p>
        </el-col>
      </el-row>
      <el-row :gutter="20" v-for="item in userCheckForm">
        
        <el-col :span="6">
          <!-- <el-input v-model="riskInput.machine" placeholder="Machine"></el-input> -->
          <!-- v-model="machineInput" -->
          <el-autocomplete
            class="inline-input"
            v-model="item.machine_no"
            :fetch-suggestions="((queryString,cb)=>{querySearch(queryString,cb,'cnc_no')})"
            placeholder="Machine"
            clearable
          ></el-autocomplete>
        </el-col>
        <el-col :span="6">
          <!-- <el-input v-model="riskInput.faiNo" placeholder="FAI No"></el-input> -->
          <!-- v-model="faiNoInput" -->
          <el-autocomplete
            class="inline-input"
            v-model="item.fai_no"
            :fetch-suggestions="((queryString,cb)=>{querySearch(queryString,cb,'fai_no_list')})"
            placeholder="FAI No"
            clearable
          ></el-autocomplete>
        </el-col>
        <el-col :span="6">
          <!-- <el-input v-model="riskInput.riskLevel" placeholder="Risk Level"></el-input> -->
          <!-- <p style="text-align:center;margin:0px;line-height:29px;">{{ riskLevel }}</p> -->
          <p style="text-align:center;margin:0px;line-height:29px;">{{item.risk_level}}</p>
        </el-col>
        <el-col :span="6">
          <!-- <el-select v-model="checkInput" clearable placeholder="Check"> -->
          <el-select v-model="item.chk_result" clearable placeholder="Check">
            <el-option
              v-for="value in checkList"
              :key="value"
              :label="value"
              :value="value">
            </el-option>
          </el-select>
        </el-col>
      </el-row>
      <div slot="footer" class="dialog-footer">
        <!-- <el-col :span="2"> -->
          <el-button @click="addOneFormRow()">+
          </el-button>
        <!-- </el-col>     -->
        <!-- <el-col :span="2"> -->
          <el-button @click="minusOneFormRow()">-
          </el-button>
        <!-- </el-col>    -->
        <el-button style="right:8px;" @click="closeRiskSuggestDialog()">Cancel</el-button>
        <el-button type="primary" style="right:0px;" @click="submitUserCheck()">Submit</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import OptionBar from "../components/OptionBar.vue";
import TitleOfProject from "../components/TItleOfProject.vue";
import echarts from "echarts";
import "echarts/dist/extension/dataTool";
import axios from 'axios'
// import testData from './heat_map_response.json'

import * as dvApi from "@/api/ei/dv";

export default {
  name: "hotMapChart",
  components: {
    OptionBar,
    TitleOfProject
  },
  data() {
    return {

      num2riskLevel:{
        '0.0':'ok',
        '1.0':'ok*',
        '2.0':'SIP Alert +',
        '-2.0':'SIP Alert -',
        '2.5':'Drawing Alert +',
        '-2.5':'Drawing Alert -',
        '3.0':'SIP Reject +',
        '-3.0':'SIP Reject -',
        '4.0':'Drawing Reject +',
        '-4.0':'Drawing Reject -'
      },
      num2outlier:{
        '0.0':'NaN',
        '1.0':'outlier_ok',
        '2.0':'outlier_ok*',
        '3.0':'outlier_potential',
        '4.0':'outlier_potential*',
        '5.0':'outlier_risk',
        '6.0':'outlier_risk*'
      },
      num2dispersion:{
        '0.0':'NaN',
        '1.0':'dispersion_ok',
        '2.0':'dispersion_ok*',
        '3.0':'dispersion_risk',
        '4.0':'dispersion_risk*'
      },
      num2deviation:{
        '0.0':'NaN',
        '1.0':'deviation_ok',
        '2.0':'deviation_ok*',
        '3.0':'deviation_risk',
        '4.0':'deviation_risk*'
      },
      label2variable:{
        "Dim Risk Level":"dim_risk_level",
        "Dim Outlier Detection":"dim_outlier_detection",
        "Dim Dispersion Detection":"dim_dispersion_detection",
        "Dim Deviation Detection":"dim_deviation_detection"
      },
      // riskLevel2num:{
      //   'ok':0.0,
      //   'ok*':1.0,
      //   'SIP Alert +':2.0,
      //   'SIP Alert -':-2.0,
      //   'Drawing Alert +':2.5,
      //   'Drawing Alert -':-2.5,
      //   'SIP Reject +':3.0,
      //   'SIP Reject -':-3.0,
      //   'Drawing Reject +':4.0,
      //   'Drawing Reject -':-4.0
      // },
      // outlier2num:{
      //   'NaN':0.0,
      //   'outlier_ok':1.0,
      //   'outlier_ok*':2.0,
      //   'outlier_potential':3.0,
      //   'outlier_potential*':4.0,
      //   'outlier_risk':5.0,
      //   'outlier_risk*':6.0
      // },
      // dispersion2num:{
      //   'NaN':0.0,
      //   'dispersion_ok':1.0,
      //   'dispersion_ok*':2.0,
      //   'dispersion_risk':3.0,
      //   'dispersion_risk*':4.0
      // },
      // deviation2num:{
      //   'NaN':0.0,
      //   'deviation_ok':1.0,
      //   'deviation_ok*':2.0,
      //   'deviation_risk':3.0,
      //   'deviation_risk*':4.0
      // },
      title:'Dim Risk Level Checked by User',
      showVendorProjectTitle: false,

      headerId:'',
      dataRound:'',

      cellQual: {
        projName: "",
        vendorName: ""
      },
      plotDataAll: [],
      plotChkData:[],
      radio: "Dim Risk Level",
      showOriginOrUpdate: 'Origin',
      show: false,
      showRuleData: [],
      updateDate: "",
      dialogVisible: false,
      showRiskInput: false,
      
      machineInput: '',
      faiNoInput:'',
      riskLevel:'None',
      checkInput:'',
      checkList: [],
      userCheckTableData: [],
      userCheckForm:[]
    };
  },
  mounted() {
      // this.plotDataAll = dvApi.testData_json();
      // this.initPlot();
      this.getCheckList('Dim Risk Level')
      // console.log('init')
  },
  methods: {
    //将option组件request到的一些data传递到父组件使用
    popRiskSuggestDialog() {
      this.showRiskInput = true;
      this.userCheckForm=[{
        "header_id":this.headerId,
        "data_round":this.dataRound,
        "machine_no":"",
        "fai_no":"",
        "rule_name":this.label2variable[this.radio],
        "risk_level":"None",
        "chk_result":""
      }]
    },
    closeRiskSuggestDialog() {
      this.showRiskInput = false
      // this.userCheckForm = []
    },
    submitUserCheck() {
      // let data=[
      //   {
      //     header_id: this.headerId,
      //     machine_no: this.machineInput,
      //     fai_no: this.faiNoInput,
      //     rule_name: this.label2variable[this.radio],
      //     chk_result:this.checkInput
      //   }
      // ]
      dvApi
        .pushUserDefinedHotmapRules(this.userCheckForm)
        .then(res => {
          // this.plotChkData=res
          // console.log(res.msg)
          alert(res.msg)
          this.showRiskInput = false
          // this.userCheckForm=[]
        })
        .catch(err => {
          //暂时用alert
          alert(err);
        });
      dvApi
        .getHotMapChkresult(this.headerId)
        .then(res => {
          this.plotChkData=res
          // console.log(res.msg)
        })
        .catch(err => {
          //暂时用alert
          alert(err);
        });
    },
    querySearch(queryString, callback, key) {
      // let key = 'cnc_no'
      let list
      list=this.plotDataAll[this.label2variable[this.radio]][key]
      // if (this.radio=="Dim Risk Level"){
      //   list = this.plotDataAll["dim_risk_level"][key]
      // }
      // if (this.radio=="Dim Outlier Detection"){
      //   list = this.plotDataAll["dim_outlier_detection"][key]
      // }
      // if (this.radio=="Dim Dispersion Detection"){
      //   list = this.plotDataAll["dim_dispersion_detection"][key]
      // }
      // if (this.radio=="Dim Deviation Detection"){
      //   list = this.plotDataAll["dim_deviation_detection"][key]
      // }
      let searchList=[]
      for(let item of list) {
        searchList.push({'value': item})
      }
      // console.log(searchList)
      var results = queryString ? searchList.filter(this.createFilter(queryString)) : searchList;
      // 三目运算符，若queryString存在，执行前者，否则执行后者
      // console.log(results)
      // 调用 callback 返回建议列表的数据
      callback(results);
    },
    createFilter(queryString) {
      return (item) => { //箭头函数，传入单个item判断后返回true或者false，===0为按顺序查找， >-1为模糊查找
        return (item.value.toLowerCase().indexOf(queryString.toLowerCase()) > -1);
      };
    },
    searchRiskLevel(){
      let data
      let num2rule
      data = this.plotDataAll[this.label2variable[this.radio]]["data"]
      // if (this.radio=="Dim Risk Level"){
      //   data = this.plotDataAll["dim_risk_level"]["data"]
      //   // num2rule=this.num2riskLevel
      // }
      // if (this.radio=="Dim Outlier Detection"){
      //   data = this.plotDataAll["dim_outlier_detection"]["data"]
      //   // num2rule=this.num2outlier
      // }
      // if (this.radio=="Dim Dispersion Detection"){
      //   data = this.plotDataAll["dim_dispersion_detection"]["data"]
      //   // num2rule=this.num2dispersion
      // }
      // if (this.radio=="Dim Deviation Detection"){
      //   data = this.plotDataAll["dim_deviation_detection"]["data"]
      //   // num2rule=this.num2deviation
      // }

      // let result=data.filter((item) => {
      //   return (item.cnc_no==this.machineInput && item.fai_no==this.faiNoInput);
      //   });
      // if (this.machineInput != '' && this.faiNoInput != ''){
      //   if (result.length != 0){
      //     // console.log(result)
      //     this.riskLevel=this.checkList[parseFloat(result[0]['risk2num']).toFixed(1).toString()]
      //     // console.log(this.machineInput)
      //     // console.log(this.faiNoInput)
      //     // console.log(this.riskLevel)
      //   }
      // }else{
      //   this.riskLevel='None'
      // }
      for (let record of this.userCheckForm){
        // console.log(record)
        let result=data.filter((item) => {
          return (item.cnc_no==record.machine_no && item.fai_no==record.fai_no);
        });
        if (record.machine_no != '' && record.fai_no != ''){
          if (result.length != 0){
            // console.log(result)
            record.risk_level=this.checkList[parseFloat(result[0]['risk2num']).toFixed(1).toString()]
            // console.log(this.machineInput)
            // console.log(this.faiNoInput)
            // console.log(this.riskLevel)
          }
        }else{
          record.risk_level='None'
        }
      }

    },
    getCheckList(ruleName) {
      // console.log(ruleName)
      if (ruleName=="Dim Risk Level"){
        this.checkList=this.num2riskLevel
      }
      if (ruleName=="Dim Outlier Detection"){
        this.checkList=this.num2outlier
      }
      if (ruleName=="Dim Dispersion Detection"){
        this.checkList=this.num2dispersion
      }
      if (ruleName=="Dim Deviation Detection"){
        this.checkList=this.num2deviation
      }
    },
    closeDialog(){
      this.machineInput=''
      this.faiNoInput=''
      this.checkInput=''
    },
    updateData(param) {
      let option = this.$refs.option;
      this.cellQual.projName = option.project;
      this.cellQual.vendorName = option.vendor;
      this.updateDate = option.selectingOption[0].dim_updated_at;
      this.showVendorProjectTitle = true;
      this.headerId=option.selectingOption[0].dim_header_id
      this.dataRound=option.selectingOption[0].data_round
      console.log(this.headerId)
      //目前只筛选dimNo
      // let dimNo = option.searchMoreConditions.dimNo;
      // if (dimNo != 0) {
      //   this.plotDataAll = option.totalData.filter(function(item) {
      //     return !(dimNo.indexOf(item.dim_no) == -1);
      //   });
      // } else {
      //   this.plotDataAll = option.totalData;
      // }
      this.plotDataAll = option.totalData;
      dvApi
        .getHotMapChkresult(this.headerId)
        .then(res => {
          this.plotChkData=res
          // console.log(res.msg)
        })
        .catch(err => {
          //暂时用alert
          alert(err);
        });
      
      if (this.plotDataAll.hasOwnProperty("msg")){
          alert(this.plotDataAll.msg)
      }
      
    },
    getUserCheckTableData(){
      this.userCheckTableData=[]
      let originData
      let checkData
      originData=this.plotDataAll[this.label2variable[this.radio]]
      checkData=this.plotChkData[this.label2variable[this.radio]]
      // if (this.radio=="Dim Risk Level"){
      //   originData=this.plotDataAll['dim_risk_level']
      //   checkData=this.plotChkData['dim_risk_level']
      // }
      // if (this.radio=="Dim Outlier Detection"){
      //   originData=this.plotDataAll['dim_outlier_detection']
      //   checkData=this.plotChkData['dim_outlier_detection']
      // }
      // if (this.radio=="Dim Dispersion Detection"){
      //   originData=this.plotDataAll['dim_dispersion_detection']
      //   checkData=this.plotChkData['dim_dispersion_detection']
      // }
      // if (this.radio=="Dim Deviation Detection"){
      //   originData=this.plotDataAll['dim_deviation_detection']
      //   checkData=this.plotChkData['dim_deviation_detection']
      // }
      let originDataRecord
      let checkDataRecord
      // console.log(originDataRecord)
      for(let cncNo of originData['cnc_no']){
        for(let faiNo of originData['fai_no_list']){
          originDataRecord=originData['data'].filter((item) => {
            return (item.cnc_no==cncNo && item.fai_no==faiNo);
          })
          checkDataRecord=checkData['data'].filter((item) => {
            return (item.cnc_no==cncNo && item.fai_no==faiNo);
          })
            // console.log(originDataRecord[0].calc_result)
            // console.log(checkDataRecord[0].chk_result)
          if (originDataRecord[0].calc_result != checkDataRecord[0].chk_result){
            // console.log(originDataRecord[0].calc_result)
            // console.log(checkDataRecord[0].chk_result)
            // console.log(1)
            this.userCheckTableData.push({
              machine:cncNo,
              faiNo:faiNo,
              riskLevel:originDataRecord[0].calc_result,
              check:checkDataRecord[0].chk_result
              })
          }
        }
      }
      // console.log(this.userCheckTableData)
    },
    clickUserCheckButton() {
      this.dialogVisible=true
    },
    addOneFormRow(){
      this.userCheckForm.push(
        {
          "header_id":this.headerId,
          "data_round":this.dataRound,
          "machine_no":"",
          "fai_no":"",
          "rule_name":this.label2variable[this.radio],
          "risk_level":"None",
          "chk_result":""
        }
      )
    },
    minusOneFormRow(){
      this.userCheckForm.pop()
    },
    formatPlotData() {},

    initPlot(ruleName) {
      var riskLevelColorRule=[
        {value: "0.0", label: this.num2riskLevel["0.0"], color:'rgba(0,145,0,0.8)'}, //深绿
        {value: "1.0", label: this.num2riskLevel["1.0"], color:'rgba(0,236,0,0.8)'}, //浅绿
        {value: "2.0", label: this.num2riskLevel["2.0"], color:'rgba(255,88,9,0.8)'}, //橙色
        {value: "-2.0", label: this.num2riskLevel["-2.0"], color:'rgba(255,88,9,0.8)'}, //橙色
        {value: "2.5", label: this.num2riskLevel["2.5"], color:'rgba(255,88,9,0.8)'}, //橙色
        {value: "-2.5", label: this.num2riskLevel["-2.5"], color:'rgba(255,88,9,0.8)'}, //橙色
        {value: "3.0", label: this.num2riskLevel["3.0"], color:'rgba(255,117,117,0.8)'}, //浅红
        {value: "-3.0", label: this.num2riskLevel["-3.0"], color:'rgba(255,117,117,0.8)'}, //浅红
        {value: "4.0", label: this.num2riskLevel["4.0"], color:'rgba(174,0,0,0.8)'}, //深红
        {value: "-4.0", label: this.num2riskLevel["-4.0"], color:'rgba(174,0,0,0.8)'}, //深红
      ]
      var outlierColorRule=[
        {value: "0.0", label: this.num2outlier["0.0"], color:'rgba(235,235,235,0.8)'},
        {value: "1.0", label: this.num2outlier["1.0"], color:'rgba(0,145,0,0.8)'},//深绿
        {value: "2.0", label: this.num2outlier["2.0"], color:'rgba(0,145,0,0.8)'},
        {value: "3.0", label: this.num2outlier["3.0"], color:'rgba(255,88,9,0.8)'},//橙色
        {value: "4.0", label: this.num2outlier["4.0"], color:'rgba(255,88,9,0.8)'},//橙色
        {value: "5.0", label: this.num2outlier["5.0"], color:'rgba(174,0,0,0.8)'},
        {value: "6.0", label: this.num2outlier["6.0"], color:'rgba(174,0,0,0.8)'},//深红
      ]
      var dispersionColorRule=[
        {value: "0.0", label: this.num2dispersion["0.0"], color:'rgba(235,235,235,0.8)'},
        {value: "1.0", label: this.num2dispersion["1.0"], color:'rgba(0,145,0,0.8)'},//深绿
        {value: "2.0", label: this.num2dispersion["2.0"], color:'rgba(0,145,0,0.8)'},
        {value: "3.0", label: this.num2dispersion["3.0"], color:'rgba(174,0,0,0.8)'},
        {value: "4.0", label: this.num2dispersion["4.0"], color:'rgba(174,0,0,0.8)'},//深红
      ]
      var deviationColorRule=[
        {value: "0.0", label: this.num2deviation["0.0"], color:'rgba(235,235,235,0.8)'},
        {value: "1.0", label: this.num2deviation["1.0"], color:'rgba(0,145,0,0.8)'},//深绿
        {value: "2.0", label: this.num2deviation["2.0"], color:'rgba(0,145,0,0.8)'},
        {value: "3.0", label: this.num2deviation["3.0"], color:'rgba(174,0,0,0.8)'},
        {value: "4.0", label: this.num2deviation["4.0"], color:'rgba(174,0,0,0.8)'},//深红
      ]
      let chartDv = this.$refs.chartDv;
      let echart = echarts.init(chartDv);
      
      // let data=this.plotDataAll
      let data
      // console.log(ruleName)
      // console.log(this.plotDataAll)
      // var data=testData[ruleName]
      
      let num2rule
      let rulePieces
      if (this.showOriginOrUpdate=='Origin') {
        if (ruleName=="Dim Risk Level"){
          num2rule=this.num2riskLevel
          rulePieces=riskLevelColorRule
          // data=this.plotDataAll['dim_risk_level']
        }
        if (ruleName=="Dim Outlier Detection"){
          num2rule=this.num2outlier
          rulePieces=outlierColorRule
          // data=this.plotDataAll['dim_outlier_detection']
        }
        if (ruleName=="Dim Dispersion Detection"){
          num2rule=this.num2dispersion
          rulePieces=dispersionColorRule
          // data=this.plotDataAll['dim_dispersion_detection']
        }
        if (ruleName=="Dim Deviation Detection"){
          num2rule=this.num2deviation
          rulePieces=deviationColorRule
          // data=this.plotDataAll['dim_deviation_detection']
        }
        data=this.plotDataAll[this.label2variable[ruleName]]
      }else{
        if (ruleName=="Dim Risk Level"){
          num2rule=this.num2riskLevel
          rulePieces=riskLevelColorRule
          // data=this.plotChkData['dim_risk_level']
        }
        if (ruleName=="Dim Outlier Detection"){
          num2rule=this.num2outlier
          rulePieces=outlierColorRule
          // data=this.plotChkData['dim_outlier_detection']
        }
        if (ruleName=="Dim Dispersion Detection"){
          num2rule=this.num2dispersion
          rulePieces=dispersionColorRule
          // data=this.plotChkData['dim_dispersion_detection']
        }
        if (ruleName=="Dim Deviation Detection"){
          num2rule=this.num2deviation
          rulePieces=deviationColorRule
          // data=this.plotChkData['dim_deviation_detection']
        }
        data=this.plotChkData[this.label2variable[ruleName]]
      }
      // console.log(data)
      let xlabel = data.cnc_no
      let ylabel = data.fai_no_list
      let arr=[]

      for (var item of data.data){
        arr.push(
          // [item.cnc_no, item.fai_no.toString(), item.projected_yields, item.risk2num]
            [item.cnc_no, item.fai_no.toString(), item.risk2num]
          );
      }
      console.log(this.showOriginOrUpdate)
      
      // console.log(xlabel)
      // console.log(ylabel)
      let option = {
        tooltip: {
            show:true,
            position: 'top',
            formatter:function(params){
              return [ 
                'CNC: ' + params.name + '&nbsp&nbsp&nbsp&nbspFAI NO: ' + params.data[1],
                'Risk Level :' + num2rule[params.data[2].toFixed(1).toString()],
                // 'Projected Yields: ' + params.data[2]
              ].join("<br/>");
            },
        },
        animation: false,
        grid: {
            height: '90%',
            width: '10%',
            top: '2%',
            left: '20%'
        },
        xAxis: {
            type: 'category',
            data: xlabel,
            splitArea: {
                show: true
            }
        },
        yAxis: {
            type: 'category',
            data: ylabel,
            splitArea: {
                show: true
            }
        },
        visualMap: {
            type: 'piecewise',
            splitNumber: 10,
            pieces:rulePieces,
            calculable: false,
            orient: 'vertical',
            left: '0%',
            bottom: '50%'
        },
        series: [{
            name: 'Risk Level',
            type: 'heatmap',
            data: arr,
            label: {
                normal:{
                    show: true,
                    formatter:function(params){
                      let temp=num2rule[params.data[2].toFixed(1).toString()]
                      // console.log(temp.charAt(temp.length-1))
                      if (ruleName!='Dim Risk Level' && temp.charAt(temp.length-1) == '*'){
                        return '*';
                      }else {
                        return '';
                      }
                    },
                    fontSize: 16,
                    fontFamily:'宋体',
                },
                emphasis:{
                    show: false,
                }
            },
            emphasis: {
                itemStyle: {
                    shadowBlur: 10,
                    shadowColor: 'rgba(0, 0, 0, 0.5)'
                }
            }
          }]
      
      };
      echart.setOption(option);
    },
  },
  watch: {
    plotDataAll: function(newValue, oldValue) {
      this.initPlot('Dim Risk Level');//值的改变触发initPlot函数
      this.show = true
    },
    plotChkData:function(newValue, oldValue) {
      this.getUserCheckTableData()
    },
    radio: function(newValue, oldValue) {
      // console.log(newValue)

      this.initPlot(newValue)
      this.show = true
      this.title=newValue+ ' Checked by User'
      console.log(this.title)
      this.getUserCheckTableData()
      this.getCheckList(newValue)
    },
    // machineInput:function(newValue, oldValue) {
    //   this.searchRiskLevel()
    // },
    // faiNoInput:function(newValue, oldValue) {
    //   this.searchRiskLevel()
    // },

    // userCheckForm:{function(newValue, oldValue) {
    //   console.log('change')
    //   this.searchRiskLevel()
    // },
    userCheckForm:{
      handler: function (newVal) {
            console.log(newVal)   
            this.searchRiskLevel()
          },      
      deep: true    //深度监听
    },
    
    showOriginOrUpdate:function(newValue, oldValue) {
      // this.searchRiskLevel()
      this.initPlot(this.radio)
    },
  }
};
</script>
<style scoped>
  .ruleName {
  text-align: center;
}
</style>