<template>
  <div class="home-page">

    <div class="banner">
      <div class="container">
        <h1 class="logo-font">拉勾教育</h1>
        <p>A place to share your knowledge.</p>
      </div>
    </div>

    <div class="container page">
      <div class="row">
        <div class="col-md-9">
          <div class="feed-toggle">
            <ul class="nav nav-pills outline-active">
              <li class="nav-item"
                  v-if="user">
                <nuxt-link class="nav-link"
                           exact
                           :class="{'active':tab==='your_feed'}"
                           :to="{name:'home',query:{tab:'your_feed'}}">Your Feed</nuxt-link>
              </li>
              <li class="nav-item">
                <nuxt-link class="nav-link"
                           exact
                           :class="{'active':tab==='global_feed'}"
                           :to="{name:'home',query:{tab:'global_feed'}}">Global Feed</nuxt-link>
              </li>
              <li class="nav-item"
                  v-if="tag">
                <nuxt-link class="nav-link"
                           :class="{'active':tab==='tag'}"
                           :to="{name:'home',query:{tab:'tag'}}"># {{ tag }} </nuxt-link>
              </li>
            </ul>
          </div>
          <articles-list :articles="articles"></articles-list>

          <!-- 分页 -->
          <nav>
            <ul class="pagination">
              <li class="page-item"
                  :class="{'active':item==page}"
                  v-for="item in totalPage"
                  :key="item">
                <nuxt-link class="page-link"
                           :to="{name:'home',query:{page:item,tag:$route.query.tag,tab:tab}}">{{item}}</nuxt-link>
              </li>
            </ul>
          </nav>
          <!-- /分页 -->

        </div>

        <div class="col-md-3">
          <div class="sidebar">
            <p>Popular Tags</p>

            <div class="tag-list">
              <nuxt-link :to="{name:'home',query:{tag:item,tab:'tag'}}"
                         v-for="item in tags"
                         :key="item"
                         class="tag-pill tag-default">{{item}}</nuxt-link>
            </div>
          </div>
        </div>

      </div>
    </div>

  </div>
</template>

<script>
import articlesList from "../../components/articles-list.vue";
import { api } from "/plugins/request.js";
export default {
  components: { articlesList },
  name: "home",
  asyncData({
    isDev,
    route,
    store,
    env,
    params,
    query,
    req,
    res,
    redirect,
    error,
  }) {},
  async asyncData({ store, query }) {
    const page = Number.parseInt(query.page) || 1;
    const limit = 20;
    const tag = query.tag;
    const tab = query.tab || "global_feed";
    const loadArticles =
      store.state.user && tab === "your_feed"
        ? api("articles/feed")
        : api("articles/getArticles", {
            limit,
            offset: (page - 1) * limit,
            tag,
          });
    const [articleRes, tagRes] = await Promise.all([
      loadArticles,
      api("tags/getTags"),
    ]);
    const { articles, articlesCount } = articleRes.data;
    const { tags } = tagRes.data;
    articles.forEach((item) => {
      item.favoriteDisable = false;
    });
    return {
      limit,
      page,
      tags,
      tag,
      articles,
      articlesCount,
      tab,
    };
  },
  watchQuery: ["page", "tag", "tab"],
  computed: {
    totalPage() {
      return Math.ceil(this.articlesCount / this.limit);
    },
  },
  methods: {},
};
</script>

<style scoped lang="less">
</style>
