# Vue-paging
使用Vue组件实现的分页

### 描述
>  本人也是新手小白，具体的内容实现也是在网上找的，然后改改啦，好了下面进入正题；
>  想要实现的功能：
>  * 1 第一要素当然是要分页正常。
>  * 2 更新数据后页面不算新，但是地址栏参数要变
>  * 3 点完分页后刷新页面，要保持在当前页啦

### 效果图

 ![Aaron Swartz](http://wx1.sinaimg.cn/mw690/0060lm7Tly1frbvgxsging30bp01rgn3.gif)
 
### 直接上代码了

> 父组件设置模板，计算分页

```
 // 分页模板
  var tm =`<ul class="pagination">
      <li><a href="javascript:;" @click="setCurrent(current - 1)" > << </a></li>
      <li><a href="javascript:;" @click="setCurrent(1)"> 首页 </a></li>    
      <li v-for="item in list" :class="{'active': current == item.val}" >
      <a href="javascript:;" @click="setCurrent(item.val)">{{item.text}}</a>
      </li>
      <li><a href="javascript:;" @click="setCurrent(total_page)"> 尾页 </a></li>
      <li><a href="javascript:;" @click="setCurrent(current + 1)"> >> </a></li>
   </ul>`;
 
 // 创建全局组件
 Vue.component("test-paging", {
  template:tm,
  data:function(){
    return {
      current : this.current_page
    }
  },
  props:{
    paging:{
      type:Object
    },
    current:{
      default:1
    },
    page_group:{// 分页显示条数
      type:Number,
      default:5,
      coerce:function(v){
        v = v > 0 ? v : 5;
        return v % 2 === 1 ? v : v + 1;
      }
    }
  },
  computed:{
    total_row:function(){ // 总共多少条数据
      return this.paging.total_row;
    },
    total_page:function(){  // 总页数
      return this.paging.total_page
    },
    current_page:function(){  // 当前页
      let curp = 1;
      if(this.paging.current_page){
        curp = this.paging.current_page
      }
      return curp;
    },

    list:function(){ //获取分页页码列表
      let len = this.total_page;
      var temp = [], list = [];
      let count = Math.floor(this.page_group / 2);
      let center  =this.current;

      if(len <= this.page_group){
        while(len--){
          temp.push({text:this.total_page - len, val:this.total_page - len});
        }
        return temp;
      }

      while(len--){
        temp.push(this.total_page - len);
      }
      

      let idx = temp.indexOf(center);
      if(idx < count){
        center = center + count - idx;
      }
      if(this.current > this.total_page - count){
        center = this.total_page - count;
      }
      temp = temp.splice(center - count - 1, this.page_group);

      do {
        var t = temp.shift();
        list.push({
          text: t,
          val: t
        });
      } while (temp.length);
      if(this.total_page > this.page_group){
        if(this.current > count + 1){
          list.unshift({text:'...', val:list[0].val - 1})
        }

        if(this.current < this.total_page - count){
          list.push({text:'...', val:list[list.length - 1].val + 1})
        }

      }
      return list;
    }
  },
  methods: {
    setCurrent: function (idx) {
      // 监听并向父组件传递值
      if (this.current != idx && idx > 0 && idx < this.total_page + 1) {
        this.current = idx;
        this.$emit('pagechange',idx );
      }
    }
  }
});
 
 ```
 
 > style
 
 ```
 /*
  分页组件样式
  样式是半透明的，需要自己设置背景看的清哟。
*/

.paging{
  height: 40px;
  line-height: 40px;
  width: 100%;
  text-align: center;
  border-radius: 0;
}

.paging .pagination {
  margin: 0;
  height: 40px;
  line-height: 40px;
  -webkit-border-radius: 0;
  -moz-border-radius: 0;
  border-radius: 0;
}

.paging .pagination {
  border: 1px solid rgba(255, 255, 255, 0.7);
  background: rgba(255, 255, 255, 0.15);
  -webkit-box-shadow: outset 2px 5px 10px rgba(0, 0, 0, 0.35);
  -moz-box-shadow: outset 2px 5px 10px rgba(0, 0, 0, 0.35);
  box-shadow: outset 2px 5px 10px rgba(0, 0, 0, 0.35);
}


.pagination > li > a,
.pagination > .disabled > a,
.pagination > .disabled > a:focus,
.pagination > .disabled > a:hover,
.pagination > li > a:focus,
.pagination > li > a:hover,
.pagination > li > span:focus,
.pagination > li > span:hover {
  padding: 10px;
  color: #fff;
  border: none;
  background: none;
  font-weight: 500;
}
.pagination > li > a:focus,
.pagination > li > a:hover,
.pagination > li > span:focus,
.pagination > li > span:hover {
  font-weight: 600;
}

.pagination>.active>a {
  color: #5ac8fa !important;
  font-weight:600;
  background-color: initial !important;
  border-color: initial !important;
}
 ```
 
 > 父组件
 
 ```
 <div class="paging mt50">
  <test-paging v-bind:paging="paging" @pagechange="pagechange"></test-paging>
 </div>

<script>
 $(document).ready(function(){

   //获取地址栏中的分页参数
   let page = GetParameterString('page');

   // 页面第一次加载 异步获取数据
   httpdList(page);


   // 填充 列表 数据
   var vm = new Vue({
     el:'#list',
     data:{
       lists:'',
       paging:''
     },
     methods:{
       pagechange:function(currentPage){
         // 获取写的列表数据
         httpdList(currentPage);
         // 更新浏览器地址
         SetAddress(currentPage);
       }
     }
   });


  function httpdList(info){
    let page = info || 1;
    $.post(
      "url",
      { 'pagesize': pagesize,
        'page' :  page
      },
      function(result){
        if(result.status){
          alert(result.msg);
          return;
        }
        var data = result.result;
        vm.lists = data.list; // 列表数据填充
        vm.paging = data.page; // 分页数据数据填充
      });
  }
 });
</script>

 ```
 
> 最后放上两个 获取地址栏参数和修改地址栏参数的函数

```

//  获取地址栏参数
function GetParameterString(name){
  var reg = new RegExp("(^|&)"+ name +"=([^&]*)(&|$)");
  var r = window.location.search.substr(1).match(reg);
  if(r!=null)return  unescape(r[2]); return null;
}

//  设置地址栏显示，异步改变数据时 想将地址栏一起更改
function SetAddress(page){
  let url = window.location.pathname;
  let search = window.location.search;
  let surl = '';


  if(/page=[0-9]+/.test(search)){
    search = search.replace(/page=[0-9]+/,'page='+page);
    
  }else{
    if(search){
      search += '&page=' +page;
    }else{
      search += '?page=' +page;
    }
  }

  if(! /main\.html#/.test(url)){
    surl = '/main.html#'+url+search
  }else{
    surl = url+search;
  }
  window.history.pushState(null,document.title,surl);

//  如果使用 iframe 请使用下面这个
//  window.parent.history.pushState(null,document.title,surl);
}
```
