<template>
  <div class="m-art-rewards">
    <header class="m-pos-r" style="padding-top: 0.9rem">
      <div class="m-pos-f m-box m-justify-bet m-aln-center m-head-top m-bb1 m-main">
        <div class="m-box m-flex-grow1 m-aln-center m-flex-base0">
          <svg class='m-style-svg m-svg-def' @click='goBack'>
            <use xmlns:xlink="http://www.w3.org/1999/xlink" xlink:href="#base-back"></use>
          </svg>
        </div>
        <div class="m-box-model m-flex-grow1 m-aln-center m-flex-base0 m-head-top-title">打赏列表</div>
        <div class="m-box m-flex-grow1 m-aln-center m-flex-base0 m-justify-end"></div>
      </div>
    </header>
    <main>
      <jo-load-more ref="loadmore" :autoLoad="false" @onRefresh="onRefresh" @onLoadMore="onLoadMore">
        <section
          class="m-box m-aln-center m-justify-bet m-art-reward-item m-bb1 m-main"
          v-for="({ user, id, created_at }, index) in rewards"
          :key="`reward-${id}-${index}`">

          <avatar :user="user" size="tiny" />

          <h2 class="m-box m-flex-grow1 m-flex-shrink1 m-text-cut"><b>{{ user.name }}</b>打赏了{{ typeMap[type] }}</h2>

          <time class="m-flex-grow0 m-flex-shrink0" :datetime="created_at">{{ created_at | time2tips }}</time>
        </section>
      </jo-load-more>
    </main>
  </div>
</template>
<script>
export default {
  name: "rewards",
  data() {
    return {
      rewards: [],
      typeMap: {
        feed: "动态",
        news: "资讯",
        post: "帖子"
      }
    };
  },
  computed: {
    type() {
      return this.$route.meta.type;
    },
    article() {
      return this.$route.params.article;
    },
    url() {
      // 动态 GET /feeds/{feed}/rewards
      // 资讯 GET /news/{news}/rewards
      // 帖子 GET /plus-group/group-posts/:post/rewards

      switch (this.type) {
        case "feed":
          return `/feeds/${this.article}/rewards`;
        case "news":
          return `/news/${this.article}/rewards`;
        case "post":
          return `/plus-group/group-posts/${this.article}/rewards`;
      }
    }
  },
  methods: {
    onRefresh(callback) {
      /**
       * 刷新列表
       *
       *    名称          类型         说明
            limit         integer     默认 15 ，数据返回条数 默认为15
            offset        integer     默认 0 ，翻页标识。
            order         string      默认 desc, asc 升序 desc 降序
            order_type    string      默认 date, amount 打赏金额 date 打赏时间
       */
      this.$http
        .get(this.url, {
          params: {
            limit: 15
          }
        })
        .then(({ data = [] }) => {
          this.rewards = data;
          callback(data.length < 15);
        })
        .catch(e => {
          console.log(e);
          callback(true);
        });
    },
    onLoadMore(callback) {
      this.$http
        .get(this.url, {
          params: {
            limit: 15,
            offset: this.rewards.length
          }
        })
        .then(({ data = [] }) => {
          this.rewards = [...this.rewards, ...data];
          callback(data.length < 15);
        })
        .catch(e => {
          console.log(e);
          callback(true);
        });
    }
  },

  mounted() {
    this.$refs.loadmore.beforeRefresh();
  }
};
</script>

<style lang="less">
.m-art-reward-item {
  padding: 30px 20px;
  color: #999;
  time {
    font-size: 24px;
    color: #ccc;
  }
  h2 {
    font-size: 30px;
    margin-left: 30px;
    margin-right: 30px;
    b {
      color: #000;
      margin-right: 10px;
    }
  }
}
</style>
