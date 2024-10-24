仿饿了么App	1	2020/3/2	罗安桦	前端	前端开源技术	通过vscode创建项目，通过webpack打包项目	vue2+vue2全家桶	支持微信支付+手机登陆+定位	对接微信支付和高德定位	vue+javascript+css	能支持微信支付，高德定位	支持多人同时使用

  
项目名称
源代码

	提交至少60页源码（50行/页；文本清除格式后，不算封面、空行）
	体现操作手册中对应一二级功能的功能性代码
	最后页应是一个功能模块的完整结束
	若软件全量代码小于3000行，提交全量代码











Login.vue
<template>
  <div class="login">
    <div class="logo">
      <img src="../assets/logo.jpg" alt="my login image">
    </div>
    <!-- 手机号 -->
    <InputGroup
      type="number"
      v-model="phone"
      placeholder="手机号"
      :btnTitle="btnTitle"
      :disabled="disabled"
      :error="errors.phone"
      @btnClick="getVerifyCode"
    />
    <!-- 验证码 -->
    <InputGroup type="number" v-model="verifyCode" placeholder="验证码" :error="errors.code"/>

    <!-- 用户服务协议 -->
    <div class="login_des">
      <p>
        新用户登录即自动注册，表示已同意
        <span>《用户服务协议》</span>
      </p>
    </div>
    <!-- 登录按钮 -->
    <div class="login_btn">
      <button :disabled="isClick" @click="handleLogin">登录</button>
    </div>
  </div>
</template>

<script>
import InputGroup from "../components/InputGroup";
export default {
  name: "login",
  data() {
    return {
      phone: "",
      verifyCode: "",
      errors: {},
      btnTitle: "获取验证码",
      disabled: false
    };
  },
  computed: {
    isClick() {
      if (!this.phone || !this.verifyCode) return true;
      else return false;
    }
  },
  methods: {
    handleLogin() {
      // 取消错误提醒
      this.errors = {};
      // 发送请求
      this.$axios
        .post("/api/posts/sms_back", {
          phone: this.phone,
          code: this.verifyCode
        })
        .then(res => {
          // console.log(res.data);
          // 检验成功 设置登录状态并且跳转到/
          localStorage.setItem("ele_login", res.data.user._id);
          this.$router.push("/");
        })
        .catch(err => {
          // 返回错误信息
          this.errors = {
            code: err.response
          };
        });
    },
    getVerifyCode() {
      if (this.validatePhone()) {
        this.validateBtn();
        // 发送网络请求
        this.$axios
          .post("/api/posts/sms_send", {
            tpl_id: "140481",
            key: "795be723dd9e88c3ea98e2b6713ab795",
            phone: this.phone
          })
          .then(res => {
            console.log(res);
          });
      }
    },
    validateBtn() {
      let time = 5;
      let timer = setInterval(() => {
        if (time == 0) {
          clearInterval(timer);
          this.btnTitle = "获取验证码";
          this.disabled = false;
        } else {
          // 倒计时
          this.btnTitle = time + "秒后重试";
          this.disabled = true;
          time--;
        }
      }, 1000);
    },
    validatePhone() {
      // 验证手机号码
      if (!this.phone) {
        this.errors = {
          phone: "手机号码不能为空"
        };
        return false;
      } else if (!/^1[345678]\d{9}$/.test(this.phone)) {
        this.errors = {
          phone: "请填写正确的手机号码"
        };
        return false;
      } else {
        this.errors = {};
        return true;
      }
    }
  },
  components: {
    InputGroup
  }
};
</script>

<style scoped>
.login {
  width: 100%;
  height: 100%;
  padding: 30px;
  box-sizing: border-box;
  background: #fff;
}

.logo {
  text-align: center;
}
.logo img {
  width: 150px;
}
.text_group,
.login_des,
.login_btn {
  margin-top: 20px;
}
.login_des {
  color: #aaa;
  line-height: 22px;
}
.login_des span {
  color: #4d90fe;
}
.login_btn button {
  width: 100%;
  height: 40px;
  background-color: #48ca38;
  border-radius: 4px;
  color: white;
  font-size: 14px;
  border: none;
  outline: none;
}
.login_btn button[disabled] {
  background-color: #8bda81;
}
</style>




Home.vue
<template>
  <div class="home">
    <div class="header">
      <div class="address_map" @click="$router.push({name: 'address',params: {city: city}})">
        <i class="fa fa-map-marker"></i>
        <span>{{address}}</span>
        <i class="fa fa-sort-desc"></i>
      </div>
    </div>
    <div class="search_wrap" :class="{'fixedview':showFilter}" @click="$router.push('/search')">
      <div class="shop_search">
        <i class="fa fa-search"></i>
        搜索商家 商家名称
      </div>
    </div>
    <div id="container">
      <!-- 轮播 -->
      <mt-swipe :auto="4000" class="swiper">
        <mt-swipe-item v-for="(img,index) in swipeImgs" :key="index">
          <img :src="img" alt>
        </mt-swipe-item>
      </mt-swipe>
      <!-- 分类 -->
      <mt-swipe :auto="0" class="entries">
        <mt-swipe-item v-for="(entry,i) in entries" :key="i" class="entry_wrap">
          <div class="foodentry" v-for="(item,index) in entry" :key="index">
            <div class="img_wrap">
              <img :src="item.image" alt>
            </div>
            <span>{{item.name}}</span>
          </div>
        </mt-swipe-item>
      </mt-swipe>
    </div>
    <!-- 推荐商家 -->
    <div class="shoplist-title">推荐商家</div>

    <!-- 导航 -->
    <FilterView :filterData="filterData" @searchFixed="showFilterView" @update="update"/>

    <!-- 商家信息 -->
    <mt-loadmore
      :top-method="loadData"
      :bottom-method="loadMore"
      :bottom-all-loaded="allLoaded"
      :auto-fill="false"
      :bottomPullText="bottomPullText"
      ref="loadmore"
    >
      <div class="shoplist">
        <IndexShop v-for="(item,index) in restaurants" :key="index" :restaurant="item.restaurant"/>
      </div>
    </mt-loadmore>
  </div>
</template>

<script>
import { Swipe, SwipeItem, Loadmore } from "mint-ui";
import FilterView from "../components/FilterView";
import IndexShop from "../components/IndexShop";
export default {
  name: "home",
  data() {
    return {
      swipeImgs: [],
      entries: [],
      filterData: null,
      showFilter: false,
      page: 1,
      size: 5,
      restaurants: [], // 存放所有商家容器
      allLoaded: false,
      bottomPullText: "上拉加载更多",
      data: null
    };
  },
  computed: {
    address() {
      return this.$store.getters.address;
    },
    city() {
      return (
        this.$store.getters.location.addressComponent.city ||
        this.$store.getters.location.addressComponent.province
      );
    }
  },
  created() {
    this.getData();
  },
  methods: {
    getData() {
      this.$axios("/api/profile/shopping").then(res => {
        // console.log(res.data);
        this.swipeImgs = res.data.swipeImgs;
        this.entries = res.data.entries;
      });
      this.$axios("/api/profile/filter").then(res => {
        // console.log(res.data);
        this.filterData = res.data;
      });
      this.loadData();
    },
    loadData() {
      this.page = 1;
      this.allLoaded = false;
      this.bottomPullText = "上拉加载更多";
      // 拉取商家信息
      this.$axios
        .post(`/api/profile/restaurants/${this.page}/${this.size}`, this.data)
        .then(res => {
          // console.log(res.data);
          this.$refs.loadmore.onTopLoaded();
          this.restaurants = res.data;
        });
    },
    loadMore() {
      if (!this.allLoaded) {
        this.page++;
        // 拉取商家信息
        this.$axios
          .post(`/api/profile/restaurants/${this.page}/${this.size}`)
          .then(res => {
            //  加载完之后重新渲染
            this.$refs.loadmore.onBottomLoaded();
            if (res.data.length > 0) {
              res.data.forEach(item => {
                this.restaurants.push(item);
              });
              if (res.data < this.size) {
                this.allLoaded = true;
                this.bottomPullText = "没有更多了哦";
              }
            } else {
              // 数据为空
              this.allLoaded = true;
              this.bottomPullText = "没有更多了哦";
            }
          });
      }
    },
    showFilterView(isShow) {
      this.showFilter = isShow;
    },
    update(condation) {
      // console.log(condation);
      this.data = condation;
      this.loadData();
    }
  },
  components: {
    FilterView,
    IndexShop
  }
};
</script>

<style scoped>
.home {
  width: 100%;
  height: 100%;
  overflow: auto;
  box-sizing: border-box;
}
.header,
.search_wrap {
  background-color: #009eef;
  padding: 10px 16px;
}
.header .address_map {
  color: #fff;
  font-weight: bold;
}
.address_map i {
  margin: 0 3px;
  font-size: 18px;
}
.address_map span {
  display: inline-block;
  width: 80%;
  overflow: hidden;
  white-space: nowrap;
  text-overflow: ellipsis;
}
.search_wrap .shop_search {
  /* margin-top: 10px; */
  background-color: #fff;
  padding: 10px 0;
  border-radius: 4px;
  text-align: center;
  color: #aaa;
}

.search_wrap {
  position: sticky;
  top: 0px;
  z-index: 999;
  box-sizing: border-box;
}
.swiper {
  height: 100px;
}
.swiper img {
  width: 100%;
  height: 100px;
}

.entries {
  background: #fff;
  height: 47.2vw;
  text-align: center;
  overflow: hidden;
}
.foodentry {
  width: 20%;
  float: left;
  position: relative;
  margin-top: 2.933333vw;
}
.foodentry .img_wrap {
  position: relative;
  display: inline-block;
  width: 12vw;
  height: 12vw;
}
.img_wrap img {
  width: 100%;
  height: 100%;
}
.foodentry span {
  display: block;
  color: #666;
  font-size: 0.32rem;
}
/* 推荐商家 */
.shoplist-title {
  display: flex;
  align-items: flex;
  justify-content: center;
  height: 9.6vw;
  line-height: 9.6vw;
  font-size: 16px;
  color: #333;
  background: #fff;
}
.shoplist-title:after,
.shoplist-title:before {
  display: block;
  content: "一";
  width: 5.333333vw;
  height: 0.266667vw;
  color: #999;
}
.shoplist-title:before {
  margin-right: 3.466667vw;
}
.shoplist-title:after {
  margin-left: 3.466667vw;
}

.fixedview {
  width: 100%;
  position: fixed;
  top: 0px;
  z-index: 999;
}

.mint-loadmore {
  height: calc(100% - 95px);
  overflow: auto;
}
</style>

































Me.vue
<template>
  <div class="me">
    <div class="headInfo">
      <div class="head-img"></div>
      <div class="head-profile">
        <p class="user-id" v-if="userInfo">{{userInfo._id}}</p>
        <p v-else class="user-id" @click="handleLogin">登录/注册</p>
        <p class="user-phone">
          <i class="fa fa-mobile"></i>
          <span v-if="userInfo">{{encryptPhone(userInfo.phone)}}</span>
          <span v-else>登录后享受更多特权</span>
        </p>
      </div>
      <i class="fa fa-angle-right"></i>
    </div>
    <div v-if="userInfo">
      <div class="address-cell">
        <i class="fa fa-map-marker"></i>
        <div class="address-index" @click="myAddress">
          <span>我的地址</span>
          <i class="fa fa-angle-right"></i>
        </div>
      </div>
      <button @click="handleLogout" class="loginOut-btn">退出登录</button>
    </div>
  </div>
</template>

