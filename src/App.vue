<template>
  <div id="app">
    <div v-for="comment in comments" :key="comment.id">
      <p>{{ comment.content }}</p>
      <button @click="toggleReply(comment)">Reply</button>
      <div v-if="comment.showReply">
        <input v-model="replyText" placeholder="Reply">
        <button @click="submitReply(comment)">Send</button>
      </div>
      <button @click="editComment(comment.id)">Edit</button>
      <button @click="deleteComment(comment.id)">Delete</button>
    </div>
    <div>
      <input v-model="newComment" placeholder="Comment">
      <button @click="addComment">Comment Add</button>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      comments: [],
      newComment: '',
      replyText: '',
    };
  },
  methods: {
    toggleReply(comment) {
      comment.showReply = !comment.showReply;
    },
    submitReply(comment) {
      comment.replies.push({ content: this.replyText });
      this.replyText = '';
    },
    editComment(commentId) {
      const comment = this.comments.find(comment => comment.id === commentId);
      if (comment) {
        const newText = prompt('Edit:', comment.content);
        if (newText !== null) {
          comment.content = newText;
        }
      }
    },
    deleteComment(commentId) {
      const index = this.comments.findIndex(comment => comment.id === commentId);
      if (index !== -1) {
        this.comments.splice(index, 1);
      }
    },
    addComment() {
      if (this.newComment.trim() === '') return;
      this.comments.push({
        content: this.newComment,
        createdAt: 'now',
        id: new Date().getTime(),
        replies: [],
        score: 0,
        user: {
          image: {
            png: '/images/avatars/image-juliusomo.png',
            webp: '/images/avatars/image-juliusomo.webp',
          },
          username: 'juliusomo',
        },
        showReply: false,
      });
      this.newComment = '';
    },
  },
};
</script>

