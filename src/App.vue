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
              <span class="name">{{ comment.user.username }} <p v-if="isCommentOwner(comment.user)" class="comment-owner">you</p></span>
              <p class="date">{{ comment.createdAt }}</p>
            </div>
            <div class="rp-button">
              <button @click="deleteCommentConfirmation(comment.id)" v-if="isCurrentUserComment(comment)" class="delete">Delete</button>
              <button @click="editComment(comment)" v-if="isCurrentUserComment(comment)" class="edit">Edit</button>
              <button @click="toggleReply(comment)" class="reply">Reply</button>
            </div>
          </div>
          <div class="text-content">
            <p v-if="comment.isEditing && isCurrentUserComment(comment)" class="comment-content-edit">
              <textarea v-model="comment.content"></textarea>
              <button @click="saveEdit(comment)">Update</button>
            </p>
            <p v-else>
              {{ comment.content }}
            </p>
          </div>
        </div>
      </div>
      <div class="comment-reply-submit" v-if="comment.showReply">
        <img :src="getUserImage(currentUser.image)" alt="User Avatar" v-if="currentUser && currentUser.image && currentUser.image.png" />
        <textarea v-model="comment.replyText" class="comment-reply-text" placeholder="Add a reply..." spellcheck="false"></textarea>
        <button @click="submitReply(comment)">Reply</button>
      </div>
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
                <span class="name">{{ reply.user.username }} <p v-if="isCommentOwner(reply.user)" class="comment-owner">you</p></span>
                <p class="date">{{ reply.createdAt }}</p>
              </div>
              <div class="rp-button">
                <button @click="deleteReplyConfirmation(reply.id)" v-if="isCurrentUserComment(reply)" class="delete">Delete</button>
                <button @click="editReply(reply)" v-if="isCurrentUserComment(reply)" class="edit">Edit</button>
              </div>
            </div>
            <div class="text-content">
              <p v-if="reply.isEditing && isCurrentUserComment(reply)" class="comment-reply-content-edit">
                <textarea v-html="getCleanContent(reply.content)"></textarea>
                <button @click="saveReplyEdit(reply)">Update</button>
              </p>
              <p v-else>
                <span v-if="reply.replyingTo" class="replyingTo">@{{ reply.replyingTo }}</span>
                <span v-html="getCleanContent(reply.content)" :class="{ 'r-content': !reply.replyingTo }"></span>
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
    <div class="confirmation-popup overlay" v-if="confirmDelete">
      <div class="confirmation-popup-content">
        <span>Delete comment</span>
        <p>Are you sure you want to delete this comment? This will remove the comment and can't be undone.</p>
        <div class="popup-button-group">
          <button @click="cancelDeleteAction" class="cancel">No, cancel</button>
          <button @click="confirmDeleteAction" class="confirm">Yes, delete</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import jsonData from './assets/data/data.json';
import DOMPurify from 'dompurify';

export default {
  data() {
    return {
      comments: [],
      newComment: '',
      currentUser: jsonData.currentUser,
      confirmDelete: false,
      commentToDelete: null,
      replyToDelete: null,
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
    const storedComments = localStorage.getItem('comments');
    const jsonComments = storedComments ? JSON.parse(storedComments) : [];
    jsonComments.forEach(comment => {
      const exists = this.comments.some(existingComment => existingComment.id === comment.id);
      if (!exists) {
        this.comments.push(comment);
      }
    });
    this.comments = this.comments.map(comment => {
      return {
        ...comment,
        replies: comment.replies.map(reply => {
          return {
            ...reply,
            isEditing: false,
          };
        }),
      };
    });
  },
  watch: {
    comments: {
      handler(newComments) {
        localStorage.setItem('comments', JSON.stringify(newComments));
      },
      deep: true,
    },
  },
  methods: {
    toggleReply(comment) {
      comment.showReply = !comment.showReply;
      if (comment.showReply) {
        comment.replyText = '';
      }
    },
    submitReply(comment) {
      const replyContent = `<p class='rc-user'>@${comment.user.username}</p> ${comment.replyText}`;
      comment.replies.push({
          id: new Date().getTime(),
          content: replyContent,
          createdAt: 'now',
          score: 0,
          user: this.currentUser,
          isEditing: false,
      });
      comment.replyText = '';
      comment.showReply = false;
    },
    editComment(comment) {
      if (comment.isEditing) return;
      comment.isEditing = true;
    },
    saveEdit(comment) {
      comment.isEditing = false;
    },
    deleteCommentConfirmation(commentId) {
      this.commentToDelete = commentId;
      this.confirmDelete = true;
    },
    deleteReplyConfirmation(replyId) {
      this.replyToDelete = replyId;
      this.confirmDelete = true;
    },
    confirmDeleteAction() {
      if (this.commentToDelete !== null) {
        const index = this.comments.findIndex(comment => comment.id === this.commentToDelete);
        if (index !== -1 && this.isCurrentUserComment(this.comments[index])) {
          this.comments.splice(index, 1);
        }
        this.commentToDelete = null;
      }
      if (this.replyToDelete !== null) {
        for (const comment of this.comments) {
          const replyIndex = comment.replies.findIndex(reply => reply.id === this.replyToDelete);
          if (replyIndex !== -1 && this.isCurrentUserComment(comment.replies[replyIndex])) {
            comment.replies.splice(replyIndex, 1);
          }
        }
        this.replyToDelete = null;
      }
      this.confirmDelete = false;
    },
    cancelDeleteAction() {
      this.commentToDelete = null;
      this.replyToDelete = null;
      this.confirmDelete = false;
    },
    editReply(reply) {
      if (reply.isEditing) return;
      reply.isEditing = true;
    },
    saveReplyEdit(reply) {
      reply.isEditing = false;
    },
    addComment() {
      if (this.newComment.trim() === '') return;
  
      const newComment = {
        id: new Date().getTime(),
        content: this.newComment,
        createdAt: 'now',
        score: 0,
        user: this.currentUser,
        replies: [],
        showReply: false,
        isEditing: false,
        replyText: '',
      };

      if (this.isCommentOwner(newComment.user)) {
        newComment.user.isCurrentUser = true;
      }

      this.comments.push(newComment);
      this.newComment = '';
    },
    isCommentOwner(user) {
      return user.username === this.currentUser.username;
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
    getCleanContent(content) {
      return DOMPurify.sanitize(content);
    }
  },
};
</script>