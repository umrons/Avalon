<style lang="scss">
  .sub-comment-list-wrap {
    .view-more {
      font-size: 12px;
      color: $color-text-normal;

      .more-btn {
        color: $color-blue-normal;
      }

      .collapse-btn {
        float: right;
        color: $color-text-light;

        &:hover {
          color: $color-blue-normal;
        }
      }
    }
  }
</style>

<template>
  <div class="sub-comment-list-wrap">
    <sub-comment-item
      v-for="comment in filterComments"
      :key="comment.id"
      :comment="comment"
      :parent-user-id="authorId"
      :type="type"
    ></sub-comment-item>
    <div class="view-more" v-if="!comments.noMore || comments.list.length > 5">
      共<strong>{{ comments.total }}</strong>条回复
      <template v-if="!comments.noMore">
        ，<button class="more-btn" @click="loadMore">查看更多</button>
      </template>
      <button
        class="collapse-btn"
        v-if="showCollapse"
        @click="collapsed = !collapsed"
      >{{ collapsed ? '展开' : '收起' }}</button>
    </div>
  </div>
</template>

<script>
  import SubCommentItem from './SubCommentItem.vue'

  export default {
    name: 'v-sub-comment-list',
    props: {
      parentComment: {
        required: true,
        type: Object
      },
      type: [String]
    },
    components: {
      SubCommentItem
    },
    computed: {
      comments () {
        return this.parentComment.comments
      },
      authorId () {
        return this.parentComment.from_user_id
      },
      filterComments () {
        if (!this.collapsed) {
          return this.comments.list
        }
        const data = this.comments
        const comments = data.list
        const result = comments.slice(0, 5)
        if (comments.every(_ => _.id <= data.maxId)) {
          return result
        }
        const ids = result.map(_ => _.id)
        return result.concat(
          comments.filter(_ => _.id > data.maxId && ids.indexOf(_.id) === -1)
        )
      }
    },
    data () {
      return {
        loading: false,
        showCollapse: false,
        collapsed: false
      }
    },
    methods: {
      async loadMore () {
        if (this.loading) {
          return
        }
        this.loading = true
        try {
          await this.$store.dispatch('comment/getSubComments', {
            ctx: this,
            type: this.type,
            parentId: this.parentComment.id
          })
          this.showCollapse = true
        } catch (e) {
          this.$toast.error(e)
        } finally {
          this.loading = false
        }
      }
    }
  }
</script>
