<template>
    <div class="income">
    <div class="income_top">
        <mt-header fixed title="营业数据" class="income_header">
            <router-link to="/home" slot="left">
                <mt-button icon="back"></mt-button>
            </router-link>
            <mt-button @click="turnmore(1)" slot="right" class="income_header_date">{{actday}}</mt-button>
        </mt-header>
        <div class="income_banner">
            <div class="income_banner_data">
                <p class="income_data_num">￥{{totalPrice}}</p>
                <p>营业额</p>
            </div>
            <div class="income_banner_data" @click="bill">
                <p class="income_data_num">{{total}}</p>
                <p>核销券数</p>
            </div>
        </div>
        <div class="income_operate">
            <div class="income_operate_result">
                <p>{{start}} -- {{end}}</p>
                <p>
                    <span>营业额：￥{{totalPrice}}元</span>
                    <span>订单数：{{orderNum}}张</span>
                </p>
            </div>
            <div class="income_operate_time" @click="turnmore(2)">
                <img src="../../../../static/images/calendar.png" alt="">
            </div>
        </div>
    </div>
    <div class="mobox" @click="closemore" v-show="ismore">
        <div class="triangle" v-show="isselectday"></div>
        <ul class="moreday" v-show="isselectday">
            <li class="adays" v-for="(item,index) in days" :key='index' :id='item.title' @click="selectday">{{item.title}}</li>
        </ul>
        <div class="select" v-show="isselecttime">
            <div class="select_top">
                <span class="stleft">选择起始时间</span>
            </div>
            <div class="date">
                <div @click.stop="openPicker(1)">开始时间：<span>{{temstart}}</span></div>
                <div @click.stop="openPicker(2)">结束时间：<span>{{temend}}</span></div>
            </div>
            <div class="selbut">
                <div class="close" @click="closemore">取消</div>
                <div class="cfrm" @click="cfrm">确定</div>
            </div>
        </div>
    </div>
    <div class="filling" id="filling"></div>
    <div class="loadBottom inbox" :style="{'-webkit-overflow-scrolling': scrollMode,height:oDvheight}">
        <div class="income_list" :style="styleObject">
          <ul id="income" class="income" @touchstart="touchStart" @touchmove="touchMove" @touchend="touchEnd">
              <li class="income_li" v-for="(item,index) in votes" :key='index' :id='item.id' @click="getliid">
                  <div class="income_li_l">
                      <div class="income_li_l_date">{{item.createTime}}</div>
                      <div class="income_li_l_money">￥{{item.soAmount}}</div>
                  </div>
                  <div class="income_li_r">
                      <div class="left">
                          <p>付款单号</p>
                          <p>付款人</p>
                          <p v-if="item.skuName">代金券</p>
                      </div>
                      <div class="right">
                        <p>{{item.id}}</p>
                        <p>{{item.userName}}</p>
                        <p v-if="item.skuName">{{item.skuName}}</p>
                      </div>
                  </div>
              </li>
              <li class="loadingBox" v-if="loadFlag">加载中..</li>
          </ul>
        </div>
  
    </div>

    <mt-datetime-picker
            ref="picker"
            type="date"
            v-if="mindata"
            year-format="{value} 年"
            month-format="{value} 月"
            date-format="{value} 日"
            v-model="pickerValue"
            :startDate = 'mindata'
            :endDate = 'maxdata'
            @confirm="handleConfirm"
        >
    </mt-datetime-picker>
    </div>
</template>

