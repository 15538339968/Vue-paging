# Vue-paging
使用Vue组件实现的分页

### 描述
>  本人也是新手小白，具体的内容实现也是在网上找的，然后改改啦，好了下面进入正题；
>  想要实现的功能：
>  * 1 第一要素当然是要分页正常。
>  * 2 更新数据后页面不算新，但是地址栏参数要变
>  * 3 点完分页后刷新页面，要保持在当前页啦

### 效果图
 ![分页效果图]（）
 
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
