<script>
  import _ from 'lodash'

  export default {
    name: 'zxt-table-list',
    props: {
      headers: {
        type: Array
      },
      datas: {
        type: Array
      },
      unkey:{
        type:String|Number,
        required:true
      },
      opers:{
        type:Array
      }
    },
    components: {
      OperEl: {
        props: {
          buttons: {
            type:Array
          },
          data:{
            type:Object
          }
        },
        render (h) {
          const parent = this.$parent
          const children = []
          if (parent.opers) {
            if (Array.isArray(this.buttons)) {
              let _this = this;
              this.buttons.forEach(bt => {
                let btn = h('el-button', {
                  attrs: {
                    'type':bt.type,
                    'size':bt.size
                  },
                  on: {
                    'click':() => {
                      if(bt.callBack && typeof bt.callBack ==='function'){
                        bt.callBack(_this.data)
                      }
                    }
                  }
                },bt.name);
                return children.push(btn)
              })
            }
          }
          return h('td', {
            'class': 'custd'
          }, [
            ...children
          ])
        },
        methods:{

        }
      }
    },
    data() {
      return {
        rowspanDatas: new Map()
      }
    },
    methods: {
      sortData: function (datas) {
        let sorts = _.map(_.filter(this.headers, 'sort'), "lable");
        datas = _.orderBy(datas, sorts);
        datas = this.processData(datas);
        return datas;
      },
      processData: function (datas) {
        let _this = this;
        let lables = _.map(_.orderBy(this.headers, ['id'], ['asc']),"lable");
        let results=[];
        let tempMap ={};
        datas.forEach(function (data) {
          let rows =[];
          _.forEach(lables, function(lable) {
            let _obj = {};
            let header = _.find(_this.headers, function(o) {
              return o.lable === lable;
            });
            //先看有没有添加，如果添加取出，rs加1，如果没有创建元素
            if(header.rowspan){
              let _templable = data[_this.unkey]+"_"+lable+"_"+data[lable];
              let _val = tempMap[_templable];
              if(_val == null || _val == undefined){
                tempMap[_templable]=1;
                _obj.lable = lable;

                _obj.value = data[lable];
                _obj.uuid = _templable;
                _obj.rs = 1;
              }else{
                tempMap[_templable]=_val+1;
                _obj.lable = lable;
                _obj.value = data[lable];
                _obj.addedrs = true;
              }
            }else{
              _obj.lable = lable;
              _obj.value = data[lable];
            }
            rows.push(_obj);
          });
          results.push({bf:data,af:rows});
        });
        //更新rowspan，找出集合中所有rs对象

        _.forOwn(tempMap, function(value, key) {
          _.forEach(results, function(rowArr) {
            let obj = _.find(rowArr.af, { 'rs': 1,'uuid':key});
            if(obj != null &&  obj != undefined){
              obj.rs = value;
            }
          });
        });
        return results;
      },
    },
    computed: {
      getDatas() {
       return this.sortData(this.datas);
      }
    },
    watch: {

    }
  }
</script>
<template>
  <div class="wrapDiv">
    <!--{{getDatas}}-->
    <table class="cusTable">
      <thead>
      <tr class="custheadtr">
        <th class="custheadth" v-for="header in headers" v-bind:key="header.name">
          {{header.name}}
        </th>
      </tr>
      </thead>
      <tbody>
      <tr class="custr" v-for="(rowArr,index) in getDatas">
        <template v-for="row in rowArr.af">
              <template v-if="row.rs">
                <td  class="custd" :rowspan="row.rs">
                  {{row.value}}
                </td>
              </template>
              <template v-else-if="row.addedrs">

              </template>
              <template v-else>
                <td  class="custd">
                  {{row.value}}
                </td>
              </template>
        </template>
        <template v-if="opers">
          <OperEl :buttons="opers" :data="rowArr.bf"></OperEl>
        </template>
      </tr>
      </tbody>
    </table>
  </div>
</template>
<style lang="less" scoped>
  .wrapDiv {
    width: 100%;
    overflow-x: auto;
  }

  .custheadtr {
    border: 1px solid #DFE6EC;
  }


  .custheadth {
    height: 40px;
    background: #EEF1F6;
    border: 1px solid #DFE6EC;
  }

  .cusTable {
    border-left: 1px solid #DFE6EC;
    width: 150%;
  }

  .custr {
    border-bottom: 1px solid #DFE6EC;
  }
  .custd {
    border-right: 1px solid #DFE6EC;
    height: 40px;
    padding: 5px;
  }

  .cusTipsClass {
    white-space: normal;
    overflow: hidden;
    text-overflow: ellipsis;
    display: block;
    vertical-align: middle;
  }
</style>
