<template>
  <panel :loading="loading" :showAction="false">
    <com-top-bar :title="title"></com-top-bar>
    <com-main-body>
      <com-article>
        <section>
          <zxt-table-list :headers="headers" :datas="resultdata" unkey="bpid" >
          </zxt-table-list>

         
        </section>
      </com-article>
      <template>
        <div class="ga-mt-4 ga-mb-4 ga-clear">
          <ga-anchor  :block="true" class="ga-fr" @click="returnBack">返回上一页
            <ga-icon name="angle-right" class="ga-ml-1" ></ga-icon>
          </ga-anchor>
        </div>
      </template>
    </com-main-body>
  </panel>
</template>
<script>

  import projectApi from '../../api/project-api'
  import ActionPanel from '../basic/ActionPanel.vue'
  import ComArticle from "../../coms/ComArticle.vue";
  import dateFormat from "../../lib/date-format";
  import GaIcon from "../../ga/elements/GaIcon.vue";
  import GaAnchor from "../../ga/elements/GaAnchor.vue";
  import ZxtTableList from "../../coms/ZxtTableList.vue";
  import ComMainBody from "../../coms/ComMainBody.vue";
  import ComTopBar from "../../coms/ComTopBar.vue";

  export default {
    components: {
      ComTopBar,
      ComMainBody,
      ZxtTableList,
      GaAnchor,
      GaIcon,
      ComArticle,
      panel: ActionPanel},
    data() {
      return {
         projectId:null,
         loading: false,
         title:'',
         resultdata:[],
         headers:[
           {id:1,name:"项目编码",align:"center",lable:"bpcode",width:'150px',sort:'desc',rowspan:true},
           {id:2,name:"项目名称",lable:"bpname",width:'130px',tips:true,sort:'asc',rowspan:true},
           {id:3,name:"采购委托名称",lable:"epname",width:'150px',sort:'desc',rowspan:true},
           {id:4,name:"采购包",lable:"pname",width:'150px',sort:'asc',rowspan:true},
           {id:5,name:"方案名称",lable:"sname",width:'150px'},
           {id:6,name:"方案编码",lable:"scode",width:'120px'},
           {id:7,name:"选商模式",lable:"smode",width:'120px'},
           {id:8,name:"采购类别",lable:"otype",width:'120px'},
           {id:9,name:"合同草案编号",lable:"ctcode",width:'150px'},
           {id:10,name:"合同编号",lable:"cfcode",width:'150px'},
           {id:11,name:"订单编码",lable:"ocode",width:'150px'},
           {id:12,name:"订单创建人",lable:"ocrtuser",width:'120px'},
           {id:13,name:"创建时间",lable:"ocrtdate",width:'150px'}
         ],
         buttons:[
           {name:'新增',type:'primary',size:'small',callBack:function (data) {
               alert(JSON.stringify(data))
           }},
           {name:'修改',type:'primary',size:'small',callBack:function (data) {
             this.resultdata=_.dropWhile(this.resultdata, function(o) {  });
           }}
         ]
      }
    },

    methods: {
      fetchData: function () {
        let data ={
          bpid:this.$route.params.pcid,
          type : this.$route.params.type
        }
        this.setTitleName(this.$route.params.type);
        if('SCHEME' ===this.$route.params.type){
          projectApi.getAllProjectView(this.$http, data).then(rData => {
            this.resultdata = rData;
          }).catch(edata => {
            this.$notify.error({
              title: this.$t('common.error'),
              message: edata.message
            })
          })
        }else if('EMERGENCY' ===this.$route.params.type){
          projectApi.getAllProjectEmergencyView(this.$http, data).then(rData => {
            this.resultdata = rData;
          }).catch(edata => {
            this.$notify.error({
              title: this.$t('common.error'),
              message: edata.message
            })
          })
        }

      },
      setTitleName:function (type) {
          if('SCHEME' ===type){
             this.title='方案型列表';
          }else if('EMERGENCY' ===type){
             this.title='应急型列表';
          }else if('RENEWAL' ===type){
            this.title='续订型列表';
          }
      },
      formatDate(row, column) {
        var date = row[column.property];
        if (date == undefined) {
          return "";
        }
        return dateFormat.formatDate(new Date(date), 'yyyy-MM-dd');
      },
      returnBack:function () {
         this.$router.go(-1);
      }
    },
    mounted: function () {
      this.fetchData();
    },
    watch: {

    },
  }
</script>

<style>


</style>
