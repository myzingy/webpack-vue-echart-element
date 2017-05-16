<template>
  <main id="app">
  <el-form ref="form" :model="form" label-width="80px">
  <el-col :span="10">
    <el-form-item label="用户组">
      <el-select v-model="form.group_selected" placeholder="请选择用户组">
        <el-option
          key=""
          label="全部组"
          value="">
        </el-option>
        <el-option
          v-for="item in form.group"
          :key="item.Id"
          :label="item.Id"
          :value="item.Id">
        </el-option>
      </el-select>
    </el-form-item>
  </el-col>
  <el-col :span="10">
    <el-form-item label="日期周期">
        <el-date-picker
          v-model="form.date"
          type="daterange"
          align="right"
          placeholder="选择日期范围"
          :picker-options="date_choice">
        </el-date-picker>
    </el-form-item>
  </el-col>
  <el-col :span="4">
    <el-form-item>
      <el-button type="primary" @click="load">统计</el-button>
    </el-form-item>
  </el-col>
</el-form>
<figure><chart :options="ticket_day_response" ref="ticket_day_response" theme="ovilia-green" auto-resize></chart></figure>
<figure><chart :options="ticket_closed" ref="ticket_closed" theme="ovilia-green" auto-resize></chart></figure>
<figure><chart :options="ticket_response_time" ref="ticket_response_time" theme="ovilia-green" auto-resize></chart></figure>
<figure><chart :options="ticket24hour" ref="ticket24hour" theme="ovilia-green" auto-resize></chart></figure>

<figure><chart :options="chat_time_line" ref="chat_time_line" theme="ovilia-green" auto-resize></chart></figure>
<figure><chart :options="chat_time_response" ref="chat_time_response" theme="ovilia-green" auto-resize></chart></figure>
<figure><chart :options="score" ref="score" theme="ovilia-green" auto-resize></chart></figure>
  </main>
</template>

<style lang="stylus">
@import "./assets/style/main.scss";
</style>

<script>
import URI from './uri'
import Vue from 'vue'
import ECharts from 'vue-echarts/components/ECharts.vue'
import 'echarts/lib/chart/bar'
import 'echarts/lib/chart/line'
//import 'echarts/lib/chart/pie'
//import 'echarts/lib/chart/map'
import 'echarts/lib/chart/radar'
//import 'echarts/lib/chart/scatter'
//import 'echarts/lib/chart/effectScatter'
import 'echarts/lib/component/tooltip'
//import 'echarts/lib/component/polar'
//import 'echarts/lib/component/geo'
import 'echarts/lib/component/legend'
import 'echarts/lib/component/title'
//import 'echarts/lib/component/visualMap'

import date_choice from './date_choice'
import ticket_day_response from './data/ticket.day.response'
import ticket_closed from './data/ticket.closed'
import ticket_response_time from './data/ticket.response.time'
import ticket24hour from './data/ticket.24hour'

import score from './data/score'
//import chat_time from './data/chat_time'
import chat_time_line from './data/chat_time_line'
import chat_time_response from './data/chat_time_response'

// built-in theme
import 'echarts/theme/dark'

// custom theme
import theme from './theme.json'

// Map of China
//import chinaMap from './china.json'

// registering map data
//ECharts.registerMap('china', chinaMap)

// registering custom theme
ECharts.registerTheme('ovilia-green', theme)

