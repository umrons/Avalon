<style lang="scss">
  .create-comment-form {
    margin-top: 10px;
    margin-bottom: 10px;

    .avatar {
      float: left;
      margin: 7px 32px 0 5px;
      @include avatar-2(48px);
    }

    .main-area {
      overflow: hidden;

      .text-wrap {
        overflow: hidden;

        textarea {
          font-size: 12px;
          background-color: #f4f5f7;
          border: 1px solid $color-gray-normal;
          border-radius: 4px;
          color: #555;
          width: 100%;
          min-height: 65px;
          padding: 5px 10px;
          resize: vertical;

          &:hover {
            background-color: #fff;
            border-color: $color-blue-normal;
          }
        }
      }

      .submit-btn {
        float: right;
        width: 70px;
        height: 64px;
        background-color: $color-blue-normal;
        transition: .1s;
        text-align: center;
        font-size: 14px;
        border-radius: 4px;
        margin-left: 10px;
        color: #fff;
        font-weight: bold;

        &:hover {
          background-color: $color-blue-light;
        }
      }
    }
  }
</style>

<template>
  <div class="create-comment-form">
    <div class="avatar">
      <img :src="$resize(userAvatar, { width: 96 })">
    </div>
    <div class="main-area">
      <button class="submit-btn" @click="submit">发表<br>评论</button>
      <div class="text-wrap">
      <textarea
        placeholder="请自觉遵守互联网相关的政策法规，严禁发布色情、暴力、反动的言论。"
        v-model.trim="content"
        maxlength="1000"
        @focus="handleTextAreaFocus"
      ></textarea>
      </div>
    </div>
  </div>
</template>

<script>
  export default {
    name: 'create-comment-form',
    props: {
      type: {
        type: String,
        required: true
      },
      id: {
        type: [Number, String],
        required: true
      }
    },
    computed: {
      isGuest () {
        return !this.$store.state.login
      },
      formatContent () {
        let content = this.content
        while (content.match('\n\n\n') !== null) {
          content = content.replace(/\n\n\n/g, '\n\n')
        }
        content = content.split('\n')

        const res = []
        content.forEach(item => {
          res.push(item ? `<p>${item}</p>` : '<p><br/></p>')
        })

        return res.join('')
      },
      submitting () {
        return this.$store.state.comment.submitting
      },
      userAvatar () {
        return this.isGuest
          ? `${this.$cdn.image}default/user-avatar`
          : this.$store.state.user.avatar
      }
    },
    data () {
      return {
        content: ''
      }
    },
    methods: {
      async submit () {
        if (this.isGuest) {
          this.$toast.info('继续操作前请先登录')
          this.$channel.$emit('sign-in')
          return
        }
        if (!this.content) {
          return
        }
        if (this.submitting) {
          return
        }
        this.$store.commit('comment/SET_SUBMITTING', { result: true })
        try {
          const newComment = await this.$store.dispatch('comment/createMainComment', {
            content: this.formatContent,
            images: [],
            type: this.type,
            id: this.id,
            ctx: this
          })
          this.content = ''
          if (this.withImage) {
            this.$refs.uploader.clearFiles()
          }
          this.$toast.success('评论成功')
          setTimeout(() => {
            const dom = document.getElementById(`comment-${newComment.id}`)
            dom && this.$scrollToY(this.$utils.getOffsetTop(dom) - 200, 600)
          }, 400)
        } catch (e) {
          this.$toast.error(e)
        } finally {
          this.$store.commit('comment/SET_SUBMITTING', { result: false })
        }
      },
      handleTextAreaFocus () {
        if (this.isGuest) {
          this.$channel.$emit('sign-in')
        }
      }
    }
  }
</script>
