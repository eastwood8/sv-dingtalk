<template>
  <div class="app-container">
    <banner v-bind:banner-metas="meta.bannerMetas" v-bind:corp-id="corpId"></banner>
    <item v-bind:item-metas="meta.itemMetas" v-bind:corp-id="corpId"></item>
    <admin v-bind:admin="meta.admin" v-bind:user-info="meta.userInfo" v-bind:corp-id="corpId"></admin>
    <userlist v-bind:userlist-metas="meta.userlistMetas" v-bind:corp-id="corpId"></userlist>
    <applist v-bind:allapplist-metas="meta.allapplistMetas" v-bind:corp-id="corpId"></applist>
    <appmanager v-if="openAppManager" v-bind:admin="meta.admin" v-bind:corp-id="corpId"></appmanager>
  </div>
</template>

<script>
import metaData from "../../mock/meta";
import { parseCorpId, parseMetaData, parseMicroApps } from "../../lib/util";

import { OPENAPIHOST } from "../../lib/env.js";
import dingWISDK from "../../lib/ding-web.js";

import banner from "./components/index-banner.vue";
import applist from "./components/index-applist.vue";
import item from "./components/index-item.vue";
import admin from "./components/index-admin.vue";
import userlist from "./components/index-userlist.vue";
import appmanager from "./components/index-appManager.vue";

import axios from "axios";

export default {
  name: "home",
  components: {
    banner: banner,
    applist: applist,
    item: item,
    admin: admin,
    userlist: userlist,
    appmanager: appmanager
  },
  data: function() {
    return {
      openAppManager: true,
      corpId: "",
      userId: "xiangxiaoqiang001",
      meta: {
        admin: false,
        bannerMetas: {
          homeBannerModels: [],
          isHomeBannerModels: false
        },
        itemMetas: {
          homeHeaderModel: {},
          act: "这里是什么？",
          // isAdminOrBoos: false,
          isAdminOrBoos: true,
          attendanceUrl: "",
          myTasksUrl: "",
          checkinsUrl: "",
          notReadReportUrl: "",
          // isHomeHeaderModel: false
          isHomeHeaderModel: true
        },
        userlistMetas: {
          homeGroups: [],
          renderLine: 0,
          isHomeGroups: false
        },
        allapplistMetas: {
          microApps: [],
          isMicroApps: false,
          corpId: ""
        },
        h5Config: {},
        userInfo: {
          name: ""
        }
      }
    };
  },
  mounted: function() {
    const self = this;
    const originalUrl = location.href;
    this.corpId = parseCorpId(originalUrl, "corpId");

    //监听userId的变化，如果有变化，立即获取用户信息
    this.$watch("userId", function() {
      self.getUserInfo();
      self.getMicroApps();
    });
    metaData.microApps.length = 0;

    if (self.userId) {
      self.getUserInfo();
      self.getMicroApps();
    }

    this.meta.bannerMetas.homeBannerModels = metaData.homeBannerModels;
    this.meta.bannerMetas.isHomeHeaderModel = true;
    this.meta.itemMetas.homeHeaderModel = metaData.homeHeaderModel;

    const signRequest = {
      url: OPENAPIHOST + "/api/auth/jsapi-oauth",
      params: {
        href: location.href
      }
    };

    // js-api 权限校验
    dingWISDK
      .jsApiOAuth(signRequest)
      .then(function(response) {
        const data = response.data;
        if (data.errcode === 0) {
          // console.log('config110 : ', data);

          // 获取开放的全部应用
          //   self.getMicroApps();

          // const meta = parseMetaData(metaData);
          // self.meta = meta;
          // self.openAppManager = true;

          // 走免登的流程
          self.getUserId();
        } else {
          alert("获取jsapi oauth 异常：" + data.errmsg);
        }
      })
      .catch(function(error) {
        console.warn("JS API 权限校验失败 error : " + JSON.stringify(error));
      });
  },
  methods: {
    getUserId: function() {
      // 获取userid
      const self = this;
      const getUserIdRequest = {
        url: OPENAPIHOST + "/api/users/auth"
      };
      dingWISDK
        .getUserId(getUserIdRequest, this.corpId)
        .then(function(response) {
          const data = response.data;
          self.userId = data.userid;
        })
        .catch(function(error) {
          alert("获取userid error ：" + JSON.stringify(error));
        });
    },
    getUserInfo: function() {
      // 根据userid获取用户详细信息
      const self = this;
      const getUserInfoRequest = {
        url: OPENAPIHOST + "/api/users/" + self.userId,
        params: {
          userid: this.userId
        }
      };
      dingWISDK
        .getUserInfo(getUserInfoRequest)
        .then(function(response) {
          const data = response.data;
          self.meta.userInfo = data;
        })
        .catch(function(error) {
          console.warn("获取用户信息 error：" + JSON.stringify(error));
        });
    },
    getMicroApps: function() {
      // 获取Apps数 据
      const self = this;
      const microAppsRequest = {
        url: OPENAPIHOST + "/api/microapps/user/" + self.userId
      };
      dingWISDK
        .getMicroApps(microAppsRequest)
        .then(function (response) {
          const data = response.data;
          if (data.errcode === 0) {
            self.meta.allapplistMetas = parseMicroApps(data.appList);

            // console.log(self.meta.allapplistMetas);
          }
        })
        .catch(function(error) {
          alert("获取microApps error：" + JSON.stringify(error));
        });
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
.app-container {
  width: 100%;
  height: 100%;
  overflow: hidden;
  overflow-y: scroll;
}
</style>
