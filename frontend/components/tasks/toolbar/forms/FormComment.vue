<template>
  <base-card
    :title="$t('comments.comments')"
    :cancel-text="$t('generic.close')"
    @cancel="$emit('click:cancel')"
  >
    <template v-if="user.id" #content>
      <form-create @add-comment="add" />
      <comment
        v-for="comment in comments"
        :key="comment.id"
        :comment="comment"
        :user-id="user.id"
        @delete-comment="remove"
        @update-comment="maybeUpdate"
      />
    </template>
  </base-card>
</template>

<script lang="ts">
import Vue from 'vue'
import Comment from '@/components/comment/Comment.vue'
import FormCreate from '@/components/comment/FormCreate.vue'
import BaseCard from '@/components/utils/BaseCard.vue'
import { UserItem } from '~/domain/models/user/user'
import { CommentReadDTO } from '~/services/application/comment/commentData'

export default Vue.extend({
  components: {
    BaseCard,
    Comment,
    FormCreate
  },

  props: {
    exampleId: {
      type: Number,
      required: true
    }
  },

  data() {
    return {
      user: {} as UserItem,
      comments: [] as CommentReadDTO[]
    }
  },

  watch: {
    exampleId: {
      handler(val) {
        if (val !== undefined) {
          this.list()
        }
      },
      immediate: true,
      deep: true
    }
  },

  async created() {
    this.user = await this.$repositories.user.getProfile()
  },

  methods: {
    async list() {
      this.comments = await this.$services.comment.list(this.$route.params.id, this.exampleId)
    },
    async add(message: string) {
      await this.$services.comment.create(this.$route.params.id, this.exampleId, message)
      this.list()
    },
    async remove(item: CommentReadDTO) {
      await this.$services.comment.delete(this.$route.params.id, item)
      this.list()
    },
    async maybeUpdate(item: CommentReadDTO) {
      await this.$services.comment.update(this.$route.params.id, item)
      this.list()
    }
  }
})
</script>
