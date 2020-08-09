<template>
  <div class="city">
    <div class="search_wrap">
      <div class="search">
        <i class="fa fa-search"></i>
        <input v-model="city_val" type="text" placeholder="输入城市名" />
      </div>
      <button @click="$router.go(-1)">取消</button>
    </div>
    <div style="height:100%" v-if="searchList.length == 0">
      <div class="location">
        <Location @click="selectcity({name:city})" :address="city"></Location>
      </div>
      <Alphabet @selectcity="selectcity" ref="allcity" :cityInfo="cityInfo" :keys="keys"></Alphabet>
    </div>
    <div class="search_list" v-else>
      <ul>
        <li @click="selectcity(item)" v-for="(item,index) in searchList" :key="index">{{item.name}}</li>
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
      keys: [], //A-Z以及hotcities
      allCities: [],
      searchList: [],
    };
  },
  computed: {
    city() {
      return (
        this.$store.getters.location.addressComponent.city ||
        this.$store.getters.location.addressComponent.province
      );
    },
  },
  created() {
    this.getCityInfo();
  },
  watch: {
    city_val() {
      this.searchCity();
    },
  },
  methods: {
    getCityInfo() {
      this.$axios("/api/posts/cities")
        .then((res) => {
          console.log(111111);
          this.cityInfo = res.data;
          //处理key  计算key
          this.keys = Object.keys(res.data);
          //移除hotcities这个key
          this.keys.pop();
          //keys 排序
          this.keys.sort();
          this.$nextTick(() => {
            this.$refs.allcity.initScroll();
          });
          //存储所有城市,用来搜索过滤
          this.keys.forEach((key) => {
            this.cityInfo[key].forEach((city) => {
              this.allCities.push(city);
            });
          });
        })
        .catch((err) => {
          console.log(err);
        });
    },
    selectcity(city) {
      this.$router.push({ name: "address", params: { city: city.name } });
    },
    searchCity() {
      if (!this.city_val) {
        //搜索框为空 数组置空
        this.searchList = [];
      } else {
        //根据输入框关键字检索城市并存到数组中
        this.searchList = this.allCities.filter((item) => {
          return item.name.indexOf(this.city_val) != -1;
        });
      }
    },
  },
  components: {
    Location,
    Alphabet,
  },
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
  background-color: #fff;
  border: none;
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
