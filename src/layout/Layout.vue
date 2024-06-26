<template>
  <a-layout>
    <a-affix>
      <!-- 布局头部 -->
      <a-layout-header>
        <!-- 平台名 -->
        <div style="float:left;">
          <img style="height:40px;margin-bottom:10px;" :src="kubeLogo" />
          <span style="padding:0 50px 0 20px;font-size:25px;font-weight:bold;color:#fff">KubeA</span>
        </div>
        <!-- 集群 -->
        <a-menu
            style="float:left;width:250px;"
            v-model:selectedKeys="selectedKeys1"
            theme="dark"
            mode="horizontal"
            :style="{ lineHeight: '64px' }">
          <a-menu-item v-for="(item) in clusterList" :key="item" @click="clusterChange(item)">{{ item }}</a-menu-item>
        </a-menu>
        <!-- 用户信息 -->
        <div style="float:right;">
          <img style="height:40px;border-radius:50%;margin-right:10px;" :src="avator"/>
          <a-dropdown style="margin-top: 10px;" :overlayStyle="{paddingTop: '20px'}">
            <a class="ant-dropdown-link" @click.prevent>
              Admin
              <down-outlined />
            </a>
            <template #overlay>
              <a-menu>
                <a-menu-item>
                  <a @click="logout()">退出登录</a>
                </a-menu-item>
                <a-menu-item>
                  <a href="javascript:;">修改密码</a>
                </a-menu-item>
              </a-menu>
            </template>
          </a-dropdown>
        </div>
      </a-layout-header>
    </a-affix>
    <a-layout style="height:calc(100vh - 68px);">
      <!-- 侧边栏，核心 -->
      <!-- collapsed处理展开和收缩  -->
      <a-layout-sider width="240" style="background: #fff" v-model:collapsed="collapsed" collapsible>
        <!-- selectedKeys表示点击选中的栏目,用于a-menu-item -->
        <!-- openKeys表示展开的栏目，用于a-sub-menu -->
        <!-- openChange事件监听 SubMenu 展开/关闭的回调 -->
        <a-menu
            :selectedKeys="selectedKeys2"
            :openKeys="openKeys"
            @openChange="onOpenChange"
            mode="inline"
            :style="{ height: '100%', borderRight: 0 }">
          <!-- routers是router/index.js中的路由信息 -->
          <template v-for="menu in routers" :key="menu">
            <!-- 处理无子路由的情况，也就是单个栏目 -->
            <!-- routeChange用于路由跳转 -->
            <a-menu-item
                v-if="menu.children && menu.children.length == 1"
                :index="menu.children[0].path"
                :key="menu.children[0].path"
                @click="routeChange('item', menu.children[0].path)">
              <template #icon>
                <component :is="menu.children[0].icon" />
              </template>
              <span>{{ menu.children[0].name }}</span>
            </a-menu-item>
            <!-- 处理有子路由的情况，也就是父栏目 -->
            <a-sub-menu
                v-else-if="menu.children && menu.children.length > 1"
                :index="menu.path"
                :key="menu.path">
              <template #icon>
                <component :is="menu.icon" />
              </template>
              <template #title>
                                <span>
                                    <span :class="[collapsed ? 'is-collapse' : '']">{{ menu.name }}</span>
                                </span>
              </template>
              <!-- 子栏目（子路由）的处理 -->
              <a-menu-item
                  v-for="child in menu.children"
                  :key="child.path"
                  :index="child.path"
                  @click="routeChange('sub', child.path)">
                <span>{{ child.name }}</span>
              </a-menu-item>
            </a-sub-menu>
          </template>
        </a-menu>
      </a-layout-sider>
      <a-layout style="padding: 0 24px">
        <!-- 面包屑 -->
        <a-breadcrumb style="margin: 16px 0">
          <a-breadcrumb-item>工作台</a-breadcrumb-item>
          <!-- router.currentRoute.value.matched表示路由的match信息，能拿到父路由和子路由的信息 -->
          <template v-for="(matched,index) in router.currentRoute.value.matched" :key="index">
            <a-breadcrumb-item v-if="matched.name">
              {{ matched.name }}
            </a-breadcrumb-item>
          </template>
        </a-breadcrumb>
        <!-- main的部分 -->
        <a-layout-content
            :style="{
                    background: 'rgb(31, 30, 30)',
                    padding: '24px',
                    margin: 0,
                    minHeight: '280px',
                    overflowY: 'auto'}">
          <!-- 路由占位符 -->
          <router-view></router-view>
        </a-layout-content>
        <!-- footer部分 -->
        <a-layout-footer style="text-align: center">
          ©2024 Created by 张钦博 Devops
        </a-layout-footer>
      </a-layout>
    </a-layout>
  </a-layout>
