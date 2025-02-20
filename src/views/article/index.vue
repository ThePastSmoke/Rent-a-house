<template>
  <div class="article-container">
    <!-- 导航栏 -->
    <van-nav-bar class="page-nav-bar" left-arrow title="黑马头条"></van-nav-bar>
    <!-- /导航栏 -->

    <div class="main-wrap">
      <!-- 加载中 -->
      <div class="loading-wrap" v-if="loading">
        <van-loading color="#3296fa" vertical>加载中</van-loading>
      </div>
      <!-- /加载中 -->

      <!-- 加载完成-文章详情 -->
      <div class="article-detail" v-else-if="article.art_id">
        <!-- 文章标题 -->
        <h1 class="article-title">{{ article.title }}</h1>
        <!-- /文章标题 -->

        <!-- 用户信息 -->
        <van-cell class="user-info" center :border="false">
          <van-image
            class="avatar"
            slot="icon"
            round
            fit="cover"
            :src="article.aut_photo"
          />
          <div slot="title" class="user-name">{{ article.aut_name }}</div>
          <div slot="label" class="publish-date">{{ article.pubdate }}</div>
        </van-cell>
        <!-- /用户信息 -->
        <FollowUser
          v-model="article.is_followed"
          :user_id="article.aut_id"
        ></FollowUser>
        <!-- 文章内容 -->
        <div
          ref="content"
          class="article-content markdown-body"
          v-html="article.content"
        ></div>
        <van-divider>正文结束</van-divider>
      </div>
      <!-- /加载完成-文章详情 -->

      <!-- 加载失败：404 -->
      <div class="error-wrap" v-else-if="isNotFound">
        <van-icon name="failure" />
        <p class="text">该资源不存在或已删除！</p>
      </div>
      <!-- /加载失败：404 -->

      <!-- 加载失败：其它未知错误（例如网络原因或服务端异常） -->
      <div class="error-wrap" v-else>
        <van-icon name="failure" />
        <p class="text">内容加载失败！</p>
        <van-button class="retry-btn">点击重试</van-button>
      </div>
      <!-- /加载失败：其它未知错误（例如网络原因或服务端异常） -->

      <!-------------------文章列表 ---------------->
      <ArticleComment
        v-if="article.art_id"
        :articleId="article.art_id"
      ></ArticleComment>
      <!-------------------文章列表 ---------------->
    </div>

    <!-- 底部区域 -->
    <div class="article-bottom">
      <van-button
        @click="isPostShow = true"
        class="comment-btn"
        type="default"
        round
        size="small"
        >写评论</van-button
      >
      <van-icon name="comment-o" :badge="article.comm_count" color="#777" />
      <!-- 收藏组件 -->
      <collectArticle
        :art_id="article.art_id"
        v-model="article.is_collected"
      ></collectArticle>
      <!-- 点赞组件 -->
      <likeArticle
        :articleId="article.art_id"
        v-model="article.attitude"
      ></likeArticle>
      <!-- <van-icon color="#777" name="good-job-o" /> -->
      <van-icon name="share" color="#777777"></van-icon>
    </div>
    <!-- /底部区域 -->
    <!--------------------------- 写评论弹层 ------------------>
    <van-popup position="bottom" v-model="isPostShow">
      <!-- 写评论弹出层内组件 -->
      <PostComment :target="article.art_id"></PostComment>
    </van-popup>
    <!---------------------------- 写评论弹层 --------------------->
  </div>
</template>

<script>
import { getArticleById } from "@/api";
import { ImagePreview } from "vant";
import FollowUser from "@/views/article/components/follow-user";
import collectArticle from "@/views/article/components/collectArticle";
import likeArticle from "@/views/article/components/like-article";
import PostComment from "@/views/article/components/post-comment.vue";
// 评论列表
import ArticleComment from "@/views/article/components/comment-list.vue";
import "github-markdown-css";
export default {
  name: "ArticleIndex",
  components: {
    FollowUser,
    collectArticle,
    likeArticle,
    ArticleComment,
    PostComment,
  },
  props: {
    articleId: {
      type: [Number, String],
      required: true,
    },
  },
  data() {
    return {
      article: {}, //请求回来的数据
      loading: false,
      isNotFound: false,
      isPostShow: false, // 评论弹层
    };
  },
  computed: {},
  watch: {},
  created() {
    this.getArticleById();
  },
  mounted() {},
  methods: {
    previewImg() {
      // 获取所有的img图片  push到一个数组里面
      const imgs = this.$refs.content.querySelectorAll("img");
      const images = [];
      // src属性
      imgs.forEach((item, index) => {
        //  push到一个数组里面
        images.push(item.src);
        item.addEventListener("click", function () {
          ImagePreview({
            images,
            startPosition: index,
          });
        });
      });
    },
    async getArticleById() {
      this.loading = true;
      try {
        const res = await getArticleById(this.articleId);
        this.article = res.data.data;
        // 调用获取图片src函数 $nextTick方法会在dom结构挂载结束后执行里面的代码 拿到最新的dom结构
        this.$nextTick(() => {
          this.previewImg();
        });
        this.loading = false;
      } catch (error) {
        this.loading = false;
        this.isNotFound = error.response.status === 404;
        console.log(error);
      }
    },
  },
};
</script>

<style scoped lang="less">
.article-container {
  /deep/.van-icon-arrow-left {
    color: #fff;
  }
  .main-wrap {
    position: fixed;
    left: 0;
    right: 0;
    top: 92px;
    bottom: 88px;
    overflow-y: scroll;
    background-color: #fff;
  }
  .article-detail {
    .article-title {
      font-size: 40px;
      padding: 50px 32px;
      margin: 0;
      color: #3a3a3a;
    }

    .user-info {
      padding: 0 32px;
      .avatar {
        width: 70px;
        height: 70px;
        margin-right: 17px;
      }
      .van-cell__label {
        margin-top: 0;
      }
      .user-name {
        font-size: 24px;
        color: #3a3a3a;
      }
      .publish-date {
        font-size: 23px;
        color: #b7b7b7;
      }
      .follow-btn {
        width: 170px;
        height: 58px;
      }
    }

    .article-content {
      padding: 55px 32px;
      /deep/ p {
        text-align: justify;
      }
    }
  }

  .loading-wrap {
    padding: 200px 32px;
    display: flex;
    align-items: center;
    justify-content: center;
    background-color: #fff;
  }

  .error-wrap {
    padding: 200px 32px;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    background-color: #fff;
    .van-icon {
      font-size: 122px;
      color: #b4b4b4;
    }
    .text {
      font-size: 30px;
      color: #666666;
      margin: 33px 0 46px;
    }
    .retry-btn {
      width: 280px;
      height: 70px;
      line-height: 70px;
      border: 1px solid #c3c3c3;
      font-size: 30px;
      color: #666666;
    }
  }

  .article-bottom {
    position: fixed;
    left: 0;
    right: 0;
    bottom: 0;
    display: flex;
    justify-content: space-around;
    align-items: center;
    box-sizing: border-box;
    height: 88px;
    border-top: 1px solid #d8d8d8;
    background-color: #fff;
    .comment-btn {
      width: 282px;
      height: 46px;
      border: 2px solid #eeeeee;
      font-size: 30px;
      line-height: 46px;
      color: #a7a7a7;
    }
    .van-icon {
      font-size: 40px;
      .van-info {
        font-size: 16px;
        background-color: #e22829;
      }
    }
  }
}
</style>
