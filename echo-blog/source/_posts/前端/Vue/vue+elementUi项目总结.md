---
title: vue+element
categories:
- 数据库
tags: vue项目总结
date:
---

1. 对vue项目的各种插件(elementUi等)、依赖(axios、less-loader、less等)管理和使用；
2. vue项目对form表单的使用：
	a、对el-form绑定大对象用:model="xxxForm"，但是针对el-form-item绑定对象使用的是++v-model++='xxxForm.xxxname';
	b、对el-form绑定引用对象ref='xxxFormRef',引用对象的作用非常强大，可以通过引用对象直接获取实例对象里面的属性和方法(关于引用对象的名称建议做到见名知意)；
	c、针对form表单需要进行属性校验和表单重置操作时，必须在el-form-item标签中绑定prop='xxxname'属性，才能生效；
3. 关于布局position决定定位属性的应用(前提是决定某个属性是固定在某个位置不动的，例如永久居中布局)
4. 对form表单的预验证操作(即真正提交数据前做一次校验)，使用form表单的引用对象调用validate方法来实现：
this.$refs.xxxFormRef.validate(valid => {
	省略代码。。。
});
5. 在main.js中使用Vue.prototype 来定义全局变量----->>axios
6. elementUi对Message组件的导入和其他组件不一样，如下：
~~~
import Vue from 'vue'
import { Button, Form, FormItem, Input, Icon, Message } from 'element-ui'

Vue.use(Button)
Vue.use(Form)
Vue.use(FormItem)
Vue.use(Input)
Vue.use(Icon)
//关于弹窗信息提示注册到项目中的方法和其他组件不一样，要注意
Vue.prototype.$message = Message
最终的使用： 
if (res.meta.status != 200) return this.$message.error("登录失败");
this.$message.success("登录成功");
~~~
7. 路由导航守卫控制访问权限的应用：如果有token验证通过，才可以进行其他访问操作，否则会强制去进行登录操作
//挂载路由导航守卫控制访问权限,要注意next()什么时候有参数，什么时候没有参数：只有当没有token时，next方法是直接指向'/login'路由的；其他都是直接放行；
router.beforeEach((to, from, next) => {
  /**
    to:即将进入的目标路由对象
    from:即将离开的目标路由对象
    next：执行下一个钩子，一定要用，否则无法对正在使用的钩子无法进行释放
   */
  //1、判断to的路由是否登录，是则直接放行
  if (to.path === '/login') return next();
  //2、不是，则判断是否有token验证
  const token = window.sessionStorage.getItem('token');
  //3、有token,则进入到指定路由
  if (!token) return next('/login');
  //4、没有则指定到登录路由
  next();
})
8. 退出的操作使用的思想是清空缓存：window.sessionStorage.clear();
9. 菜单栏的使用：
    ~~~
     <el-menu
            background-color="#2D3039"
            text-color="#fff"
            active-text-color="red"
            unique-opened
            :collapse="isOpen"
            :collapse-transition='false'
            router
          >
            <!--一级菜单-->
            <el-submenu 
                :index="menu.id +''" v-for="menu in menuList" :key="menu.id"
                            >
              <template slot="title" >
                <!--图标-->
                <i :class="menuIconObj[menu.id]"></i>
                <!--文本-->
                <span>{{menu.authName}}</span>
              </template>
              <!--二级菜单-->
                <el-menu-item :index="'/' + subMenu.path" v-for="subMenu in menu.children" :key="subMenu.id">
                    <template slot="title">
                    <i class="el-icon-menu"></i>
                    <span>{{subMenu.authName}}</span>
                    </template>
                </el-menu-item>
            </el-submenu>
          </el-menu>
**特别注意**：二级菜单是根据el-menu中的router属性及el-menu-item中的:index属性来实现页面跳转绑定的；

10. vue前端请求后台是请求参数格式：
  ~~~
 //查询用户列表信息
    async queryUserList() {
      const { data: res } = await this.$http.get("/users",
        {
          params: this.queryParams
        }
      );
      if (res.meta.status != 200) return this.$message.error(this.res.meta.msg);
      // console.log(res);
      this.userList = res.data.users;
      this.total = res.data.total;
    }
~~~
注意：参数是以一个对象的格式定义的，且参数的件为params,值可以为对象或其他；
11. 插槽作用域的使用(非常强大，非常适用):
应用场景：针对列表中数据进行操作，使用插槽作用域能够非常非常方便的获取到当前行的信息(也就是当前行所对应对象的所有信息，包括id等等)；
插槽作用域的定义格式如下：
 ~~~
<template slot-scope='scope'>
  {{scope.row}}		//获取到当前行对象的所有属性信息
</template>
~~~
12. v-for循环使用时注意事项：
- v-for真正放在的位置应该是信息项展示位置的上一层标签中(例如：某一列el-col需要展示数据信息时,应该将v-for放在el-row);
- v-for中的属性index应该是放在item属性之后的，代表索引；
13. vue中可以使用`<pre>`标签对作用于插槽中的数据信息进行json格式化处理，方便查看层级关系；
