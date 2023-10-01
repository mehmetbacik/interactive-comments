<template>
  <div id="app">
    <div v-for="comment in comments" :key="comment.id">
      <p v-if="comment.isEditing && isCurrentUserComment(comment)">
        <textarea v-model="comment.content"></textarea>
        <button @click="saveEdit(comment)">Save</button>
      </p>
      <p v-else>
        {{ comment.content }}
        <button @click="editComment(comment)" v-if="isCurrentUserComment(comment)">Edit</button>
      </p>
      <p>Created at: {{ comment.createdAt }}</p>
      <p>Score: {{ comment.score }}</p>
      <p>By: {{ comment.user.username }}</p>
      <img :src="getUserImage(comment.user.image)" alt="User Avatar" v-if="comment.user && comment.user.image && comment.user.image.png" />
      <button @click="toggleReply(comment)">Reply</button>
      <button @click="vote(comment, 1)">+</button>
      <button @click="vote(comment, -1)">-</button>
      <div v-if="comment.showReply">
        <img :src="getUserImage(currentUser.image)" alt="User Avatar" v-if="currentUser && currentUser.image && currentUser.image.png" />
        <input v-model="comment.replyText" placeholder="Reply">
        <button @click="submitReply(comment)">Send</button>
      </div>
      <button @click="deleteComment(comment.id)" v-if="isCurrentUserComment(comment)">Delete</button>
      <div v-for="reply in comment.replies" :key="reply.id">
        <p v-if="reply.isEditing && isCurrentUserComment(reply)">
          <textarea v-model="reply.content"></textarea>
          <button @click="saveReplyEdit(reply)">Save</button>
        </p>
        <p v-else>
          {{ reply.content }}
          <button @click="editReply(reply)" v-if="isCurrentUserComment(reply)">Edit</button>
        </p>
        <p>Created at: {{ reply.createdAt }}</p>
        <p>Score: {{ reply.score }}</p>
        <p>By: {{ reply.user.username }}</p>
        <img :src="getUserImage(reply.user.image)" alt="User Avatar" v-if="reply.user && reply.user.image && reply.user.image.png" />
        <button @click="vote(reply, 1)">+</button>
        <button @click="vote(reply, -1)">-</button>
        <button @click="deleteReply(reply.id)" v-if="isCurrentUserComment(reply)">Delete</button>
      </div>
    </div>
    <div>
      <img :src="getUserImage(currentUser.image)" alt="User Avatar" v-if="currentUser && currentUser.image && currentUser.image.png" />
      <input v-model="newComment" placeholder="Comment">
      <button @click="addComment">Comment Add</button>
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