<template>
  <div id="interactive-comments">
    <div class="comments-content" v-for="comment in comments" :key="comment.id">
      <div class="current-user-comment">
        <div class="score">
          <button @click="vote(comment, 1)"><i class="plus"></i></button>
          <p>{{ comment.score }}</p>
          <button @click="vote(comment, -1)"><i class="minus"></i></button>
        </div>
        <div class="content">
          <div class="top-content">
            <div class="user-inf">
              <img :src="getUserImage(comment.user.image)" alt="User Avatar" v-if="comment.user && comment.user.image && comment.user.image.png" />
              <p class="name">{{ comment.user.username }}</p>
              <p class="date">{{ comment.createdAt }}</p>
            </div>
            <div class="rp-button">
              <button @click="toggleReply(comment)">Reply</button>
            </div>
          </div>
          <div class="text-content">
            <p v-if="comment.isEditing && isCurrentUserComment(comment)">
              <textarea v-model="comment.content"></textarea>
              <button @click="saveEdit(comment)">Save</button>
            </p>
            <p v-else>
              {{ comment.content }}
              <button @click="editComment(comment)" v-if="isCurrentUserComment(comment)">Edit</button>
            </p>
          </div>
        </div>
      </div>
      <div class="comment-reply-submit" v-if="comment.showReply">
        <img :src="getUserImage(currentUser.image)" alt="User Avatar" v-if="currentUser && currentUser.image && currentUser.image.png" />
        <textarea v-model="comment.replyText" class="comment-reply-text" placeholder="Add a reply..." spellcheck="false"></textarea>
        <button @click="submitReply(comment)">Reply</button>
      </div>
      <button @click="deleteComment(comment.id)" v-if="isCurrentUserComment(comment)">Delete</button>
      <div class="comment-reply" v-if="comment.replies.length > 0">
        <div class="comment-reply-content" v-for="reply in comment.replies" :key="reply.id">
          <div class="score">
            <button @click="vote(reply, 1)"><i class="plus"></i></button>
            <p>{{ reply.score }}</p>
            <button @click="vote(reply, -1)"><i class="minus"></i></button>
          </div>
          <div class="content">
            <div class="top-content">
              <div class="user-inf">
                <img :src="getUserImage(reply.user.image)" alt="User Avatar" v-if="reply.user && reply.user.image && reply.user.image.png" />
                <p class="name">{{ reply.user.username }}</p>
                <p class="date">{{ reply.createdAt }}</p>
              </div>
              <div class="rp-button">
                <button @click="deleteReply(reply.id)" v-if="isCurrentUserComment(reply)" class="delete">Delete</button>
                <button @click="editReply(reply)" v-if="isCurrentUserComment(reply)" class="edit">Edit</button>
              </div>
            </div>
            <div class="text-content">
              <p v-if="reply.isEditing && isCurrentUserComment(reply)" class="comment-reply-content-edit">
                <textarea v-model="reply.content"></textarea>
                <button @click="saveReplyEdit(reply)">Update</button>
              </p>
              <p v-else>
                {{ reply.content }}
              </p>
            </div>
          </div>
        </div>
      </div>
    </div>
    <div class="comment-add">
      <img :src="getUserImage(currentUser.image)" alt="User Avatar" v-if="currentUser && currentUser.image && currentUser.image.png" />
      <textarea v-model="newComment" class="comment-text" placeholder="Add a comment..." spellcheck="false"></textarea>
      <button @click="addComment">Send</button>
    </div>
  </div>
</template>

<script>
import jsonData from './assets/data/data.json';

export default {
  data() {
    return {
      comments: [],
      newComment: '',
      currentUser: jsonData.currentUser,
    };
  },
  mounted() {
    this.comments = jsonData.comments.map(comment => {
      return {
        ...comment,
        showReply: false,
        isEditing: false,
        replyText: '',
        replies: comment.replies.map(reply => {
          return {
            ...reply,
            isEditing: false,
          };
        }),
      };
    });
  },
  methods: {
    toggleReply(comment) {
      comment.showReply = !comment.showReply;
      if (comment.showReply) {
        comment.replyText = `@${comment.user.username} `;
      } else {
        comment.replyText = '';
      }
    },
    submitReply(comment) {
      comment.replies.push({
        id: new Date().getTime(),
        content: comment.replyText,
        createdAt: 'now',
        score: 0,
        user: this.currentUser,
        isEditing: false,
      });
      comment.replyText = '';
    },
    editComment(comment) {
      if (comment.isEditing) return;
      comment.isEditing = true;
    },
    saveEdit(comment) {
      comment.isEditing = false;
    },
    deleteComment(commentId) {
      const index = this.comments.findIndex(comment => comment.id === commentId);
      if (index !== -1 && this.isCurrentUserComment(this.comments[index])) {
        this.comments.splice(index, 1);
      }
    },
    editReply(reply) {
      if (reply.isEditing) return;
      reply.isEditing = true;
    },
    saveReplyEdit(reply) {
      reply.isEditing = false;
    },
    deleteReply(replyId) {
      const commentWithReply = this.comments.find(comment => {
        return comment.replies.find(reply => reply.id === replyId);
      });

      const index = commentWithReply.replies.findIndex(reply => reply.id === replyId);
      if (index !== -1 && this.isCurrentUserComment(commentWithReply.replies[index])) {
        commentWithReply.replies.splice(index, 1);
      }
    },
    addComment() {
      if (this.newComment.trim() === '') return;
      this.comments.push({
        id: new Date().getTime(),
        content: this.newComment,
        createdAt: 'now',
        score: 0,
        user: this.currentUser,
        replies: [],
        showReply: false,
        isEditing: false,
        replyText: '',
      });
      this.newComment = '';
    },
    vote(item, value) {
      if (!item.voted) {
        item.voted = 0;
      }
      if (item.voted === value) {
        item.score -= value;
        item.voted = 0;
      } else {
        item.score += value;
        item.voted = value;
      }
    },
    isCurrentUserComment(comment) {
      if (jsonData.currentUser) {
        return comment.user.username === jsonData.currentUser.username;
      }
      return false;
    },
    getUserImage(image) {
      if (image && image.png) {
        return require(`${image.png}`);
      }
    },
  },
};
</script>