<script>
export default {
  name: "me",
  data() {
    return {
      userInfo: ""
    };
  },
  beforeRouteEnter(to, from, next) {
    next(vm => vm.getData());
  },
  methods: {
    handleLogin() {
      this.$router.push("/login");
    },
    getData() {
      const user_id = localStorage.ele_login;
      this.$axios(`/api/user/user_info/${user_id}`).then(res => {
        // console.log(res.data);
        this.userInfo = res.data;
      });
    },
    encryptPhone(phone) {
      return phone.replace(/(\d{3})\d{4}(\d{4})/, "$1****$2");
    },
    handleLogout() {
      localStorage.removeItem("ele_login");
      this.$router.push("/login");
    },
    myAddress() {
      if (this.userInfo.myAddress.length > 0) {
        this.$router.push("/myAddress");
      } else {
        this.$router.push({
          name: "addAddress",
          params: {
            title: "添加地址",
            addressInfo: {
              name: "",
              sex: "",
              phone: "",
              address: "",
              bottom: "",
              tag: ""
            }
          }
        });
      }
    }
  }
};
</script>

<style scoped>
.me {
  width: 100%;
  height: 100%;
  overflow: auto;
  box-sizing: border-box;
}
.headInfo {
  display: flex;
  background-image: linear-gradient(90deg, #0af, #0085ff);
  padding: 6.666667vw 4vw;
  color: #fff;
  align-items: center;
}
.head-img {
  width: 60px;
  height: 60px;
  border-radius: 50%;
  background-position: 0px 0px;
  background-size: 60px;
  background-image: url(https://shadow.elemecdn.com/faas/h5/static/sprite.3ffb5d8.png);
}
.head-profile {
  overflow: hidden;
  margin-left: 4.8vw;
  flex-grow: 1;
}
.head-profile .user-id {
  max-width: 40vw;
  overflow: hidden;
  white-space: nowrap;
  text-overflow: ellipsis;
  font-size: 1.3rem;
  margin-bottom: 2.133333vw;
  font-weight: 700;
  display: flex;
  align-items: center;
}
.head-profile .user-phone {
  display: flex;
  align-items: center;
  font-size: 0.8rem;
}
.user-phone > i {
  margin-right: 0.666667vw;
  font-size: 1rem;
}
.headInfo > i {
  font-size: 1.2rem;
}

.address-cell {
  margin-top: 2.666667vw;
  border-top: 1px solid #ddd;
  border-bottom: 1px solid #ddd;
  font-size: 1rem;
  line-height: 4.533333vw;
  background: #ffffff;
  display: flex;
  align-items: center;
  padding-left: 6.666667vw;
  color: #333;
}
.address-cell > i {
  font-size: 1.3rem;
  color: rgb(74, 165, 240);
  margin-right: 2.666667vw;
}
.address-index {
  display: flex;
  justify-content: space-between;
  width: 100%;
  padding: 3.733333vw 2.666667vw 3.733333vw 0;
  align-content: center;
}
.address-index > i {
  font-size: 1.3rem;
  color: #ccc;
}
.loginOut-btn {
  display: block;
  width: 100%;
  text-align: center;
  padding: 3.733333vw 0;
  margin: 5.333333vw 0;
  color: #ff5339;
  border-radius: 0.8vw;
  font-size: 1rem;
  font-weight: 700;
  background-color: #fff;
  border: 0;
}
</style>

Order.vue
<template>
  <div class="order">
    <div class="order-card-body" v-for="(order,index) in orderlist" :key="index">
      <div
        class="order-card-wrap"
        @click="$router.push({name:'orderInfo',params:order})"
        v-if="order.orderInfo"
      >
        <img :src="order.orderInfo.shopInfo.image_path" alt>
        <div class="order-card-content">
          <div class="order-card-head">
            <div class="title">
              <a>
                <span>{{order.orderInfo.shopInfo.name}}</span>
                <i class="fa fa-angle-right"></i>
              </a>
              <p>订单已完成</p>
            </div>
            <p class="date-time">{{order.date}}</p>
          </div>
          <div class="order-card-detail">
            <p class="detail">{{order.orderInfo.selectFoods[0].name}}</p>
            <p class="price">¥{{order.totalPrice}}</p>
          </div>
        </div>
      </div>
      <div class="order-card-bottom">
        <button class="cardbutton" @click="$router.push('/shop')">再来一单</button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "order",
  data() {
    return {
      orderlist: []
    };
  },
  beforeRouteEnter(to, from, next) {
    next(vm => {
      vm.getData();
    });
  },
  methods: {
    getData() {
      this.$axios(`/api/user/orders/${localStorage.ele_login}`).then(res => {
        // console.log(res.data);
        this.orderlist = res.data.orderlist;
      });
    }
  }
};
</script>

<style scoped>
.order {
  width: 100%;
  height: 100%;
  overflow: auto;
  box-sizing: border-box;
  margin-bottom: 2.666667vw;
}
.order-card-body {
  margin-top: 2.666667vw;
  background-color: #fff;
  padding: 3.733333vw 0 0 4vw;
}
.order-card-wrap {
  display: flex;
}
.order-card-wrap > img {
  height: 8.533333vw;
  border-radius: 0.533333vw;
  border: 1px solid #eee;
  width: 8.533333vw;
  margin-right: 2.666667vw;
}
.order-card-content {
  flex: 1;
}
.order-card-head {
  border-bottom: 1px solid #eee;
  padding-right: 3.466667vw;
  padding-bottom: 2.666667vw;
}
.order-card-head .title {
  display: flex;
  align-items: center;
  justify-content: space-between;
}
.order-card-head .title > a {
  font-size: 1rem;
  line-height: 1.5em;
  color: #333;
  text-decoration: none;
}
.order-card-head .title > a > span {
  display: inline-block;
  max-width: 10em;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}
.order-card-head .title > a > i {
  margin-left: 1.333333vw;
  color: #ccc;
  vertical-align: super;
}
.order-card-head .title > p {
  font-size: 0.8rem;
  text-align: right;
  color: #333;
  max-width: 14em;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}
.date-time {
  font-size: 0.6rem;
  color: #999;
}
.order-card-detail {
  display: flex;
  justify-content: space-between;
  padding: 4vw 3.466667vw 4vw 0;
  font-size: 0.8rem;
}
.order-card-detail .detail {
  color: #666;
  display: flex;
  align-items: center;
}
.order-card-detail .price {
  flex-basis: 16vw;
  text-align: right;
  color: #333;
  font-weight: 700;
}
.order-card-bottom {
  display: flex;
  border-top: 1px solid #eee;
  padding: 2.666667vw 4.266667vw;
  justify-content: flex-end;
}
.cardbutton {
  padding: 1.333333vw 2.666667vw;
  border: 1px solid #2395ff;
  border-radius: 0.533333vw;
  background-color: transparent;
  outline: none;
  font-size: 0.8rem;
  color: #2395ff;
  margin-left: 2vw;
}
</style>

















City.vue
<template>
  <div class="city">
    <div class="search_wrap">
      <div class="search">
        <i class="fa fa-search"></i>
        <input v-model="city_val" type="text" placeholder="输入城市名">
      </div>
      <button @click="$router.push({name:'address',params:{city:city}})">取消</button>
    </div>
    <div style="height:100%" v-if="searchList.length ==0">
      <div class="location">
        <Location @click="selectCity({name:city})" :address="city"/>
      </div>
      <Alphabet @selectCity="selectCity" ref="allcity" :cityInfo="cityInfo" :keys="keys"/>
    </div>
    <div class="search_list" v-else>
      <ul>
        <li @click="selectCity(item)" v-for="(item,index) in searchList" :key="index">{{item.name}}</li>
      </ul>
    </div>
  </div>
</template>

<script>
import Location from "../components/Location";
import Alphabet from "../components/Alphabet";
export default {
  name: "City",
  data() {
    return {
      city_val: "",
      cityInfo: null,
      keys: [],
      allCities: [],
      searchList: []
    };
  },
  computed: {
    city() {
      return (
        this.$store.getters.location.addressComponent.city ||
        this.$store.getters.location.addressComponent.province
      );
    }
  },
  components: {
    Location,
    Alphabet
  },
  created() {
    this.getCityInfo();
  },
  watch: {
    city_val() {
      // console.log(this.city_val);
      this.searchCity();
    }
  },
  methods: {
    getCityInfo() {
      this.$axios("/api/posts/cities")
        .then(res => {
          this.cityInfo = res.data;

          // 处理key 计算key
          this.keys = Object.keys(res.data);
          // hotcities这个key移除掉
          this.keys.pop();
          // keys 排序
          this.keys.sort();
          this.$nextTick(() => {
            this.$refs.allcity.initScroll();
          });

          // 存储所有城市, 用来搜索过滤
          this.keys.forEach(key => {
            // console.log(key);
            this.cityInfo[key].forEach(city => {
              // console.log(city);
              this.allCities.push(city);
            });
          });
        })
        .catch(err => {
          console.log(err);
        });
    },
    selectCity(city) {
      // console.log(city);
      this.$router.push({ name: "address", params: { city: city.name } });
    },
    searchCity() {
      if (!this.city_val) {
        // 如果搜索框为空, 数组置空
        this.searchList = [];
      } else {
        // 根据输入框的关键字检索城市 并存入到searchList数组中
        this.searchList = this.allCities.filter(item => {
          return item.name.indexOf(this.city_val) != -1;
        });
        // console.log(this.searchList);
      }
    }
  }
};
</script>

<style scoped>
.city {
  width: 100%;
  height: 100%;
  overflow: auto;
  box-sizing: border-box;
  padding-top: 45px;
}
.search_wrap {
  position: fixed;
  top: 0;
  height: 45px;
  width: 100%;
  background: #fff;
  box-sizing: border-box;
  padding: 3px 16px;
  display: flex;
  justify-content: space-between;
}
.search {
  background-color: #eee;
  border-radius: 10px;
  line-height: 40px;
  width: 85%;
  box-sizing: border-box;
  padding: 0 16px;
}
.search input {
  background: #eee;
  outline: none;
  border: none;
  margin-left: 5px;
}
.search_wrap button {
  outline: none;
  color: #009eef;
}

.location {
  background: #fff;
  padding: 8px 16px;
  height: 65px;
  box-sizing: border-box;
}

.search_list {
  background: #fff;
  padding: 5px 16px;
}
.search_list li {
  padding: 10px;
  border-bottom: 1px solid #eee;
}
</style>













Address.vue
<template>
  <div class="address">
    <Header :isLeft="true" title="选择收货地址"/>
    <div class="city_search">
      <div class="search">
        <span class="city" @click="$router.push('/city')">
          {{city}}
          <i class="fa fa-angle-down"></i>
        </span>
        <i class="fa fa-search"></i>
        <input type="text" v-model="search_val" placeholder="小区/写字楼/学校等">
      </div>
      <Location @click="selectAddress" :address="address"/>
    </div>
    <div class="area">
      <ul class="area_list" v-for="(item,index) in areaList" :key="index">
        <li @click="selectAddress(item)">
          <h4>{{item.name}}</h4>
          <p>{{item.district}}{{item.address}}</p>
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
import Header from "../components/Header";
import Location from "../components/Location";
export default {
  name: "Address",
  data() {
    return {
      city: "", // 当前的城市
      search_val: "",
      areaList: []
    };
  },
  computed: {
    address() {
      return this.$store.getters.location.formattedAddress;
    }
  },
  watch: {
    search_val() {
      this.searchPlace();
    }
  },
  methods: {
    searchPlace() {
      const self = this;
      // console.log(this.search_val);
      AMap.plugin("AMap.Autocomplete", function() {
        // 实例化Autocomplete
        var autoOptions = {
          //city 限定城市，默认全国
          city: self.city
        };
        var autoComplete = new AMap.Autocomplete(autoOptions);
        autoComplete.search(self.search_val, function(status, result) {
          // 搜索成功时，result即是对应的匹配数据
          // console.log(result);
          self.areaList = result.tips;
        });
      });
    },
    selectAddress(item) {
      // 设置地址
      if (item) {
        this.$store.dispatch(
          "setAddress",
          item.district + item.address + item.name
        );
      } else {
        this.$store.dispatch("setAddress", this.address);
      }

      // 跳转home
      this.$router.push("/home");
    }
  },
  components: {
    Header,
    Location
  },
  beforeRouteEnter(to, from, next) {
    // console.log(to.params.city + "test");
    next(vm => {
      vm.city = to.params.city;
    });
  }
};
</script>

<style scoped>
.address {
  width: 100%;
  height: 100%;
  overflow: auto;
  box-sizing: border-box;
  padding-top: 45px;
}

.city_search {
  background-color: #fff;
  padding: 10px 20px;
  color: #333;
}

.search {
  background-color: #eee;
  height: 40px;
  border-radius: 10px;
  box-sizing: border-box;
  line-height: 40px;
}
.search .city {
  padding: 0 10px;
}
.city i {
  margin-right: 10px;
}
.search input {
  margin-left: 5px;
  background-color: #eee;
  outline: none;
  border: none;
}

.area {
  margin-top: 16px;
  background: #fff;
}
.area li {
  border-bottom: 1px solid #eee;
  padding: 8px 16px;
  color: #aaa;
}
.area li h4 {
  font-weight: bold;
  color: #333;
  margin-bottom: 5px;
}
</style>
































Search.vue
<template>
  <div class="search">
    <Header :isLeft="true" title="搜索"/>
    <div class="search_header">
      <form class="search_wrap">
        <i class="fa fa-search"></i>
        <input type="text" v-model="key_word" placeholder="输入商家,商品信息">
        <button @click.prevent="searchHandle">搜索</button>
      </form>
    </div>
    <div class="shop" v-if="result && !showShop">
      <div class="empty_wrap" v-if="empty">
        <img src="https://fuss10.elemecdn.com/d/60/70008646170d1f654e926a2aaa3afpng.png" alt>
        <div class="empty_txt">
          <h4>附近没有搜索结果</h4>
          <span>换个关键词试试吧</span>
        </div>
      </div>
      <div v-else>
        <SearchIndex @click="$router.push('/shop')" :data="result.restaurants"/>
        <SearchIndex @click="shopItemClick" :data="result.words"/>
      </div>
    </div>
    <div class="container" v-if="showShop">
      <!-- 导航 -->
      <FilterView :filterData="filterData" @update="update"/>
      <div class="shoplist" v-infinite-scroll="loadMore" :infinite-scroll-disabled="loading">
        <IndexShop v-for="(item,index) in restaurants" :key="index" :restaurant="item.restaurant"/>
      </div>
    </div>
  </div>
</template>

<script>
import Header from "../components/Header";
import SearchIndex from "../components/SearchIndex";
import FilterView from "../components/FilterView";
import IndexShop from "../components/IndexShop";
import { InfiniteScroll } from "mint-ui";
export default {
  name: "Search",
  data() {
    return {
      key_word: "",
      result: null,
      empty: false,
      showShop: false,
      filterData: null,
      restaurants: [],
      page: 0,
      size: 7,
      loading: false,
      data: null
    };
  },
  watch: {
    key_word() {
      this.empty = false;
      this.showShop = false;
      this.keyWordChange();
    }
  },
  created() {
    this.$axios("/api/profile/filter").then(res => {
      // console.log(res.data);
      this.filterData = res.data;
    });
  },
  methods: {
    keyWordChange() {
      // console.log(this.key_word);
      this.$axios(`/api/profile/typeahead/${this.key_word}`)
        .then(res => {
          // console.log(res.data);
          this.result = res.data;
        })
        .catch(err => {
          this.result = null;
        });
    },
    searchHandle() {
      if (!this.key_word) return;
      // 搜索
      if (
        this.result &&
        (this.result.restaurants.length > 0 || this.result.words.length)
      ) {
        // console.log("有内容");
        this.shopItemClick();
      } else {
        this.empty = true;
      }
    },
    shopItemClick() {
      this.page = 0;
      this.restaurants = [];
      this.showShop = true;
    },
    update(condation) {
      // console.log(condation);
      this.data = condation;
      this.shopItemClick();
    },
    loadMore() {
      this.page++;
      this.$axios
        .post(`/api/profile/restaurants/${this.page}/${this.size}`, this.data)
        .then(res => {
          // this.restaurants = res.data;
          if (res.data.length > 0) {
            res.data.forEach(item => {
              this.restaurants.push(item);
            });
          } else {
            this.loading = true;
          }
        });
    }
  },
  components: {
    Header,
    SearchIndex,
    FilterView,
    IndexShop
  }
};
</script>

<style scoped>
.search {
  width: 100%;
  height: 100%;
  overflow: auto;
  box-sizing: border-box;
}
.search_header {
  margin-top: 45px;
  background: #fff;
  padding: 0 4.266667vw;
}
.search_header .search_wrap {
  padding: 2.933333vw 2.933333vw 2.933333vw 0;
  display: flex;
  align-items: center;
  position: relative;
}
.search_wrap .fa-search {
  width: 2.933333vw;
  height: 2.933333vw;
  color: #888;
  position: absolute;
  top: 4.6666666vw;
  left: 2.933333vw;
}
.search_wrap input {
  flex-grow: 1;
  border-radius: 0.266667vw;
  background-color: #f8f8f8;
  padding: 1.733333vw 4vw 1.733333vw 8.8vw;
  color: #666;
  outline: none;
  border: none;
}
.search_wrap button {
  outline: none;
  border: none;
  color: #333;
  font-size: 0.426667rem;
  background: #fff;
  font-weight: 700;
  margin-left: 2.933333vw;
  font-size: 14px;
}

.shop {
  width: 100%;
  height: calc(100% - 95px);
  overflow: auto;
}

.empty_wrap {
  width: 100%;
  height: 100%;
  overflow: hidden;
  background: #fff;
  display: flex;
  justify-content: center;
}
.empty_wrap img {
  width: 35vw;
  height: 35vw;
}
.empty_txt h4 {
  color: #666;
  font-size: 1rem;
  margin: 12vw 0 2vw 0;
}
.empty_txt span {
  color: #999;
  font-size: 0.8rem;
}
</style>















Comment.vue
<template>
  <div class="comment" v-if="evaluation">
    <!-- 商家评分 -->
    <section class="rating-wrap">
      <div class="rating-info">
        <h4>{{evaluation.rating.shop_score.toFixed(1)}}</h4>
        <div class="shop-score">
          <span>商家评分</span>
          <Rating :rating="evaluation.rating.shop_score"/>
        </div>
      </div>
      <div class="other-score">
        <div class="tp-score">
          <div>
            <span>味道</span>
            <p>{{evaluation.rating.taste_score.toFixed(1)}}</p>
          </div>
          <div>
            <span>包装</span>
            <p>{{evaluation.rating.package_score.toFixed(1)}}</p>
          </div>
        </div>
        <div class="rider-score">
          <span>配送</span>
          <p>{{evaluation.rating.rider_score.toFixed(1)}}</p>
        </div>
      </div>
    </section>
    <!-- 评论区 -->
    <div class="shop-info">
      <!-- 标签 -->
      <ul class="tags">
        <li
          :class="{'unsatisfied':item.unsatisfied}"
          v-for="(item,index) in evaluation.tags"
          :key="index"
        >
          {{item.name}}
          <span v-if="item.count > 0">{{item.count}}</span>
        </li>
      </ul>
      <!-- 内容 -->
      <ul class="comments-wrap">
        <li v-for="(item,index) in evaluation.comments" :key="index">
          <div class="comment-user">
            <img
              :src="item.avatar || 'https://shadow.elemecdn.com/faas/h5/static/sprite.3ffb5d8.png'"
              alt
            >
          </div>
          <div class="comments-info">
            <div class="comment-name">
              <h4>{{item.username}}</h4>
              <small>{{item.rated_at}}</small>
            </div>
            <div class="comment-rating">
              <Rating :rating="item.rating"/>
              <span
                :style="{color: ratingcontent(item.rating).color}"
              >{{ratingcontent(item.rating).txt}}</span>
            </div>
            <div class="comment-text">{{item.comment_text}}</div>
            <div class="comment-reply">{{item.reply.content}}</div>
            <ul class="comment-imgs">
              <li v-for="(img,i) in item.order_images" :key="i">
                <img :src="img.image_hash" alt>
              </li>
            </ul>
          </div>
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
import Rating from "../../components/Rating";
export default {
  name: "Comments",
  data() {
    return {
      evaluation: null
    };
  },
  created() {
    this.getData();
  },
  methods: {
    getData() {
      this.$axios("/api/profile/comments").then(res => {
        console.log(res.data);
        this.evaluation = res.data;
      });
    },
    ratingcontent(rating) {
      const content = [
        { txt: "吐槽", color: "rgb(137,159,188)" },
        { txt: "较差", color: "rgb(137, 159, 188)" },
        { txt: "一般", color: "rgb(251, 154, 11)" },
        { txt: "满意", color: "rgb(251, 154, 11)" },
        { txt: "超赞", color: "rgb(255, 96, 0)" }
      ];
      return content[rating - 1];
    }
  },
  components: {
    Rating
  }
};
</script>

<style scoped>
.comment {
  height: calc(100% - 10.666667vw);
  line-height: 1.2;
}
.rating-wrap {
  display: flex;
  margin-bottom: 2.133333vw;
  padding: 5.333333vw 0 8vw 6.4vw;
  background-color: #fff;
}
.rating-info {
  display: flex;
  justify-content: space-between;
  width: 33.6vw;
  align-items: center;
  color: #666;
}
.rating-info h4 {
  align-items: center;
  font-size: 2.2rem;
  color: #ff6000;
}
.rating-info .shop-score {
  align-items: flex-start;
  display: flex;
  flex-direction: column;
  justify-content: center;
}
.other-score {
  flex: 1;
  display: flex;
  justify-content: space-between;
  align-items: center;
  color: #666;
}
.tp-score {
  display: flex;
  flex-direction: row;
  justify-content: space-between;
  flex: 1;
  padding: 0 7.2vw;
  align-items: center;
}
.tp-score > div {
  display: flex;
  flex-direction: column;
  align-items: center;
}
.tp-score > div > span,
.rider-score > span {
  font-size: 0.8rem;
  margin-bottom: 1.333333vw;
}
.tp-score > div > p,
.rider-score > p {
  font-size: 1rem;
}
.rider-score {
  width: 22.933333vw;
  border-left: 1px solid #ddd;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}
.shop-info {
  background-color: #fff;
  padding: 2.666667vw 3.2vw 0;
  font-size: 0.8rem;
}
.tags {
  padding-bottom: 2.666667vw;
  border-bottom: 1px solid #eee;
}
.tags li {
  color: #6d7885;
  background-color: #ebf5ff;
  display: inline-block;
  padding: 0 2.4vw;
  height: 7.466667vw;
  line-height: 7.466667vw;
  margin: 0.933333vw;
  font-size: 0.7rem;
  border-radius: 0.533333vw;
}

.unsatisfied {
  color: #aaa !important;
  background-color: #f5f5f5 !important;
}

.comments-wrap > li {
  padding: 4vw 0 3.2vw;
  border-bottom: 0.133333vw solid #eee;
  display: flex;
}
.comment-user {
  width: 10.666667vw;
}
.comment-user img {
  width: 30px;
  height: 30px;
  border-radius: 50%;
}
.comments-info {
  flex: 1;
  font-size: 0.9rem;
}
.comment-name {
  display: flex;
  align-items: center;
  justify-content: space-between;
}
.comment-name h4 {
  margin-top: 0;
  color: #333;
  margin-right: 1.6vw;
}
.comment-name small {
  font-size: 0.6rem;
  color: #999;
}
.comment-rating {
  display: flex;
  margin: 1.6vw 0 0.533333vw;
  align-items: center;
}
.comment-rating > span {
  font-size: 0.6rem;
  margin-left: 1.066667vw;
}
.comment-text {
  color: #333;
  word-break: break-word;
  margin: 2.133333vw 0;
}
.comment-reply {
  margin: 2.666667vw 0;
  padding: 2.666667vw;
  border-radius: 1.066667vw;
  background: #f3f3f3;
  position: relative;
  font-size: 0.8rem;
}
.comment-reply::after {
  content: " ";
  position: absolute;
  bottom: 100%;
  left: 4vw;
  width: 0;
  height: 0;
  border-style: solid;
  border-width: 0 2.133333vw 2.133333vw;
  border-color: transparent transparent #f3f3f3;
}
.comment-imgs {
  margin-top: 1.066667vw;
  margin-bottom: 3.2vw;
}
.comment-imgs > li {
  display: inline-block;
  margin: 0 0.533333vw;
}
.comment-imgs > li img {
  width: 40vw;
  height: 40vw;
}
</style>


























Food.vue
<template>
  <transition name="move">
    <div class="food" v-if="isShow">
      <div class="foodpanel-close" @click="$emit('close')">
        <img
          src="https://fuss10.elemecdn.com/8/ba/bcfa8cc62b20e044bd2ea1c1c7f3dpng.png?imageMogr/format/webp/"
          alt
        >
      </div>
      <div class="foodpanel-body">
        <div class="foodpanel-foodimg">
          <img :src="food.image_path" alt>
        </div>
        <div class="foodpanel-foodinfo">
          <h4>{{food.name}}</h4>
          <div class="foodpanel-foodsales">
            <span>月售{{food.month_sales}}</span>
            <span>好评率 {{food.satisfy_rate}}%</span>
          </div>
          <div class="foodpanel-priceLine">
            <span>¥{{food.activity.fixed_price}}</span>
            <CartControll class="cart-btn" :food="food"/>
          </div>
          <p>{{food.description}}</p>
        </div>
      </div>
    </div>
  </transition>
</template>

<script>
import CartControll from "../../components/Shops/CartControll";
export default {
  name: "Food",
  props: {
    food: Object,
    isShow: Boolean
  },
  components: {
    CartControll
  }
};
</script>

<style scoped>
.food {
  position: fixed;
  top: 0;
  left: 0;
  height: 100%;
  width: 100%;
  z-index: 100;
}
.foodpanel-close {
  width: 7.466667vw;
  height: 7.466667vw;
  border-radius: 50%;
  background-color: rgba(0, 0, 0, 0.2);
  display: flex;
  align-items: center;
  justify-content: center;
  position: absolute;
  z-index: 1;
  top: 2.133333vw;
  right: 4vw;
}
.foodpanel-close img {
  width: 5.333333vw;
  height: 5.333333vw;
}
.foodpanel-body {
  position: relative;
  background-color: #fff;
  display: flex;
  flex-direction: column;
  width: 100%;
  height: 100%;
  overflow-y: auto;
  padding-bottom: 20vw;
  box-sizing: border-box;
  overflow-x: hidden;
}
.foodpanel-foodimg {
  width: 100%;
  height: 100vw;
  display: block;
}
.foodpanel-foodimg img {
  width: 100%;
  height: 100%;
}
.foodpanel-foodinfo {
  padding: 4vw 4vw 0;
  width: 100%;
  min-height: 29.333333vw;
}
.foodpanel-foodinfo h4 {
  display: flex;
  align-items: center;
  margin-bottom: 2.4vw;
  font-size: 1.2rem;
  font-weight: 700;
  width: 74.666667vw;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}
.foodpanel-foodsales {
  font-size: 0.8rem;
  color: #666;
  line-height: 1;
  margin-bottom: 2.4vw;
}
.foodpanel-priceLine {
  display: flex;
  align-items: center;
  margin-bottom: 2.4vw;
  position: relative;
}
.foodpanel-priceLine > span {
  font-size: 1rem;
  line-height: 4.266667vw;
  color: #ff5339;
  padding-bottom: 0.933333vw;
  display: flex;
  align-items: baseline;
}
.foodpanel-priceLine .cart-btn {
  position: absolute;
  right: 8vw;
}
.foodpanel-foodinfo > p {
  font-size: 0.8rem;
  color: #666;
  line-height: 3.733333vw;
  padding-right: 8vw;
}

.move-enter-active,
.move-leave-active {
  transition: all 0.25s ease-in;
  transform: translate(0, -500px);
}

.move-enter,
.move-leave-to {
  transform: translate(0, 500px);
}
</style>
























Goods.vue
<template>
  <div class="goods" v-if="shopInfo">
    <!-- 商家推荐 -->
    <div class="recommend" v-for="(recommend, index) in shopInfo.recommend" :key="index">
      <p class="recommend-name">{{recommend.name}}</p>
      <div class="recommend-wrap">
        <ul>
          <li v-for="(item,index) in recommend.items" :key="index">
            <img :src="item.image_path" alt>
            <div class="recommend-food">
              <p class="recommend-food-name">{{item.name}}</p>
              <p class="recommend-food-zm">月售{{item.month_sales}} 好评率{{item.satisfy_rate}}</p>
            </div>
            <div class="recommend-food-price">
              <p>¥{{item.activity.fixed_price}}</p>
              <CartControll :food="item"/>
            </div>
          </li>
        </ul>
      </div>
    </div>

    <!-- 商品分类 -->
    <div class="menuview">
      <!-- 左侧分类列表 -->
      <div class="menu-wrapper" ref="menuScroll">
        <ul>
          <li
            :class="{'current':currentIndex === index}"
            @click="selectMenu(index)"
            v-for="(item,index) in shopInfo.menu"
            :key="index"
          >
            <img v-if="item.icon_url" :src="item.icon_url" alt>
            <span>{{item.name}}</span>
          </li>
        </ul>
      </div>

      <!-- 右侧商品内容 -->
      <div class="foods-wrapper" ref="foodScroll">
        <ul>
          <li class="food-list-hook" v-for="(item,index) in shopInfo.menu" :key="index">
            <!-- 内容上 -->
            <div class="category-title">
              <strong>{{item.name}}</strong>
              <span>{{item.description}}</span>
            </div>
            <!-- 内容下 -->
            <div
              @click="handleFood(food)"
              class="fooddetails"
              v-for="(food,i) in item.foods"
              :key="i"
            >
              <!-- 左 -->
              <img :src="food.image_path" alt>
              <!-- 右 -->
              <section class="fooddetails-info">
                <h4>{{food.name}}</h4>
                <p class="fooddetails-des">{{food.description}}</p>
                <p class="fooddetails-sales">月售{{food.month_sales}}份 好评率{{food.satisfy_rate}}</p>
                <div class="fooddetails-price">
                  <span class="price">¥{{food.activity.fixed_price}}</span>
                  <CartControll :food="food"/>
                </div>
              </section>
            </div>
          </li>
        </ul>
      </div>
    </div>

    <!-- 购物车 -->
    <ShopCart :shopInfo="shopInfo"/>

    <!-- 商品详情 -->
    <Food :food="selectedFood" :isShow="showFood" @close="showFood=false"/>
  </div>
</template>

<script>
import CartControll from "../../components/Shops/CartControll";
import BScroll from "better-scroll";
import ShopCart from "./ShopCart";
import Food from "./Food";
export default {
  name: "Goods",
  data() {
    return {
      shopInfo: null,
      menuScroll: {}, // 左侧滚动对象
      foodScroll: {}, // 右侧滚动对象
      scrollY: 0, // 右侧菜单当前滚动到的y值
      listHeight: [], // 12个区列表高度
      selectedFood: null,
      showFood: false
    };
  },
  created() {
    this.getData();
  },
  computed: {
    // 根据右侧滚动的位置, 确定对应的索引下标
    currentIndex() {
      for (let i = 0; i < this.listHeight.length; i++) {
        let height1 = this.listHeight[i];
        let height2 = this.listHeight[i + 1];

        // 判断是否在两个高度之间
        if (this.scrollY >= height1 && this.scrollY < height2) {
          return i;
        }
      }

      return 0;
    }
  },
  methods: {
    getData() {
      this.$axios("/api/profile/batch_shop").then(res => {
        // console.log(res.data);
        res.data.recommend.forEach(recommend => {
          recommend.items.forEach(item => {
            item.count = 0;
          });
        });

        res.data.menu.forEach(menu => {
          menu.foods.forEach(food => {
            food.count = 0;
          });
        });

        this.shopInfo = res.data;
        // console.log(this.shopInfo);
        this.$nextTick(() => {
          // DOM已经更新
          this.initScroll();
          // 计算12个区的高度
          this.calculateHeight();
        });
      });
    },
    initScroll() {
      this.menuScroll = new BScroll(this.$refs.menuScroll, {
        click: true
      });

      this.foodScroll = new BScroll(this.$refs.foodScroll, {
        probeType: 3,
        click: true
      });

      this.foodScroll.on("scroll", pos => {
        // console.log(pos.y);
        this.scrollY = Math.abs(Math.round(pos.y));
      });
    },
    selectMenu(index) {
      // console.log(index);
      let foodlist = this.$refs.foodScroll.getElementsByClassName(
        "food-list-hook"
      );

      let el = foodlist[index];
      // console.log(el);
      // 滚动到对应元素的位置
      this.foodScroll.scrollToElement(el, 250);
    },
    calculateHeight() {
      let foodlist = this.$refs.foodScroll.getElementsByClassName(
        "food-list-hook"
      );
      // 每个区的高度添加到数组中
      let height = 0;
      this.listHeight.push(height);

      for (let i = 0; i < foodlist.length - 1; i++) {
        let item = foodlist[i];
        // 累加
        height += item.clientHeight;
        this.listHeight.push(height);
      }
      // console.log(this.listHeight);
    },
    handleFood(food) {
      console.log(food);
      this.selectedFood = food;
      this.showFood = true;
    }
  },
  components: {
    CartControll,
    ShopCart,
    Food
  }
};
</script>

<style scoped>
.goods {
  position: relative;
  height: calc(100% - 10.666667vw);
}

.recommend {
  padding-top: 4.266667vw;
  background-color: #fff;
}
.recommend-name {
  padding-left: 4.266667vw;
  color: #333;
  font-size: 1rem;
  font-weight: 700;
  margin-bottom: 2.666667vw;
}
.recommend-wrap {
  overflow-x: scroll;
  display: flex;
  width: 100%;
}
.recommend-wrap ul {
  display: flex;
  padding-left: 4.266667vw;
}
.recommend-wrap ul li {
  flex: none;
  width: 32vw;
  margin-right: 2.666667vw;
}
.recommend-wrap li img {
  display: block;
  width: 32vw;
  height: 32vw;
  border-top-left-radius: 0.8vw;
  border-top-right-radius: 0.8vw;
  max-width: 100%;
}
.recommend-food .recommend-food-name {
  color: #333;
  font-size: 0.8rem;
  margin: 1.866667vw 0 1.233333vw;
  white-space: nowrap;
  text-overflow: ellipsis;
  overflow: hidden;
}
.recommend-food .recommend-food-zm {
  color: #999;
  font-size: 0.6rem;
  margin-bottom: 1.866667vw;
  min-height: 1em;
}
.recommend-food-price {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding-right: 0.266667vw;
}
.recommend-food-price p {
  font-size: 1rem;
  color: #ff5339;
}

::-webkit-scrollbar {
  width: 0 !important;
}

.menuview {
  box-sizing: border-box;
  height: 100%;
  padding-bottom: 10.8vw;
  background-color: #fff;
  display: flex;
}
.menu-wrapper {
  overflow-y: hidden;
  /* height: 100%; */
  height: calc(100% - 12.8vw);
  background-color: #f8f8f8;
  padding-bottom: 10.666667vw;
  width: 20.533333vw;
}
.menu-wrapper li {
  padding: 4.666667vw 2vw;
  font-size: 0.6rem;
  color: #666;
  line-height: 1.2;
}
.menu-wrapper li img {
  max-width: 100%;
  width: 3.466667vw;
  height: 3.466667vw;
  vertical-align: top;
  margin-right: 0.8vw;
}

.foods-wrapper {
  overflow-y: hidden;
  /* height: 100%; */
  height: calc(100% - 12.8vw);
  width: 79.466667vw;
  padding-bottom: 10.666667vw;
}
.category-title {
  margin-left: 2.666667vw;
  padding: 2vw 8vw 2vw 0;
  display: flex;
  align-items: center;
  overflow: hidden;
}
.category-title strong {
  margin-right: 1.333333vw;
  font-weight: 700;
  font-size: 0.8rem;
  color: #666;
  flex: none;
}
.category-title span {
  flex: 1;
  color: #999;
  font-size: 0.6rem;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}
.fooddetails {
  min-height: 30.666667vw;
  padding: 2.666667vw 0 2.666667vw 2.666667vw;
  margin-bottom: 0.133333vw;
  display: flex;
}
.fooddetails img {
  width: 25.333333vw;
  height: 25.333333vw;
  flex: none;
  margin-right: 2.666667vw;
  border-radius: 0.533333vw;
}
.fooddetails-info {
  flex: 1;
  padding-bottom: 6.666667vw;
  padding-right: 4vw;
}
.fooddetails-info h4 {
  padding-right: 4vw;
  font-weight: 700;
  overflow: hidden;
  font-size: 1rem;
  white-space: nowrap;
  width: 40vw;
  text-overflow: ellipsis;
  color: #333;
}
.fooddetails-des {
  margin: 1.333333vw 0;
  font-size: 0.6rem;
  color: #999;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  width: 42.666667vw;
}
.fooddetails-sales {
  margin: 1.733333vw 0 !important;
  color: #999;
  font-size: 0.6rem;
  line-height: 1;
  min-height: 1em;
}
.fooddetails-price {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-top: 3.733333vw;
}
.fooddetails-price .price {
  font-size: 1rem;
  line-height: 4.266667vw;
  color: #ff5339;
  padding-bottom: 0.933333vw;
  display: flex;
  align-items: baseline;
}

.menu-wrapper .current {
  background-color: #fff !important;
  color: #333 !important;
}
</style>


Seller.vue
<template>
  <div class="seller" v-if="sellerInfo">
    <section>
      <img :src="sellerInfo.header_image" alt>
      <h3>{{sellerInfo.title}}</h3>
      <p>{{sellerInfo.brand_intros[0].brief}}</p>
      <div>查看品牌故事</div>
    </section>
  </div>
</template>

<script>
export default {
  name: "Seller",
  data() {
    return {
      sellerInfo: null
    };
  },
  created() {
    this.getData();
  },
  methods: {
    getData() {
      this.$axios("/api/profile/seller").then(res => {
        // console.log(res.data);
        this.sellerInfo = res.data;
      });
    }
  }
};
</script>

<style scoped>
.seller section {
  margin-bottom: 2.666667vw;
  padding: 4.266667vw 4vw 4vw;
  font-size: 0.9rem;
  background-color: #fff;
  color: #666;
  border-bottom: 1px solid #eee;
}
section > img {
  width: 92vw;
  height: 52vw;
  margin-bottom: 4.266667vw;
}
section > h3 {
  color: #333;
  font-weight: 700;
  font-size: 1rem;
  margin-bottom: 1.066667vw;
}
section > p {
  height: 40px;
  line-height: 1.4;
  overflow: hidden;
  text-overflow: ellipsis;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
}
section > div {
  margin: 4vw 0 -4vw;
  text-align: center;
  font-size: 0.346667rem;
  padding: 4vw 0;
}
</style>

Shop.vue
<template>
  <div class="shop" v-if="shopInfo">
    <!-- 头部 -->
    <nav class="header-nav">
      <div class="nav_bg">
        <img :src="shopInfo.rst.scheme" alt>
      </div>
      <div class="nav_back">
        <i @click="$router.push('/home')" class="fa fa-chevron-left"></i>
      </div>
      <div class="shop_image">
        <img :src="shopInfo.rst.image_path" alt>
      </div>
    </nav>

    <!-- 商家信息 -->
    <div class="index-rst">
      <div class="rst-name">
        <span @click="showInfoModel = true">{{shopInfo.rst.name}}</span>
        <i class="fa fa-caret-right"></i>
      </div>
      <!-- 弹窗信息 -->
      <InfoModel @close="showInfoModel = false" :rst="shopInfo.rst" :showInfoModel="showInfoModel"/>

      <!-- 评分月售 -->
      <div class="rst-order">
        <span>评分{{shopInfo.rst.rating}}</span>
        <span>月售{{shopInfo.rst.recent_order_num}}单</span>
        <span>蜂鸟专送约{{shopInfo.rst.order_lead_time}}分钟</span>
      </div>
      <!-- 优惠信息 -->
      <Activity :activities="shopInfo.rst.activities"/>

      <!-- 公告 -->
      <p class="rst-promotion">公告: {{shopInfo.rst.promotion_info}}</p>
    </div>

    <!-- 导航 -->
    <NavBar/>
    <keep-alive>
      <router-view></router-view>
    </keep-alive>
  </div>
</template>

<script>
import InfoModel from "../../components/Shops/InfoModel";
import Activity from "../../components/Shops/Activity";
import NavBar from "../../components/Shops/NavBar";

export default {
  name: "Shop",
  data() {
    return {
      shopInfo: null,
      showInfoModel: false
    };
  },
  created() {
    this.getData();
  },
  methods: {
    getData() {
      this.$axios("/api/profile/batch_shop").then(res => {
        // console.log(res.data);
        this.shopInfo = res.data;
      });
    }
  },
  components: {
    InfoModel,
    Activity,
    NavBar
  }
};
</script>

<style scoped>
.shop {
  width: 100%;
  height: 100%;
  overflow: auto;
  box-sizing: border-box;
}
.header-nav {
  position: relative;
}
.nav_bg img {
  width: 100%;
  height: 26.666667vw;
}
.nav_back {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 26.666667vw;
  background: rgba(0, 0, 0, 0.5);
}
.nav_back i {
  color: #fff;
  font-size: 1.3rem;
  margin-left: 1.333333vw;
  margin-top: 1.333333vw;
}
.shop_image {
  position: absolute;
  top: 0;
  left: 50%;
  margin-left: -10vw;
  margin-top: 11vw;
}
.shop_image img {
  width: 20vw;
  height: 20vw;
  border-radius: 0.8vw;
}

.index-rst {
  padding: 8vw 0 0;
  display: flex;
  flex-direction: column;
  align-items: center;
  background: #fff;
  box-shadow: inset 0 -0.666667vw 0.666667vw hsla;
}
.index-rst .rst-name {
  flex: 1;
  width: 72vw;
  font-size: 1.3rem;
  font-weight: 700;
  white-space: nowrap;
  padding-right: 2.666667vw;
  display: flex;
  align-items: center;
  justify-content: center;
  margin: 1.6vw 0;
}
.rst-name span {
  text-align: left;
  overflow: hidden;
  text-overflow: ellipsis;
}

.index-rst .rst-order {
  white-space: nowrap;
  height: 3.2vw;
  margin-top: 1.733333vw;
  color: #666;
  text-align: center;
  font-size: 0.6rem;
}
.rst-order span {
  margin: 0 3px;
}
.index-rst .rst-promotion {
  width: 80vw;
  font-size: 0.6rem;
  color: #999;
  overflow: hidden;
  text-overflow: ellipsis;
  margin: 2.266667vw auto 2.666667vw;
  padding: 0;
  white-space: nowrap;
}
</style>

























ShopCar.vue
<template>
  <div class="shopcart">
    <transition name="fade">
      <div
        class="cartview-cartmask"
        @click.self="showCartView=false"
        v-if="showCartView && !isEmpty"
      >
        <div class="cartview-cartbody">
          <div class="cartview-cartheader">
            <span>已选商品</span>
            <button @click="clearFoods">
              <i class="fa fa-trash-o"></i>
              <span>清空</span>
            </button>
          </div>
          <div class="entityList-cartbodyScroller">
            <ul class="entityList-cartlist">
              <li class="entityList-entityrow" v-for="(food,index) in selectFoods" :key="index">
                <h4>
                  <span>{{food.name}}</span>
                </h4>
                <span class="entityList-entitytotal">{{food.activity.fixed_price}}</span>
                <CartControll :food="food"/>
              </li>
            </ul>
          </div>
        </div>
      </div>
    </transition>
    <div class="bottomNav-cartfooter" :class="{'bottomNav-carticon-empty':isEmpty}">
      <span class="bottomNav-carticon" @click="showCartView = !showCartView">
        <i class="fa fa-cart-plus"></i>
        <span v-if="totalCount">{{totalCount}}</span>
      </span>
      <div class="bottomNav-cartInfo" @click="showCartView = !showCartView">
        <p class="bottomNav-carttotal">
          <span v-if="isEmpty">未选购商品</span>
          <span v-else>¥{{totalPrice.toFixed(2)}}</span>
        </p>
        <p class="bottomNav-cartdelivery">另需配送费{{shopInfo.rst.float_delivery_fee}}元</p>
      </div>
      <button class="submit-btn">
        <span v-if="isEmpty">¥{{shopInfo.rst.float_minimum_order_amount}}元起送</span>
        <span @click="settlement" v-else>去结算</span>
      </button>
    </div>
  </div>
</template>

<script>
import CartControll from "../../components/Shops/CartControll";
export default {
  name: "ShopCart",
  data() {
    return {
      totalCount: 0,
      totalPrice: 0,
      selectFoods: [],
      showCartView: false
    };
  },
  props: {
    shopInfo: Object
  },
  computed: {
    isEmpty() {
      let empty = true;
      this.totalCount = 0;
      this.totalPrice = 0;
      this.selectFoods = [];
      this.shopInfo.recommend.forEach(recommend => {
        recommend.items.forEach(item => {
          if (item.count) {
            empty = false;
            this.totalCount += item.count;
            this.totalPrice += item.activity.fixed_price * item.count;
            this.selectFoods.push(item);
          }
        });
      });

      this.shopInfo.menu.forEach(menu => {
        menu.foods.forEach(food => {
          if (food.count) {
            empty = false;
            this.totalCount += food.count;
            this.totalPrice += food.activity.fixed_price * food.count;
            this.selectFoods.push(food);
          }
        });
      });
      return empty;
    }
  },
  methods: {
    clearFoods() {
      this.shopInfo.recommend.forEach(recommend => {
        recommend.items.forEach(item => {
          item.count = 0;
        });
      });

      this.shopInfo.menu.forEach(menu => {
        menu.foods.forEach(item => {
          item.count = 0;
        });
      });
    },
    settlement() {
      this.$store.dispatch("setOrderInfo", {
        shopInfo: this.shopInfo.rst,
        selectFoods: this.selectFoods
      });
      this.$router.push("/settlement");
    }
  },
  components: {
    CartControll
  }
};
</script>

<style scoped>
.bottomNav-cartfooter {
  position: fixed;
  right: 0;
  bottom: 0;
  left: 0;
  display: flex;
  align-items: center;
  padding-left: 21.066667vw;
  background-color: rgba(61, 61, 63, 0.9);
  height: 12.8vw;
  z-index: 1000;
}

.bottomNav-carticon {
  position: absolute;
  left: 3.2vw;
  bottom: 2vw;
  width: 13.333333vw;
  height: 13.333333vw;
  box-sizing: border-box;
  border-radius: 100%;
  background-color: #3190e8;
  border: 1.333333vw solid #444;
  box-shadow: 0 -0.8vw 0.533333vw 0 rgba(0, 0, 0, 0.1);
}
.bottomNav-carticon > i {
  position: absolute;
  top: 7px;
  right: 0;
  bottom: 0;
  left: 7px;
  color: #fff;
  font-size: 1.6rem;
}
.bottomNav-cartInfo {
  flex: 1;
}
.bottomNav-carttotal {
  font-size: 0.8rem;
  line-height: normal;
  color: #fff;
}
.bottomNav-cartdelivery {
  color: #999;
  font-size: 0.6rem;
}
.submit-btn {
  height: 100%;
  width: 28vw;
  background-color: #38ca73;
  color: #fff;
  text-align: center;
  text-decoration: none;
  font-size: 0.9rem;
  font-weight: 600;
  line-height: 12.8vw;
  border: none;
  outline: none;
}

.bottomNav-carticon-empty > span {
  background-image: radial-gradient(circle, #363636 6.266667vw, #444 0);
}
.bottomNav-carticon-empty > span > i {
  color: #606065 !important;
}
.bottomNav-carticon-empty .bottomNav-carttotal > span {
  color: #999;
}
.bottomNav-carticon-empty .submit-btn {
  background-color: #535356 !important;
}

.bottomNav-carticon > span {
  position: absolute;
  right: -1.2vw;
  top: -1.333333vw;
  line-height: 1;
  background-image: linear-gradient(-90deg, #ff7416, #ff3c15 98%);
  color: #fff;
  border-radius: 3.2vw;
  padding: 0.833333vw 1.333333vw;
  font-size: 0.6rem;
}

.cartview-cartmask {
  position: fixed;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
  background-color: rgba(0, 0, 0, 0.4);
  z-index: 200;
}
.cartview-cartbody {
  position: fixed;
  left: 0;
  width: 100%;
  background-color: #fff;
  bottom: 12.8vw;
  z-index: 201;
  opacity: 1;
  font-size: 1rem;
}
.cartview-cartheader {
  display: flex;
  align-items: center;
  padding: 0 4vw;
  border-bottom: 0.133333vw solid #ddd;
  background-color: #eceff1;
  color: #666;
  height: 10.666667vw;
}
.cartview-cartheader > span {
  flex: 1;
  display: flex;
  align-items: center;
}
.cartview-cartheader > button {
  border: none;
  outline: none;
  flex: none;
  display: flex;
  align-items: center;
  padding-left: 4vw;
  color: #666;
  text-decoration: none;
  font-size: 0.8rem;
  line-height: 4vw;
  background: none;
}
.cartview-cartheader > button > span {
  margin-left: 0.8vw;
}
.entityList-cartbodyScroller {
  overflow: auto;
  max-height: 80vw;
}
.entityList-entityrow {
  border-bottom: 0.133333vw solid #eee;
  display: flex;
  align-items: center;
  padding: 2vw 3.333333vw 2vw 0;
  min-height: 12.666667vw;
  margin-left: 3.333333vw;
}
.entityList-entityrow > h4 {
  flex: 5.5;
  line-height: normal;
}
.entityList-entityrow > h4 > span {
  display: inline-block;
  font-style: normal;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  vertical-align: middle;
  max-width: 46.666667vw;
}
.entityList-entitytotal {
  color: rgb(255, 83, 57);
  flex: 2.5;
  text-align: left;
  white-space: nowrap;
  font-weight: 700;
}
.entityList-entitytotal::before {
  content: "\A5";
  font-size: 0.6rem;
  color: currentColor;
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.25s ease-out;
}

.fade-enter,
.fade-leave-to {
  opacity: 0;
}</style>
AddAddress.vue
<template>
  <div class="addAddress">
    <Header :isLeft="true" :title="title"/>
    <!-- 添加地址 -->
    <div class="viewbody">
      <div class="content">
        <FormBlock
          label="联系人"
          placeholder="姓名"
          :tags="sexes"
          :sex="addressInfo.sex"
          @checkSex="checkSex"
          v-model="addressInfo.name"
        />
        <FormBlock v-model="addressInfo.phone" label="电话" placeholder="手机号码"/>
        <FormBlock
          v-model="addressInfo.address"
          @click="showSearch=true"
          label="地址"
          placeholder="小区/写字楼/学校等"
          icon="angle-right"
        />
        <FormBlock
          v-model="addressInfo.bottom"
          label="门牌号"
          placeholder="10号楼5单元404"
          icon="edit"
          textarea="textarea"
        />
        <div class="formblock">
          <div class="label-wrap">标签</div>
          <TabTag :tags="tags" :selectTag="addressInfo.tag" @checkTag="checkTag"/>
        </div>
      </div>
      <!-- 确定按钮 -->
      <div class="form-button-wrap">
        <button @click="handleSave" class="form-button">确定</button>
      </div>
    </div>
    <!-- 搜索地址 -->
    <AddressSearch @close="showSearch=false" :showSearch="showSearch" :addressInfo="addressInfo"/>
  </div>
</template>

<script>
import Header from "../../components/Header";
import FormBlock from "../../components/Orders/FormBlock";
import TabTag from "../../components/Orders/TabTag";
import AddressSearch from "../../components/Orders/AddressSearch";
import { Toast } from "mint-ui";
export default {
  name: "AddAddress",
  data() {
    return {
      title: "",
      tags: ["家", "学校", "公司"],
      sexes: ["先生", "女士"],
      addressInfo: {},
      showSearch: false
    };
  },
  beforeRouteEnter(to, from, next) {
    next(vm => {
      vm.addressInfo = to.params.addressInfo;
      vm.title = to.params.title;
    });
  },
  methods: {
    checkTag(item) {
      // console.log(item);
      this.addressInfo.tag = item;
    },
    checkSex(item) {
      // console.log(item);
      this.addressInfo.sex = item;
    },
    handleSave() {
      // console.log(this.addressInfo);
      if (!this.addressInfo.name) {
        this.showMsg("请输入联系人");
        return;
      }

      if (!this.addressInfo.phone) {
        this.showMsg("请输入手机号");
        return;
      }

      if (!this.addressInfo.address) {
        this.showMsg("请输入收货地址");
        return;
      }

      // 存储数据
      if (this.title == "添加地址") {
        this.addAddress();
      } else {
        this.editAddress();
      }
    },
    showMsg(msg) {
      Toast({
        message: msg,
        position: "bottom",
        duration: 2000
      });
    },
    addAddress() {
      this.$axios
        .post(
          `/api/user/add_address/${localStorage.ele_login}`,
          this.addressInfo
        )
        .then(res => {
          if (!this.$store.getters.userInfo) {
            this.$store.dispatch("setUserInfo", this.addressInfo);
          }
          this.$router.push("myAddress");
        })
        .catch(err => console.log(err));
    },
    editAddress() {
      this.$axios
        .post(
          `/api/user/edit_address/${localStorage.ele_login}/${
            this.addressInfo._id
          }`,
          this.addressInfo
        )
        .then(res => {
          this.$router.push("/myAddress");
        });
    }
  },
  components: {
    Header,
    FormBlock,
    TabTag,
    AddressSearch
  }
};
</script>

<style scoped>
.addAddress {
  width: 100%;
  height: 100%;
  box-sizing: border-box;
  padding-top: 45px;
}
.viewbody {
  width: 100%;
  height: 100%;
  overflow: auto;
  box-sizing: border-box;
  background-color: #f5f5f5;
}

.content {
  padding-left: 4vw;
  background: #fff;
  font-size: 0.94rem;
}

.formblock {
  background-color: #fff;
  border-bottom: 1px solid #eee;
  display: flex;
}
.formblock .label-wrap {
  flex-basis: 17.333333vw;
  padding: 3.733333vw 0;
  line-height: 4.8vw;
  color: #333;
  font-weight: 700;
}

/* 确定按钮 */
.form-button-wrap {
  padding: 5.333333vw 4vw;
  display: flex;
}
.form-button-wrap .form-button {
  background: #00d762;
  text-align: center;
  border-radius: 0.533333vw;
  flex: 1;
  font-size: 1.1rem;
  line-height: 5.066667vw;
  color: #fff;
  padding: 3.333333vw 0;
  border: none;
  font-weight: 500;
}
</style>























MyAddress.vue
<template>
  <div class="myAddress">
    <Header :isLeft="true" :title="title"/>

    <!-- 显示收货地址 -->
    <div class="address-view">
      <div class="address-card" v-for="(address,index) in allAddress" :key="index">
        <div class="address-card-select">
          <i class="fa fa-check-circle" v-if="selectIndex == index"></i>
        </div>

        <div class="address-card-body" @click="setAddressInfo(address,index)">
          <p class="address-card-title">
            <span class="username">{{address.name}}</span>
            <span v-if="address.sex" class="gender">{{address.sex}}</span>
            <span class="phone">{{address.phone}}</span>
          </p>
          <p class="address-card-address">
            <span class="tag" v-if="address.tag">{{address.tag}}</span>
            <span class="address-text">{{address.address}}</span>
          </p>
        </div>
        <div class="address-card-edit">
          <i @click="handleEdit(address)" class="fa fa-edit"></i>
          <i @click="handleDelete(address,index)" class="fa fa-close"></i>
        </div>
      </div>
    </div>

    <!-- 新增收货地址 -->
    <div class="address-view-bottom" @click="addAddress">
      <i class="fa fa-plus-circle"></i>
      <span>新增收货地址</span>
    </div>
  </div>
</template>

<script>
import Header from "../../components/Header";
export default {
  name: "MyAddress",
  data() {
    return {
      title: "我的地址",
      allAddress: [],
      selectIndex: 0
    };
  },
  beforeRouteEnter(to, from, next) {
    next(vm => vm.getData());
  },
  methods: {
    addAddress() {
      this.$router.push({
        name: "addAddress",
        params: {
          title: "添加地址",
          addressInfo: {
            name: "",
            sex: "",
            phone: "",
            address: "",
            bottom: "",
            tag: ""
          }
        }
      });
    },
    getData() {
      this.$axios(`/api/user/user_info/${localStorage.ele_login}`).then(res => {
        // console.log(res.data);
        this.allAddress = res.data.myAddress;
      });
    },
    handleEdit(address) {
      this.$router.push({
        name: "addAddress",
        params: {
          title: "编辑地址",
          addressInfo: address
        }
      });
    },
    handleDelete(address, index) {
      this.$axios
        .delete(`/api/user/address/${localStorage.ele_login}/${address._id}`)
        .then(res => {
          this.allAddress.splice(index, 1);
        });
    },
    setAddressInfo(address, index) {
      this.selectIndex = index;
      // 将address对象存储到vuex
      this.$store.dispatch("setUserInfo", address);
      this.$router.push("/settlement");
    }
  },
  components: {
    Header
  }
};
</script>

<style scoped>
.myAddress {
  width: 100%;
  height: 100%;
  box-sizing: border-box;
  padding-top: 45px;
}

.address-view-bottom {
  position: fixed;
  height: 13.866667vw;
  bottom: 0;
  width: 100%;
  background: #fff;
  display: flex;
  justify-content: center;
  align-items: center;
  border-top: 0.266667vw solid #ddd;
  color: #3190e8;
  font-size: 1rem;
}
.address-view-bottom > i {
  font-size: 1.3rem;
  margin-right: 8px;
}

.address-view {
  height: 100%;
  overflow-y: auto;
  padding-bottom: 13.866667vw;
}
.address-card {
  background-color: #fff;
  padding: 4vw;
  border-bottom: 1px solid #ddd;
  display: flex;
  min-height: 18.133333vw;
}
.address-card-body {
  flex: 1;
  overflow: hidden;
}
.address-card-title {
  font-size: 1rem;
  color: #666;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  padding-bottom: 1.066667vw;
}
.address-card-title .username {
  color: #333;
  font-weight: 700;
}
.address-card-title .gender {
  padding: 0 1.6vw 0 0.8vw;
}
.address-card-address {
  color: #666;
  font-size: 0.84rem;
  display: flex;
  align-items: center;
}
.address-card-address .tag {
  display: inline-block;
  position: relative;
  margin-right: 0.8vw;
  padding: 0.533333vw;
  color: #ff5722;
  white-space: nowrap;
  border: 1px solid #ff5722;
  border-radius: 0.133333vw;
  font-size: 0.6rem !important;
  line-height: 2.666667vw;
}
.address-text {
  line-height: 4.533333vw;
}

/* 编辑和删除 */
.address-card-edit {
  flex-basis: 13.066667vw;
  display: flex;
  justify-content: space-around;
  align-items: center;
}
.address-card-edit > i {
  color: #aaa;
  font-size: 1.2rem;
}

/*  选中图标 */
.address-card-select {
  flex-basis: 7.333333vw;
  min-width: 7.333333vw;
  display: flex;
  align-items: center;
}
.address-card-select > i {
  font-size: 1.5rem;
  color: #4cd964;
}
</style>













OrderInfo.vue
<template>
  <div class="orderInfo">
    <Header title="订单详情" :isLeft="true"/>
    <div class="view-body" v-if="orderDetail">
      <div class="status-head">
        <div class="status-text">订单已送达</div>
        <div class="status-title">感谢您对南工的信任, 期待再次光临</div>
      </div>
      <div class="restaurant-card">
        <!-- 点餐信息 -->
        <CartGroup
          v-if="orderDetail.orderInfo"
          :orderInfo="orderDetail.orderInfo"
          :totalPrice="orderDetail.totalPrice"
        />
      </div>

      <!-- 配送信息 -->
      <div class="detail-card">
        <div class="title">配送信息</div>
        <ul class="card-list">
          <li class="list-item">
            <span>送达时间:</span>
            <div>{{orderDetail.date}}</div>
          </li>
          <li class="list-item">
            <span>送货地址:</span>
            <div class="content">
              <span>{{orderDetail.userInfo.name}}{{orderDetail.userInfo.sex}}</span>
              <span>{{orderDetail.userInfo.phone}}</span>
              <span>{{orderDetail.userInfo.address}}{{orderDetail.userInfo.bottom}}</span>
            </div>
          </li>
        </ul>
      </div>

      <!-- 订单信息 -->
      <div class="detail-card">
        <div class="title">订单信息</div>
        <ul class="card-list">
          <li class="list-item">
            <span>订单号:</span>
            {{orderDetail._id}}
          </li>
          <li class="list-item">
            <span>支付方式:</span>
            在线支付
          </li>
          <li class="list-item">
            <span>下单时间:</span>
            {{orderDetail.date}}
          </li>
          <li class="list-item" v-if="orderDetail.remarkInfo">
            <span>订单备注:</span>
            <span class="remark">
              {{orderDetail.remarkInfo.remark}} \
              {{orderDetail.remarkInfo.tableware}}
            </span>
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script>
import Header from "../../components/Header";
import CartGroup from "../../components/Orders/CartGroup";
export default {
  data() {
    return {
      orderDetail: null
    };
  },
  beforeRouteEnter(to, from, next) {
    next(vm => {
      vm.orderDetail = to.params;
      // console.log(vm.orderDetail);
    });
  },
  components: {
    Header,
    CartGroup
  }
};
</script>

<style scoped>
.orderInfo {
  width: 100%;
  height: 100%;
  box-sizing: border-box;
  padding-top: 45px;
}

.view-body {
  width: 100%;
  height: 100%;
  overflow: auto;
  box-sizing: border-box;
}
.status-head {
  margin: 2.666667vw;
  box-shadow: 0 0.133333vw 0.266667vw 0 rgba(0, 0, 0, 0.05);
  background-color: #fff;
  padding: 0 3.2vw 5.333333vw;
}
.status-head .status-text {
  margin: 0 auto 4.266667vw;
  color: #333;
  font-size: 1.5rem;
  text-align: left;
  padding: 4.266667vw 0;
  border-bottom: 0.133333vw solid #ddd;
}
.status-head .status-title {
  font-size: 1rem;
  color: #333;
  margin-bottom: 1.866667vw;
}

.restaurant-card {
  margin: 2.666667vw;
  box-shadow: 0 0.133333vw 0.266667vw 0 rgba(0, 0, 0, 0.05);
  background-color: #fff;
  padding: 0 3.2vw 5.333333vw;
}

/* 配送和订单 */
.detail-card {
  margin: 2.666667vw;
  box-shadow: 0 0.133333vw 0.266667vw 0 rgba(0, 0, 0, 0.05);
  background-color: #fff;
  padding: 0 3.2vw 5.333333vw;
}

.status-head .status-text {
  margin: 0 auto 4.266667vw;
  color: #333;
  font-size: 1.5rem;
  text-align: left;
  padding: 4.266667vw 0;
  border-bottom: 0.133333vw solid #ddd;
}
.status-head .status-title {
  font-size: 1rem;
  color: #333;
  margin-bottom: 1.866667vw;
}
.title {
  font-size: 1rem;
  color: #333;
  border-bottom: 1px solid #eee;
  line-height: 10.4vw;
}
.list-item {
  color: #6e6e6e;
  border-bottom: 1px solid #f2f2f2;
  display: flex;
  align-items: baseline;
  line-height: 4.8vw;
  font-size: 0.88rem;
  padding: 3.2vw 8vw 2.666667vw 0;
}
.list-item .content {
  line-height: 1.5em;
  padding-bottom: 2.666667vw;
  display: flex;
  flex-direction: column;
  flex: 1;
}
.list-item span:first-child {
  flex: none;
}
.remark {
  text-overflow: ellipsis;
  overflow: hidden;
  flex-grow: 1;
  white-space: nowrap;
}
</style>

Pay.vue
<template>
  <div class="pay">
    <Header title="在线支付"/>
    <div class="main" v-if="orderInfo">
      <div class="tip">
        <p class="countdown-title">支付剩余时间</p>
        <h3 class="countdown-text">{{countDown}}</h3>
      </div>
      <section class="home">
        <ul class="list info-list">
          <li>
            <span class="order-name">{{orderInfo.shopInfo.name}}</span>
            <span class="text-highlight">¥{{totalPrice}}</span>
          </li>
        </ul>
        <div class="title">在线支付</div>
        <ul class="list payment-list">
          <li class="payment-list-item">
            <div class="icon">
              <img src="../../assets/wechart.jpg" alt>
              <span>微信</span>
            </div>
            <i class="fa fa-check-circle selected"></i>
          </li>
        </ul>
      </section>
      <button @click="pay" :disabled="timeOut" class="btn-submit">确认支付</button>
    </div>
  </div>
</template>

<script>
import Header from "../../components/Header";
import { setInterval, clearInterval } from "timers";
import { MessageBox } from 'mint-ui';
export default {
  name: "Pay",
  data() {
    return {
      countDown: "00:15:00",
      timer: null,
      timeOut: false
    };
  },
  beforeRouteEnter(to, from, next) {
    next(vm => {
      vm.countTimeDown();
      vm.addOrder();
    });
  },
  computed: {
    orderInfo() {
      return this.$store.getters.orderInfo;
    },
    totalPrice() {
      return this.$store.getters.totalPrice;
    },
    userInfo() {
      return this.$store.getters.userInfo;
    },
    remarkInfo() {
      return this.$store.getters.remarkInfo;
    }
  },
  methods: {
    countTimeDown() {
      let minute = 14;
      let second = 59;

      this.timer = setInterval(() => {
        second--;

        if (second == "00" && minute == "00") {
          this.countDown = "订单已超时";
          clearInterval(this.timer);
          this.timeOut = true;
          return;
        }

        if (second == "00") {
          second = 59;
          minute--;

          if (minute < 10) {
            minute = "0" + minute;
          }
        }

        if (second < 10) {
          second = "0" + second;
        }

        this.countDown = "00:" + minute + ":" + second;
      }, 1000);
    },
    pay() {
      //前端传给后端的数据
      const data = {
        body: "南工",
        out_trade_no: new Date().getTime().toString(),  //订单号
        total_fee: 1
      };
      alert("进入到pay方法中");
      // 请求 http://www.thenewstep.cn/wxzf/example/jsapi.php
      //微信支付接口域名必须备案
      fetch("http://www.thenewstep.cn/wxzf/example/jsapi.php", {
        method: "POST",
        headers: {
          "Content-type": "application/json"
        },
        body: JSON.stringify(data)
      })
        .then(res => res.json())
        .then(data => {
          //data是所有的微信参数
          this.onBridgeReady(data);
        })
        .catch(err => {
          alert("请求失败");
        });
    },
    onBridgeReady(data) {
      WeixinJSBridge.invoke("getBrandWCPayRequest", data, res => {
        if (res.err_msg == "get_brand_wcpay_request:ok") {
          // 使用以上方式判断前端返回,微信团队郑重提示：
          //res.err_msg将在用户支付成功后返回ok，但并不保证它绝对可靠。
          // alert("支付成功");
          // 生成订单
          // this.addOrder();
        }else {
          alert("支付成功");
        }
      });
    },
    addOrder() {
      let orderlist = {
        orderInfo: this.orderInfo,
        userInfo: this.userInfo,
        totalPrice: this.totalPrice,
        remarkInfo: this.remarkInfo
      };
      console.log(orderlist);
      this.$axios
        .post(`/api/user/add_order/${localStorage.ele_login}`, orderlist)
        .then(res => {
          console.log(res.data);
          MessageBox.prompt('请输入密码').then(({ value, action }) => {
            console.log(value);
            if(value == 188188){
              alert("支付成功");
              this.$router.push("/order");
            }else {
               alert("密码错误，请重新输入");
               this.addOrder();
            }
          });
          
        });
    }
  },
  components: {
    Header
  }
};
</script>

<style scoped>
.pay {
  width: 100%;
  height: 100%;
  box-sizing: border-box;
  padding-top: 45px;
}

.main {
  width: 100%;
  height: 100%;
  overflow: auto;
  box-sizing: border-box;
}
.tip {
  display: block;
  text-align: center;
  background-color: #fff;
  color: #333;
  padding: 2rem;
  line-height: 2rem;
}
countdown-title {
  font-size: 0.88rem;
  color: #999;
}
.countdown-text {
  height: 2rem;
  color: #333;
  font-size: 1.8rem;
}
down-text {
  height: 2rem;
  color: #333;
  font-size: 1.8rem;
}
.list {
  border-bottom: 0.5px solid #eee;
  background: #fff;
}
.info-list {
  padding: 0 1.5rem;
}
.info-list li {
  border-top: 0.5px solid #eee;
  display: flex;
  padding: 1.5rem 0;
  font-size: 1rem;
  line-height: 1rem;
  justify-content: space-between;
}
.info-list li .order-name {
  color: #333;
  display: inline-block;
  vertical-align: bottom;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  margin-right: 16px;
  max-width: 60%;
}
.text-highlight {
  color: #ff6000;
}
.title {
  background-color: #f5f5f5;
  font-size: 0.88rem;
  padding: 1.2rem 1.2rem 1rem;
  color: #999;
  display: flex;
  justify-content: space-between;
  align-items: center;
}
.payment-list-item {
  border-bottom: 0.5px solid #eee;
  padding: 0.9rem 1rem;
  font-size: 1.1rem;
  display: flex;
  align-items: center;
  justify-content: space-between;
}
.icon {
  display: flex;
  align-items: center;
}
.payment-list-item img {
  width: 2.4rem;
  height: 2.4rem;
  margin-right: 20px;
}
.payment-list-item > i {
  font-size: 1.5rem;
  color: #eee;
}
.selected {
  color: #4cd964 !important;
}
.btn-submit-wrap {
  margin: 1rem auto;
  width: 90%;
}
.btn-submit {
  font-size: 1.1rem;
  line-height: 1.5rem;
  background-color: #4cd964;
  width: 100%;
  outline: none;
  border: none;
  color: #fff;
  border-radius: 5px;
  padding: 0.5rem;
  box-sizing: border-box;
}

/* 不可点击btn */
.btn-submit[disabled] {
  background-color: #bbb !important;
}
</style>

Remark.vue
<template>
  <div class="remark">
    <Header :isLeft="true" title="订单备注"/>
    <!-- 订单备注 -->
    <div class="view-body">
      <section>
        <textarea placeholder="填写额外对餐厅或骑手小哥备注的信息" v-model="text" maxlength="100"></textarea>
        <div class="switch">
          <span
            @click="handleRadioItem(item)"
            :class="{'selected': item.select}"
            v-for="(item,index) in radioItem"
            :key="index"
            class="switch-item item-line"
          >{{item.name}}</span>
        </div>
        <div class="switch" v-for="(item,index) in switchItem" :key="index">
          <span
            :class="{'selected': item.select}"
            @click="item.select = !item.select"
            class="switch-item"
          >{{item.name}}</span>
        </div>
      </section>
      <button @click="submitClick" class="btn-submit">确定</button>
    </div>
  </div>
</template>

<script>
import Header from "../../components/Header";
export default {
  name: "Remark",
  data() {
    return {
      radioItem: [
        { select: false, name: "不要辣" },
        { select: false, name: "少点辣" },
        { select: false, name: "多点辣" }
      ],
      switchItem: [
        { select: false, name: "不要香菜" },
        { select: false, name: "不要洋葱" },
        { select: false, name: "多点醋" },
        { select: false, name: "多点葱" }
      ],
      text: ""
    };
  },
  methods: {
    handleRadioItem(item) {
      this.radioItem.forEach(element => {
        element.select = false;
      });
      item.select = true;
    },
    submitClick() {
      let selectItems = "";
      this.radioItem.forEach(element => {
        if (element.select) {
          selectItems += element.name + ",";
        }
      });

      this.switchItem.forEach(element => {
        if (element.select) {
          selectItems += element.name + ",";
        }
      });

      selectItems += this.text;
      // console.log(selectItems);
      // 存储
      this.$store.dispatch("setRemarkInfo", {
        tableware: this.$store.getters.remarkInfo.tableware,
        remark: selectItems
      });

      this.$router.go(-1);
    }
  },
  components: {
    Header
  }
};
</script>

<style scoped>
.remark {
  width: 100%;
  height: 100%;
  box-sizing: border-box;
  padding-top: 45px;
}

.view-body {
  width: 100%;
  height: 100%;
  overflow: auto;
  box-sizing: border-box;
  font-size: 0.8rem;
  color: #333;
}
.view-body section {
  margin-top: 2.133333vw;
  padding: 4.266667vw;
  background-color: #fff;
  margin-bottom: 2.133333vw;
}
.view-body section textarea {
  width: 100%;
  height: 29.866667vw;
  margin-bottom: 4.266667vw;
  padding: 4.266667vw;
  border: 1px solid #ccc;
  border-radius: 0.666667vw;
  background-color: #f9f9f9;
  color: #666;
  resize: none;
  box-sizing: border-box;
  outline: none;
}
.switch {
  display: inline-block;
  margin: 0 2.666667vw 2.666667vw 0;
  border: 1px solid #ddd;
  overflow: hidden;
  border-radius: 1.066667vw;
}
.switch-item {
  display: inline-block;
  padding: 0 2.133333vw;
  height: 8vw;
  line-height: 8vw;
  text-align: center;
  color: #666;
}
.item-line::after {
  content: " ";
  display: inline-block;
  height: 4vw;
  width: 1px;
  background: #ddd;
  line-height: 8vw;
  vertical-align: middle;
  left: 2.266667vw;
  position: relative;
}
.btn-submit {
  display: block;
  padding: 3.466667vw 0;
  color: #fff;
  background-color: #00e067;
  border-radius: 0.533333vw;
  opacity: 0.98;
  width: 92vw;
  margin: 3.133333vw auto 0;
  font-size: 1rem;
}

/* 选中样式 */
.switch-item.selected {
  background: #0186ff;
  color: #fff;
  position: relative;
}
</style>

Selllement.vue
<template>
  <div class="settlement">
    <Header :isLeft="true" title="确认订单"/>
    <div class="view-body" v-if="orderInfo">
      <div class>
        <!-- 收货地址 -->
        <section class="cart-address">
          <p class="title">
            订单配送至
            <span class="address-tag" v-if="userInfo && userInfo.tag">{{userInfo.tag}}</span>
          </p>
          <p class="address-detail">
            <span
              @click="$router.push('/myAddress')"
              v-if="userInfo"
            >{{userInfo.address}}{{userInfo.bottom}}</span>
            <span v-else>
              <span v-if="haveAddress" @click="$router.push('/myAddress')">选择收货地址</span>
              <span v-else @click="addAddress">新增收货地址</span>
            </span>
            <i class="fa fa-angle-right"></i>
          </p>
          <h2 v-if="userInfo" class="address-name">
            <span>{{userInfo.name}}</span>
            <span v-if="userInfo.sex">({{userInfo.sex}})</span>
            <span class="phone">{{userInfo.phone}}</span>
          </h2>
        </section>

        <!-- 送达时间 -->
        <Delivery :shopInfo="orderInfo.shopInfo"/>

        <!-- 点餐内容 -->
        <CartGroup :orderInfo="orderInfo" :totalPrice="totalPrice"/>

        <!-- 备注信息 -->
        <section class="checkout-section">
          <CartItem
            @click="showTableware=true"
            title="餐具份数"
            :subHead="remarkInfo.tableware || '未选择'"
          />
          <CartItem
            @click="$router.push('/remark')"
            title="订单备注"
            :subHead="remarkInfo.remark || '口味 偏好'"
          />
          <CartItem title="发票信息" subHead="不需要开发票"/>
        </section>

        <!-- 显示Tableware -->
        <Tableware :isShow="showTableware" @close="showTableware=false"/>
      </div>
    </div>
    <!-- 底部 -->
    <footer class="action-bar">
      <span>¥{{totalPrice}}</span>
      <button @click="handlePay()">去支付</button>
    </footer>
  </div>
</template>

<script>
import Header from "../../components/Header";
import Delivery from "../../components/Orders/Delivery";
import CartGroup from "../../components/Orders/CartGroup";
import CartItem from "../../components/Orders/CartItem";
import Tableware from "../../components/Orders/Tableware";
import { Toast } from "mint-ui";
export default {
  name: "Settlement",
  data() {
    return {
      haveAddress: false,
      showTableware: false
    };
  },
  computed: {
    userInfo() {
      return this.$store.getters.userInfo;
    },
    orderInfo() {
      return this.$store.getters.orderInfo;
    },
    totalPrice() {
      return this.$store.getters.totalPrice;
    },
    remarkInfo() {
      return this.$store.getters.remarkInfo;
    }
  },
  beforeRouteEnter(to, from, next) {
    next(vm => {
      if (!vm.userInfo) {
        vm.getData();
      }
    });
  },
  methods: {
    addAddress() {
      this.$router.push({
        name: "addAddress",
        params: {
          title: "添加地址",
          addressInfo: {
            name: "",
            sex: "",
            phone: "",
            address: "",
            bottom: "",
            tag: ""
          }
        }
      });
    },
    getData() {
      this.$axios(`/api/user/user_info/${localStorage.ele_login}`).then(res => {
        // console.log(res.data);
        if (res.data.myAddress.length > 0) {
          this.haveAddress = true;
        } else {
          this.haveAddress = false;
        }
      });
    },
    handlePay() {
      if (!this.userInfo) {
        Toast({
          message: "请选择收货地址",
          position: "bottom",
          duration: 2000
        });
        return;
      }
      this.$router.push("/pay");
    }
  },
  components: {
    Header,
    Delivery,
    CartGroup,
    CartItem,
    Tableware
  }
};
</script>

<style scoped>
.settlement {
  width: 100%;
  height: 100%;
  box-sizing: border-box;
  padding-top: 44px;
}

.view-body {
  width: 100%;
  height: 100%;
  overflow: auto;
  box-sizing: border-box;
  font-size: 0.9rem;
  color: #333;
  padding-bottom: 14.133333vw;
  padding-left: 1.6vw;
  padding-right: 1.6vw;
  background-image: linear-gradient(
      0deg,
      #f5f5f5,
      #f5f5f5 65%,
      hsla(0, 0%, 96%, 0.3) 75%,
      hsla(0, 0%, 96%, 0)
    ),
    linear-gradient(270deg, #009eef, #009eef);
  display: flex;
  flex-direction: column;
  flex-grow: 1;
}
.cart-address {
  background-color: transparent;
  padding: 4.266667vw 2.133333vw 2.933333vw 2.133333vw;
  min-height: 22.133333vw;
  color: #fff;
  overflow: hidden;
}
.cart-address .title {
  font-size: 0.9rem;
  line-height: 1.21;
  color: hsla(0, 0%, 100%, 0.8);
}
.cart-address .address-detail {
  font-size: 1.3rem;
  font-weight: 600;
  line-height: 1.88;
  overflow: hidden;
  display: flex;
  align-items: center;
}
.address-detail > span {
  display: inline-block;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  max-width: calc(100% - 4vw);
}
.address-detail > i {
  margin-left: 2.133333vw;
}

/* 显示送货地址 */
.address-name {
  font-size: 0.86rem;
  line-height: 1.21;
  margin-bottom: 1.333333vw;
}
.address-name .phone {
  margin-left: 2.133333vw;
}
.address-tag {
  border: 0.133334vw solid hsla(0, 0%, 100%, 0.8);
  margin-left: 1.6vw;
  display: inline-block;
  padding: 0.533333vw;
  white-space: nowrap;
  border-radius: 0.133333vw;
  font-size: 0.6rem !important;
  line-height: 2.666667vw;
}

.checkout-section {
  margin-bottom: 2.133333vw;
  padding: 0 5.333333vw;
  background: #fff;
  box-shadow: 0 0.133333vw 0.266667vw 0 rgba(0, 0, 0, 0.05);
}

/* 底部支付样式 */
.action-bar {
  height: 11.733333vw;
  position: fixed;
  left: 0;
  bottom: 0;
  width: 100%;
  background: #3c3c3c;
  z-index: 2;
}
.action-bar > span {
  color: #fff;
  font-size: 1.2rem;
  line-height: 11.733333vw;
  padding-left: 2.666667vw;
  vertical-align: middle;
}
.action-bar > button {
  position: absolute;
  top: 0;
  right: 0;
  bottom: 0;
  background: #00e067;
  min-width: 28vw;
  padding: 0 1.333333vw;
  border: none;
  color: #fff;
  font-size: 1.2rem;
}
</styl 