import ElementUI from 'element-ui'
import 'element-ui/lib/theme-default/index.css'
Vue.use(ElementUI)
import vk from './vk'
export default {
  components: {
    chart: ECharts
  },
  //store,
  data () {
    return {
      date_choice,

      ticket_day_response,
      ticket_closed,
      ticket_response_time,
      ticket24hour,
      chat_time_line,
      chat_time_response,
      score,
      
      seconds: -1,
      asyncCount: false,
      connected: false,
      metricIndex: 0,
      form: {
        group: [],
        group_selected:'',
        date: ["",""]
      }
    }
  },
  computed: {
    // scoreRadar () {
    //   return this.$store.getters.scoreRadar
    // },
    // metrics () {
    //   return this.$store.state.scores.map(({name}) => name)
    // },
    // isMax () {
    //   let {value, max} = this.$store.state.scores[this.metricIndex]
    //   return value === max
    // },
    // isMin () {
    //   return this.$store.state.scores[this.metricIndex].value === 0
    // }
  },
  methods: {
    load () {
      console.log(this.form.group_selected,this.form.date);
      vk.http(URI.GET_GROUPS,{
        stime:(this.form.date[0]?this.form.date[0].getTime():"").toString().substr(0,10),
        etime:(this.form.date[1]?this.form.date[1].getTime():"").toString().substr(0,10),
        group_id:this.form.group_selected
      },this.then);
      // simulating async data from server
      // this.seconds = 3
      // let bar = this.$refs.ticket_day_response
      // bar.showLoading({
      //   text: '正在加载',
      //   color: '#4ea397',
      //   maskColor: 'rgba(255, 255, 255, 0.4)'
      // })
      // let timer = setInterval(() => {
      //   this.seconds--
      //   if (this.seconds === 0) {
      //     clearTimeout(timer)
      //     bar.hideLoading()
      //     bar.mergeOptions(barAsync)
      //   }
      // }, 0)
    },
    convert () {
      let map = this.$refs.map
      let src = map.getDataURL({
        pixelRatio: window.devicePixelRatio || 1
      })
      window.open(`data:text/html,<img src="${src}" width="${map.width}" height="${map.height}">`)
    },
    increase (amount) {
      // if (!this.asyncCount) {
      //   this.$store.commit('increment', {amount, index: this.metricIndex})
      // } else {
      //   this.$store.dispatch('asyncIncrement', {amount, index: this.metricIndex, delay: 500})
      // }
    },
    then(data,code){
      console.log("demo...res",data);
      this.init__score(data);
      this.init__ticket_day_response(data);
      this.init__ticket_response_time(data);
      this.init__ticket_24hour(data);
      this.init__chat_time_line(data);
      this.init__chat_time_response(data);
      this.init__ticket_closed(data);
      // var form=this.form;
      // form.group=data.groups;
      // this.$set('form', form);
      //Vue.set(this,'form.group',data.groups)
    },
    init__score:function(data){
      this.form.group=data.groups;
      var score_data=[];
      for(var i=0;i<data.score.length;i++){
        var operator_id=data.score[i].operator_id;
        console.log('operator_id',operator_id,data.operators[operator_id]);
        var name=data.operators[operator_id]?data.operators[operator_id].Fullname:"N/A";
        score_data.push({
          name:name,
          value:[parseFloat(data.score[i].ca.substr(0,3)),parseFloat(data.score[i].cb.substr(0,3)),parseFloat(data.score[i].cc.substr(0,3)),parseFloat(data.score[i].cd.substr(0,3))]
        });
      }
      score.init(score_data);
    },
    init__ticket_day_response:function(data){
      var ticket_day={xAxis:[],legend:[],series:[]};
      for(var day in data.ticket_day_response.day){
        ticket_day.xAxis.push(day);
      }
      for(var user_id in data.ticket_day_response.val){
        var val=data.ticket_day_response.val[user_id];
        var name=data.operators[user_id]?data.operators[user_id].Fullname:"N/A";
        ticket_day.legend.push(name);
        var one={
            name:name,
            type: 'line',
            data:[],
        };
        for(var day in data.ticket_day_response.day){
          one.data.push(val[day]>0?val[day]:0);
        }
        ticket_day.series.push(one);
      };
      ticket_day_response.init(ticket_day);
    },
    init__ticket_response_time:function(data){
      var ticket_time={xAxis:[],min:[],avg:[],max:[]};
      
      for(var user_id in data.ticket_response_time){
        var name=data.operators[user_id]?data.operators[user_id].Fullname:"N/A";
        ticket_time.xAxis.push(name);
        var val=data.ticket_response_time[user_id];
        ticket_time.min.push((val.min/3600).toFixed(2));
        ticket_time.max.push((val.max/3600).toFixed(2));
        ticket_time.avg.push((val.avg/3600).toFixed(2));
      };
      ticket_response_time.init(ticket_time);
    },
    init__ticket_24hour:function(data){
      var ticket_24hour={legend:[],series:[]};
      
      for(var user_id in data.ticket24hour){
        var name=data.operators[user_id]?data.operators[user_id].Fullname:"N/A";
        ticket_24hour.legend.push(name);
        var t24=data.ticket24hour[user_id];
        var val=t24.ticket_ids.length>0?((t24.ticket_ids_response.length/t24.ticket_ids.length)*100).toFixed(2):0;
        ticket_24hour.series.push({
            name: name,
            type: 'gauge',
            detail: {formatter:'\n'},
            data: [{value: val}]
        });
      };
      ticket24hour.init(ticket_24hour);
    },
    init__chat_time_line:function(data){
      var ticket_day={xAxis:[],legend:[],series:[]};
      for(var day in data.chat_time_line.day){
        ticket_day.xAxis.push(day);
      }
      for(var user_id in data.chat_time_line.val){
        var val=data.chat_time_line.val[user_id];
        var name=data.operators[user_id]?data.operators[user_id].Fullname:"N/A";
        ticket_day.legend.push(name);
        var one={
            name:name,
            type: 'line',
            data:[],
        };
        for(var day in data.chat_time_line.day){
          one.data.push(parseFloat(val[day]>0?val[day]:0).toFixed(2));
        }
        ticket_day.series.push(one);
      };
      chat_time_line.init(ticket_day);
    },
    init__chat_time_response:function(data){
      var ticket_day={xAxis:[],legend:[],series:[]};
      for(var day in data.chat_time_response.day){
        ticket_day.xAxis.push(day);
      }
      for(var user_id in data.chat_time_response.val){
        var val=data.chat_time_response.val[user_id];
        var name=data.operators[user_id]?data.operators[user_id].Fullname:"N/A";
        ticket_day.legend.push(name);
        var one={
            name:name,
            type: 'line',
            data:[],
        };
        for(var day in data.chat_time_response.day){
          one.data.push(val[day]>0?val[day]:0);
        }
        ticket_day.series.push(one);
      };
      chat_time_response.init(ticket_day);
    },
    init__ticket_closed:function(data){
      var _closed={legend:[],series:[]};
      for(var user_id in data.ticket_closed){
        var name=data.operators[user_id]?data.operators[user_id].Fullname:"N/A";
        _closed.legend.push(name);
        var r=data.ticket_closed[user_id];
        var val=[0,0,0,0];
        if(r.amount>0){
          r.closed_amount=parseInt(r.closed_amount);
          r.amount=parseInt(r.amount);
          val=[
            ((r.closed_3days/r.amount)*100).toFixed(2),
            ((r.closed_7days/r.amount)*100).toFixed(2),
            ((r.closed_15days/r.amount)*100).toFixed(2),
            (((r.closed_amount>r.amount?r.amount:r.closed_amount)/r.amount)*100).toFixed(2)
          ];
        }
        _closed.series.push({
            name: name,
            value: val
        });
      };
      ticket_closed.init(_closed);
    },
  },
  watch: {
    connected: {
      handler (value) {
        ECharts[value ? 'connect' : 'disconnect']('radiance')
      }
    }
  },
  mounted(){
    score.init();
    this.load();
  }
}
</script>