<script>
import Vue from "vue";
import { DatetimePicker, Toast, Loadmore, Indicator } from "mint-ui";
Vue.component(DatetimePicker.name, DatetimePicker);
Vue.component(Loadmore.name, Loadmore);
import store from "@/vuex/store";
import { mapState, mapMutations } from "vuex";
export default {
  name: "Income",
  data() {
    return {
      start: "",
      end: "",
      temstart: "",
      temend: "",
      actval: "",
      total: "",
      orderNum: "0",
      pickerValue: "",
      Visible: true,
      ismore: false,
      actday: "",
      isselectday: false,
      isselecttime: false,
      maxdata: "",
      mindata: "",
      styleObject: {
          height:'0px'
      },
      days: [
        {
          title: "今日"
        },
        {
          title: "7日"
        },
        {
          title: "15日"
        }
      ],
      votes: [],
      totalPrice: 0,
      page: 1,
      allLoaded: true,
      scrollMode: "auto",
      oDvheight:'0px',
      touchStartY: 0,
      distance: 0,
      topFlag: false, //是否到顶部
      bottomFlag: false, //是否到底部
      flag: true, //节流阀
      loadFlag: false,
      today:''
    };
  },
  created: function() {
    ispush = false;
    let _this = this;
    this.getWindowHeight(1);
    const ua = navigator.userAgent.toLowerCase();
    if (/(iPhone|iPad|iPod|iOS)/i.test(ua)) {
      this.scrollMode = "touch";
    }

    this.$axios.get("/api/app/act/getDate").then(res => {
      res.data.data =res.data.data.replace(/(-)/g, '/');
      this.today =res.data.data;
      let _date =this.today;
      let _mindata =
      new Date(_this.$UTILS.dateConv(new Date(_date))).getTime() - 86400000 * 365;
      this.mindata = new Date(_mindata);
      let _maxdata =
      new Date(_this.$UTILS.dateConv(new Date(_date))).getTime() + 86400000 * 365 * 3;
      this.maxdata = new Date(_maxdata);

      let _start = new Date(_this.$UTILS.dateConv(new Date(_date))).getTime();
      _start = _this.$UTILS.dateConv(new Date(_start));
      let _end = new Date(_this.$UTILS.dateConv(new Date(_date))).getTime() + 86400000;
      _end = _this.$UTILS.dateConv(new Date(_end));

      if (this.$route.query.start) {
        _start = this.$route.query.start;
        _end = this.$route.query.end;
        this.actday = this.$route.query.actday;
      }

      this.start = _start;
      this.end = _end;
      this.temstart = _start;
      this.temend = _end;
      // this.oldstart = _start;
      // this.oldend = _end;
      this.getAmount();
      this.getTicketList();
      this.getdata();
      if(!this.actday){
        this.actday = this.days[0].title;
      }
    });
  },
  components: {
    newtime: function() {
      return (newtime = this.start + this.end);
    }
  },
  watch: {
    newtime: function() {
      this.page = 1;
      this.getdata();
      this.getAmount();
      this.getTicketList();
    }
  },
  store,
  computed: {
    ...mapState(["shopId"])
  },
  methods: {
    init: function(val) {
      
    },
    openPicker(val) {
      this.actval = val;
      this.$refs.picker.open();
    },
    handleConfirm() {
      this.allLoaded = true;
      let _this = this,
        date = _this.$UTILS.dateConv(this.pickerValue);
      if (this.actval == 1) {
        this.temstart = date;
      } else if (this.actval == 2) {
        this.temend = date;
      }
    },
    setdate: function(val) {
      if (val) {
        let arr = val.split("/");
        return arr[0] + "年" + arr[1] + "月" + arr[2] + "日";
      }
    },
    // 时间判断
    closemore:function(){
      this.temend = this.end;
      this.temstart = this.start;
    },
    cfrm: function() {
      if (this.temstart && this.temend) {
        let _start = new Date(this.temstart);
        let _end = new Date(this.temend);
        this.page = 1;
        _start = _start.getTime();
        _end = _end.getTime();
        // this.oldstart = this.start;
        // this.oldend =this.end;
        if (_end > _start) {
          this.end = this.temend;
          this.start = this.temstart;
          this.getAmount(this.start, this.end);
          this.getTicketList(this.start, this.end);
          this.getdata(this.start, this.end);
        } else {
          Toast("结束时间不能小于开始时间");
          this.temend = this.end;
          this.temstart = this.start;
        }
      } else if (!this.temstart || !this.temend) {
        Toast("请选择开始时间或结束时间");
      }
    },
    turnmore: function(val) {
      this.ismore = true;
      if (val == 1) {
        this.isselectday = true;
      } else if (val == 2) {
        this.isselecttime = true;
        this.pickerVisible = true;
      }
    },
    closemore: function() {
      this.ismore = false;
      this.isselectday = false;
      this.isselecttime = false;
    },
    // 跳转至详情
    getliid: function(e) {
      return false;
      let id = e.currentTarget.id;

      this.$router.push({ name: "Writeoff", params: { id: id, type: 1 } });
    },
    // 选择日期
    selectday: function(e) {
      this.allLoaded = true;
      let _this = this,_date = this.today;;
      this.actday = e.currentTarget.id;
      let _start = "",
        _deff = 60 * 60 * 24 * 1000;
      let _end = new Date(_this.$UTILS.dateConv(new Date(_date))).getTime() + _deff * 1;
      _end = _this.$UTILS.dateConv(new Date(_end));
      this.end = _end;
      this.page = 1;
      // this.oldend = _end;
      this.votes = [];
      if (this.actday == "今日") {
        _start = new Date(_this.$UTILS.dateConv(new Date(_date))).getTime();
        _start = _this.$UTILS.dateConv(new Date(_start));
        this.start = _start;
        // this.oldstart = _start;
        this.getAmount();
        this.getTicketList();
        this.getdata();
      } else if (this.actday == "7日") {
        _start = new Date(_this.$UTILS.dateConv(new Date(_date))).getTime() - _deff * 7;
        _start = _this.$UTILS.dateConv(new Date(_start));
        this.start = _start;
        // this.oldstart = _start;
        this.getAmount();
        this.getTicketList();
        this.getdata();
      } else if (this.actday == "15日") {
        _start = new Date(_this.$UTILS.dateConv(new Date(_date))).getTime() - _deff * 15;
        _start = _this.$UTILS.dateConv(new Date(_start));
        this.start = _start;
        // this.oldstart = _start;
        this.getAmount();
        this.getTicketList();
        this.getdata();
      }
    },
    // 获取营业总额
    getAmount(start, end) {
      let obj = {
          shopId: this.shopId,
          beginTime: start ? start : this.start,
          endTime: end ? end : this.end
        },
        _value = "";
      for (var key in obj) {
        _value += key + "=" + obj[key] + "&";
      }
      this.$axios.get("/api/app/so/totalAmount?" + _value).then(res => {
        if (res.data.code == 0 && res.data.data) {
          this.totalPrice = res.data.data;
        } else {
          this.totalPrice = 0;
        }
      });
    },
    // 获取核销券数
    getTicketList(start, end) {
      let obj = {
          shopId: this.shopId,
          begainTime: start ? start : this.start,
          endTime: end ? end : this.end
        },
        _value = "";
      for (var key in obj) {
        _value += key + "=" + obj[key] + "&";
      }
      this.$axios.get("/api/app/hx/list?" + _value).then(res => {
        this.total = res.data.data.total;
      });
    },
    //获取列表数据
    getdata: function(start, end, val, type) {
      // Indicator.open("数据加载中...");
      let obj = {
        shopId: this.shopId,
        soStatus: 2,
        beginTime: start ? start : this.start,
        endTime: end ? end : this.end,
        page: val ? val : this.page,
        rows: 10
      };
      // this.oldstart == start;
      // this.oldend == end;
      let _value = "";
      for (var key in obj) {
        _value += key + "=" + obj[key] + "&";
      }
      _value = _value.substring(0, _value.length - 1);
      this.$axios.get("/api/app/so/myorderForShop?" + _value).then(res => {
        // Indicator.close();
        if (res.data.code == 0) {
          this.loadFlag = false;
          let _data = res.data.data;
          this.orderNum = _data.total ? _data.total : 0;
          if (_data.list) {
            if (this.page == 1) {
              this.votes = [];
            }
            if (_data.list.length > 0) {
              for (let j = 0; j < _data.list.length; j++) {
                if (/^1[34578]\d{9}$/.test(_data.list[j].userName)) {
                  _data.list[j].userName =
                    _data.list[j].userName.substring(0, 3) +
                    "******" +
                    _data.list[j].userName.substring(9, 11);
                }
                this.votes.push(_data.list[j]);
              }
              if (_data.list.length < 10) {
                this.allLoaded = false;
              }
            } else {
              this.allLoaded = false;
            }
          } else {
            this.allLoaded = false;
          }
        }
      });
    },
    getScrollTop() {
      //获取顶部卷去高度
      var scrollTop = 0,
        bodyScrollTop = 0,
        documentScrollTop = 0;
      if (document.body) {
        bodyScrollTop = document.body.scrollTop;
      }
      if (document.documentElement) {
        documentScrollTop = document.documentElement.scrollTop;
      }
      scrollTop =
        bodyScrollTop - documentScrollTop > 0
          ? bodyScrollTop
          : documentScrollTop;
      return scrollTop;
    },
    getScrollHeight() {
      //盒子总高度
      var scrollHeight = 0,
        bodyScrollHeight = 0,
        documentScrollHeight = 0;
      if (document.body) {
        bodyScrollHeight = document.body.scrollHeight;
      }
      if (document.documentElement) {
        documentScrollHeight = document.documentElement.scrollHeight;
      }
      scrollHeight =
        bodyScrollHeight - documentScrollHeight > 0
          ? bodyScrollHeight
          : documentScrollHeight;
      return scrollHeight;
    },
    //屏幕可视高度
    getWindowHeight(val) {
      var windowHeight = 0;
      if (document.compatMode == "CSS1Compat") {
        windowHeight = document.documentElement.clientHeight;
      } else {
        windowHeight = document.body.clientHeight;
      }
      if(val == 1){
         this.oDvheight=windowHeight*1-231+'px';
      }else{
         return windowHeight;
      } 
    },
    touchStart(e) {
      let dishesUl = document.getElementById("income");
      let bottomH = document.getElementById("filling").clientHeight;
      this.touchStartY = e.targetTouches[0].pageY;
      if (this.getScrollTop() == 0 && this.flag) {
        this.topFlag = true;
      } else {
        this.topFlag = false;
      }
  
      // if (dishesUl.clientHeight < this.getWindowHeight() - bottomH - 5) {
      //   this.allLoaded = false;
      //   this.bottomFlag = false;
      // }

      if (
        Math.abs(
          this.getScrollHeight() - this.getScrollTop() - this.getWindowHeight()
        ) < 5 &&
        this.allLoaded
      ) {
        this.bottomFlag = true;
      } else {
        this.bottomFlag = false;
      }
    },
    touchMove(e) {
      let dishesUl = document.getElementById("income");
      this.distance = Math.ceil(+e.targetTouches[0].pageY - this.touchStartY);
      if (this.distance > 0 && this.topFlag == true && this.flag) {
        if (this.distance > 100) {
          this.distance = 100;
        }
        dishesUl.style.transform =
          "translate3d(0px, " + this.distance + "px, 0px)";
      }
      if (this.distance < 0 && this.bottomFlag == true) {
        if (this.distance < -100) {
          this.distance = -100;
        }
        dishesUl.style.transform =
          "translate3d(0px, " + this.distance + "px, 0px)";
      }
    },
    touchEnd() {
      let dishesUl = document.getElementById("income"),
        _this = this;
      if (this.distance > 0 && this.topFlag == true && this.flag) {
        this.flag = false;
        let index = 100;
        let timer = setInterval(function() {
          if (index == 0) {
            clearInterval(timer);
            _this.flag = true;
            _this.distance = 0;
          }
          index--;
          dishesUl.style.transform = "translate3d(0px, " + index + "px, 0px)";
        }, 5);
        this.page = 1;
        this.allLoaded = true;
        this.getdata();
      }
      if (this.distance < 0 && this.bottomFlag == true) {
        this.loadFlag = true;
        let index = -100;
        let timer = setInterval(function() {
          if (index == 0) {
            clearInterval(timer);
          }
          index++;
          dishesUl.style.transform = "translate3d(0px, " + index + "px, 0px)";
        }, 5);
        ++this.page;
        this.getdata();
      }
    },
    //跳转至核销列表页面
    bill() {
      let obj = {
        start: this.start,
        end: this.end,
        actday:this.actday
      };
      this.$router.push({ name: "Bill", params: obj });
    }
  }
};
</script>

