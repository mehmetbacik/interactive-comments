# Interactive Comments

Interactive Comments provides users with an interactive platform to post comments and manage them effortlessly. Developed using Vue.js, this project offers a modern and dynamic user experience. With its security-focused design, users can engage in a secure online environment.

## Features:

- **Commenting and Management:**
  - Users can enter and share text-based comments.
  - Comments support rich text formatting (bold, italic, links).
  - Users can edit, delete comments, and reply to other users.

- **Voting System:**
  - Users can vote on comments (upvote or downvote).
  - The number of votes received by a comment indicates its overall popularity.

- **Security:**
  - Comment content is sanitized to prevent XSS attacks.
  - Users can interact safely within the platform.

- **User Experience:**
  - User-friendly interface with interactive commenting features.
  - Easy and intuitive navigation between comments and replies.
  - Mobile-friendly design (responsive layout).

- **Database Backup:**
  - User comments are stored in local storage. Comments are retained even when the browser is closed or the page is refreshed.

## Technologies Used:

- **Vue.js:** A modern and flexible JavaScript framework used for creating the user interface.
  
- **DOMPurify:** Used to sanitize HTML content, providing security against XSS attacks.

- **Local Storage:** User comments are stored in the browser's local storage, ensuring data persistence even when the browser is closed.

- **Webpack:** A modular and performance-focused JavaScript compiler and bundler used in the compilation and configuration processes of the project.

## Clone the Repository

To clone the project, run the following commands:

```
git clone https://github.com/mehmetbacik/interactive-comments.git
```
```
cd interactive-comments
```

## Project setup
```
npm install
```

### Compiles and hot-reloads for development
```
npm run serve
```

### Compiles and minifies for production
```
npm run build
```

### Lints and fixes files
```
npm run lint
```

### Customize configuration
See [Configuration Reference](https://cli.vuejs.org/config/).

## Deploy

For a detailed exploration of the project, you can visit the [deployed site here](https://interactive-comments-blue.vercel.app/).

## GitHub Page

GitHub Repository: [https://github.com/mehmetbacik/interactive-comments.git](https://github.com/mehmetbacik/interactive-comments.git)

---