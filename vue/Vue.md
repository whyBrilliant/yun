# Vue

## 使用

1. 组件的引入

   1.1 常见引入方式

   - 通过**import**引入组件
   - 在父组件的组件对象**components**中将导入的子组件注册
   - 在父组件中使用该组件

   ```
   <template>
     <div>
       <Child msg="Hello World！"/>
     </div>
   </template>
   
   <script>
   import Child from "../components/Child";
   
   export default {
     name: "page",
     components: {
       Child
     }
   };
   </script>
   
   ```

   

   1.2 一个会员中心组件，有一部分区域用来展示会员与会员差异的相关展示模块，但是这两个只会展示其中一个，会员与非会员。假设我们现在有**MemberInfo**以及**UserInfo**两个组件,我们想根据用户身份进行对应模块显示.

   1.2.1 v-if v-else

   ```
   <template>
     <div class="container">
       <button @click="isMember = !isMember">{{isMember?'我不想要会员了，哼':'我要成为会员'}}</button>
       <UserInfo v-if="!isMember"/>
       <MemberInfo v-else/>
     </div>
   </template>
   
   <script>
   import UserInfo from "../components/UserInfo";
   import MemberInfo from "../components/MemberInfo";
   
   export default {
     name: "user",
     data() {
       return {
         isMember: false
       };
     },
     components: {
       UserInfo,
       MemberInfo
     }
   };
   </script>
   
   ```

   1.2.2 vue的内置组件component,渲染一个”元组件“为动态组件。根据**is**的值来决定哪个组件被渲染

   ```
   <template>
     <div class="container">
       <button @click="isMember = !isMember">{{isMember?'我不想要会员了，哼':'我要成为会员'}}</button>
       <component :is="userComponentName" title="component就是好用哟"/>
     </div>
   </template>
   
   <script>
   import UserInfo from "../components/UserInfo";
   import MemberInfo from "../components/MemberInfo";
   
   export default {
     name: "userComponent",
     data() {
       return {
         isMember: false
       };
     },
     components: {
       UserInfo,
       MemberInfo
     },
     computed: {
       userComponentName() {
         let { isMember } = this;
         return isMember ? "member-info" : "user-info";
       }
     }
   };
   </script>
   
   ```

   1.2.3 以上，现有的两个模块组件，我们仍然必须先导入并注册才能完成调用。现在，我们就不想提前注册好所需使用的子组件，因为太麻烦了并且浪费性能，我们想尝试动态导入注册使用。

   ```
   <template>
     <div class="container">
       <button @click="isMember = !isMember">{{isMember?'我不想要会员了，哼':'我要成为会员'}}</button>
       <component :is="userComponentImstance" title="component就是好用哟"/>
     </div>
   </template>
   
   <script>
   export default {
     name: "userImport",
     data() {
       return {
         isMember: false
       };
     },
     computed: {
       userComponentImstance() {
         let { isMember } = this;
         let pathName = isMember ? "MemberInfo" : "UserInfo";
         //通过import动态导入组件 配合webpack实现组件分离
         return () => import(`../components/${pathName}`);
       }
     }
   };
   </script>
   
   ```

   优点：code splitting

   缺点：件依赖关系通过拼接的字符串组件地址来保证，组件复杂了之后可能比较分散

   

   1.3 商城首页的几个活动模块根据后端返回顺序依次渲染，假设我们父组件为Home主页面，可能存在**Seckill**秒杀、**Group**团购、**Limit**限购、**Bargain**砍价四个活动模块,模块展示及排序均由数据决定。

   ```
   <template>
     <div>
       <p>假装这是一个商城首页</p>
       <button @click="shuffle">随机切换模块顺序</button>
       <component :is="item.imstance" v-for="(item ,i) in componentImstances" :info="item" :key="i"/>
     </div>
   </template>
   
   <script>
   export default {
     name: "home",
     data() {
       return {
         moduleList: [
           {
             type: "Bargain",
             title: "砍价",
             startTime: "2019-12-01",
             endTime: "2019-01-01"
           },
           {
             type: "Seckill",
   
             title: "秒杀",
             startTime: "2019-12-05",
             endTime: "2019-01-01"
           },
           {
             type: "Limit",
   
             title: "限购",
             startTime: "2019-12-07",
             endTime: "2019-01-01"
           },
           {
             type: "Group",
             title: "团购",
             startTime: "2019-12-11",
             endTime: "2019-01-01"
           }
         ]
       };
     },
     components: {},
     methods: {
       shuffle() {
         let { moduleList } = this;
         let resultArr = [];
         while (moduleList.length > 0) {
           var index = Math.floor(Math.random() * moduleList.length);
           resultArr.push(moduleList[index]);
           moduleList.splice(index, 1);
         }
         this.moduleList = resultArr;
       }
     },
     computed: {
       componentImstances() {
         let { moduleList } = this;
         return moduleList.map(item => {
           item.imstance = () => import(`../components/${item.type}`);
           return item;
         });
       }
     }
   };
   </script>
   
   ```

   为了避免动态导入的组件在未知情况下加载失败，我们可以去做一个异常模板提示。 将上面中的**componentImstances**稍作修改

   ```
   computed: {
       componentImstances() {
         let { moduleList } = this;
         return moduleList.map(item => {
           item.imstance = () => {
             return new Promise((resove, reject) => {
               let imstance = import(`../components/${item.type}`);
               imstance.then(res => {
                 resove(res);
               });
               imstance.catch(e => {
                 resove(import("../components/Error"));
               });
             });
           };
           return item;
         });
       }
     }
   
   ```

   

   