<style lang="less">
.income {
  background-color: #ebebeb;
  .income_top {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    z-index: 1000;
    background-color: #fc5e2d;
  }
  p {
    margin: 0;
  }
  width: 100%;
  height: 100%;
  font-family: "微软雅黑";
  position: relative;
  font-size: 30px;
  .iconimg {
    //   margin-top:10px;
    transform: rotate(90deg);
    border: 1px solid red;
  }
  .income_banner {
    margin-top: 82px;
    background-color: #fc5e2d;
    display: flex;
    height: 264px;
    color: #fff;
    .income_banner_data {
      flex: direction-flex;
      width: 50%;
      height: 100%;
      padding-top: 60px;
      box-sizing: border-box;
      p {
        font-size: 30px;
      }
      .income_data_num {
        margin-bottom: 20px;
        font-size: 54px;
      }
    }
  }
  .income_operate {
    height: 120px;
    padding: 20px 41px;
    box-sizing: border-box;
    background-color: #fff;
    display: flex;
    & > div {
      flex: direction-flex;
      justify-content: space-between;
    }
    .income_operate_result {
      text-align: left;
      width: 90%;
      p:first-child {
        font-size: 32px;
        color: #1f1f1f;
      }
      p:last-child {
        font-size: 24px;
        color: #808080;
        span:first-child {
          margin-right: 39px;
        }
      }
    }
    .income_operate_time {
      text-align: right;
      width: 10%;
      img {
        margin-top: 15px;
        width: 47px;
        height: 47px;
      }
    }
  }
  .income_list {
    ul {
      padding: 0;
      padding-top: 30px;
      margin: 0;
      position: relative;
      &:before {
        content: "加载中..";
        position: absolute;
        left: 0;
        top: -50px;
        height: 20px;
        width: 100%;
      }
      li {
        display: flex;
        margin: 0 30px 20px 30px;
        padding: 30px;
        box-sizing: border-box;
        text-align: left;
        background-color: #fff;
        color: #b1b1b1;
        font-size: 22px;
        & > div {
          flex: direction-flex;
          justify-content: space-between;
        }
        .income_li_l {
          width: 50%;
          .income_li_l_money {
            font-size: 50px;
            color: #191919;
          }
        }
        .income_li_r {
          width: 50%;
          .left {
            float: left;
            p:nth-child(1) {
              margin-bottom: 5px;
            }
            p:nth-child(2) {
              margin-bottom: 5px;
            }
          }
          .right {
            float: left;
            margin-left: 20px;
            p:nth-child(1) {
              margin-bottom: 5px;
            }
            p:nth-child(2) {
              margin-bottom: 5px;
            }
          }
        }
      }
      .loadingBox {
        background-color: #ebebeb;
        text-align: center;
        height: 30px;
        line-height: 30px;
        width: 100%;
        padding: 0;
        margin: 0;
        display: block;
        position: absolute;
        bottom: -30px;
        left: 0;
      }
    }
  }
  .mobox {
    width: 100%;
    position: fixed;
    left: 0;
    top: 0;
    z-index: 1001;
    overflow: auto;
    height: 100%;
    background: rgba(0, 0, 0, 0.44);
    font-size: 30px;
    .triangle {
      width: 0px;
      height: 0px;
      border-right: 15px solid rgba(0, 0, 0, 0);
      border-bottom: 15px solid #fff;
      border-left: 15px solid rgba(0, 0, 0, 0);
      float: right;
      margin-top: 65px;
      margin-right: 43px;
    }
    .moreday {
      width: 126px;
      height: 220px;
      background: #fff;
      float: right;
      margin-right: -70px;
      margin-top: 79px;
      border-radius: 20px;
      .adays {
        width: 80%;
        margin-left: 10%;
        height: 70px;
        line-height: 70px;
        font-style: normal;
        border-bottom: 1px solid #b1b1b1;
      }
      .adays:nth-last-child(1) {
        border: none;
      }
    }
  }
  .filling {
    width: 100%;
    height: 460px;
  }
  .inbox {
    position: absolute;
    top: 460px;
    width: 100%;
    background-color: #ebebeb;
  }
  .select {
    width: 672px;
    height: 350px;
    background: #fff;
    position: absolute;
    top: 50%;
    z-index: 5;
    left: 50%;
    margin-left: -336px;
    margin-top: -275px;
    .select_top {
      width: 100%;
      height: 100px;
      line-height: 100px;
      text-align: center;
    }
    .date {
      width: 100%;
      height: 144px;
      div {
        height: 70px;
        line-height: 70px;
        text-align: left;
        width: 80%;
        margin-left: 10%;
        span {
          color: #fc5e2d;
        }
      }
    }
    .selbut {
      width: 100%;
      height: 103px;
      border-top: 1px solid #e0e0e0;
      div {
        float: left;
        height: 100%;
        width: 49.8%;
        line-height: 103px;
        text-align: center;
      }
      .close {
        border-right: 1px solid #e0e0e0;
      }
    }
  }
}
</style>