</template>

<script>
import kubeLogo from '@/assets/k8s-metrics.png'
import {onBeforeMount, onMounted, reactive, ref} from "vue";
import {useRouter} from "vue-router";
import httpClient from "@/request";
import common from "@/config"
import {message} from "ant-design-vue";
import avator from "@/assets/avator.png"
export default ({
  setup() {
    const clusterList = ref()
    const selectedKeys1 = ref([])
    const appLoading = ref(false)
    const clusterListData = reactive({
      url: common.k8sClusterList,
    })

    const selectedKeys2 = ref([])
    const openKeys =ref([])

    const router = useRouter()
    const routers = ref([])
    const collapsed = ref(false)

    //以下是k8s集群相关
    const getClusterList =() => {
      appLoading.value = true
      httpClient.get(clusterListData.url)
          .then(res => {
            clusterList.value = res.data
            localStorage.setItem('cluster_num',clusterList.value.length)
            if (!selectedKeys1.value.length) {
              selectedKeys1.value[0] = clusterList.value[0]
              localStorage.setItem('k8s_cluster',clusterList.value[0])
            }
          })
          .catch(res => {
            message.error(res.msg)
      })
          .finally(()=> {
            appLoading.value = false
          })
    }

    // 鼠标单击时更改 item
    const  clusterChange = (val) => {
        if (selectedKeys1.value[0] == val){
          return
        }else {
          selectedKeys1.value = val
          localStorage.setItem('k8s_cluster',val)
          location.replace(router.currentRoute.value.path)
        }
    }
    //以上是k8s集群相关

    //以下是用户信息集群相关
    const logout = () => {
      localStorage.removeItem('username')
      localStorage.removeItem('token')
      router.push('/login')
    }
    //以上是用户信息相关

    const onOpenChange = (val) => {
      const latestOpenKey = val.find(key => openKeys.value.indexOf(key) === -1)
      openKeys.value = latestOpenKey ? [latestOpenKey] :[]
    }


    const getRouter = (val) => {
      selectedKeys2.value = [val[1].path]
      openKeys.value = [val[0].path]
    }

    const routeChange = (type,path) => {
      if (type !== 'sub'){
          openKeys.value = []
      }
      selectedKeys2.value = [path]
      if (router.currentRoute.value.path !== path){
          router.push(path)
      }

      console.log("selectedKeys1: ",selectedKeys1.value)
      console.log("selectedKeys2: ",selectedKeys2.value)
      console.log("openKeys: ",openKeys.value)
      console.log("routers: ",routers.value)
    }


    //生命周期钩子
    onBeforeMount(() => {
      getClusterList()
      if (localStorage.getItem('k8s_cluster')) {
        selectedKeys1.value[0] = localStorage.getItem('k8s_cluster')
      }

    });

    onMounted(()=>{
      routers.value = router.options.routes
      getRouter(router.currentRoute.value.matched)
      console.log(routers.value)
      //console.log('router.currentRoute.value.matched: ',router.currentRoute.value.matched)

    })

    return {
      kubeLogo,
      clusterList,
      selectedKeys1,
      clusterChange,
      avator,
      logout,
      selectedKeys2,
      openKeys,
      onOpenChange,
      router,
      routers,
      routeChange,
      collapsed

    }
  },
})
</script>


<style scoped>
.ant-layout-header {
  padding: 0 30px !important;
}
.ant-layout-content::-webkit-scrollbar {
  width:6px;
}
.ant-layout-content::-webkit-scrollbar-track {
  background-color:rgb(164, 162, 162);
}
.ant-layout-content::-webkit-scrollbar-thumb {
  background-color:#666;
}
.ant-layout-footer {
  padding: 5px 50px !important;
  color: rgb(239, 239, 239);
}
.is-collapse {
  display: none;
}
.ant-layout-sider {
  background: #141414 !important;
  overflow-y: auto;
}
.ant-layout-sider::-webkit-scrollbar {
  display: none;
}
.ant-menu-item {
  margin: 0 !important;
}
</style>