<template>
  <div class="aboutMe">
        <mt-header title="关于我们"><router-link slot="left" :to="{path:'home',query:{'ind':index}}">
        <mt-button icon="back"></mt-button></router-link></mt-header>
        <div class="weWrap">
            <img :src="data.src" alt="享7Logo">
            <p>{{data.name}}<span>{{data.versionNumber}}</span></p>
        </div>
        <div class="Our_information" v-for="(item,index) in data1" :key='index'>
            <div class="Information_details">
                <p>{{item.name}}</p>
                <span>{{item.heEntityName}}</span>
            </div>
        </div>
  </div>
</template>

<script>
import { MessageBox } from "mint-ui";
export default {
  name: "aboutMe",
  data() {
    return{
          index:2,
          msg:'aboutMe',
          data:{
            //   src: "../../../../static/images/xiang7Logo.png",
              src: require("../../../../static/images/xiang7Logo.png"),
              name:'版本号:',
              versionNumber:'V1.0',
          },
          data1:[
            {
                name:'微信公众号',
                heEntityName:'享7美食'
            },{
                name:'商务合作',
                heEntityName:'xiangqikeji2018@163.com'
            },{
                name:'官方网站',
                heEntityName:'http://www.xiang7.net'
            },{
                name:'检查更新',
                heEntityName:'当前最新版本'
            }
          ]
      }
  }
};
</script>

<style lang="less">
    .weWrap{
        width: 100%;
        height: 331px;
        background: #FC5E2D;
        margin-top: -1px;
        img{
            width: 147px;
            height: 147px;
            padding-top: 60px;
        }
        p{
            font-size: 30px;
            color: #FFFFFF;
        }
    }
    .Our_information{
            background: #FFFFFF;
            display: flex;
            justify-content: center;
            .Information_details{
                width: 90%;
                height: 107px;
                border-bottom: 1px solid #E0E0E0;
                display: flex;
                justify-content: space-between;
                align-items: center;
                font-size: 30px;
                color: #191919;
                b{
                    font-size: 36px;
                    font-weight: 550;
                }
                span{
                    font-size: 30px;
                    color: #808080;
                }
            }
        }
</style>

