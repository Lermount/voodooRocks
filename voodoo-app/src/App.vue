<template>
  <div class="container">
    <div v-if="isLoading" class="loader">
      <img class="loader-icon" src="../public/images/loader.svg" alt="loader">
    </div>

    <div v-else class="content">

      <h1 class="title">Список публикаций</h1>

      <div class="input-group mb-3">
        <span class="input-group-text" id="basic-addon1">&#128269;</span>
        <input type="text" v-model="authorName" class="form-control" placeholder="Поиск по автору" aria-label="Username"
          aria-describedby="basic-addon1">
      </div>

      <div class="cards">
        <ul class="card" v-for="post in filteredPosts" :key="post.id">
          <li class="card-body">
            <h5 class="card-title">{{ post.title }}</h5>
            <p class="card-text">{{ post.body }}</p>
            <p class="card-subtitle mb-2 text-body-secondary">{{ getAuthorName(post.userId) }}</p>
          </li>
        </ul>
      </div>

      <div v-if="showScrollToTop" class="scroll-to-top" @click="scrollToTop">&uarr;</div>

    </div>

  </div>
</template>

<script>
import axios from 'axios';
export default {

  data: () => ({
    isLoading: true,
    posts: [],
    users: [],
    authorName: '',
    showScrollToTop: false
  }),

  mounted() {
    this.getPosts();
    this.getUsers();
    window.addEventListener('scroll', this.handleScroll);
  },

  destroyed() {
    window.removeEventListener('scroll', this.handleScroll);
  },

  methods: {
    //Получаем данные о постах
    getPosts() {
      axios.get(`https://jsonplaceholder.typicode.com/posts`)
        .then(response => {
          this.posts = response?.data || {};
          this.isLoading = false // Когда все данные подгрузились отключаем loader
        })
        .catch(error => {
          console.error(error);
        });
    },

    //Получаем данные об авторах
    getUsers() {
      axios.get(`https://jsonplaceholder.typicode.com/users`)
        .then(response => {
          this.users = response?.data || {};
          this.isLoading = false  // Когда все данные подгрузились отключаем loader
        })
        .catch(error => {
          console.error(error);
        });
    },

    //Получаем имя автора для карточки статьи
    getAuthorName(userId) {
      const user = this.users.find(user => user.id === userId);
      return user ? user.name : '';
    },
    //Сравниваем имена авторов с заданным поисковым запросом без учета регистра и пробелов перед именем
    matchAuthorName(authorName) {
      return function (user) {
        return user.name.trim().toLowerCase().includes(authorName.trim().toLowerCase());
      };
    },
    //Отображаем прокрутку вверх, после того как пользователь проскролит >500px
    handleScroll() {
      const scrollPosition = window.scrollY;

      if (scrollPosition > 500) {
        this.showScrollToTop = true;
      } else {
        this.showScrollToTop = false;
      }
    },
    //Метод для плавной прокрутки страницы в саммый верх 
    scrollToTop() {
      window.scrollTo({
        top: 0,
        behavior: 'smooth'
      });
    },
  },

  computed: {
    //Фильтруем статьи по автору
    filteredPosts() {
      if (this.authorName) {
        const filteredUsers = this.users.filter(this.matchAuthorName(this.authorName));
        const filteredUserIds = filteredUsers.map(user => user.id);
        return this.posts.filter(post => filteredUserIds.includes(post.userId));
      } else {
        return this.posts;
      }
    }
  }
}
</script>

<style scoped>
.container {
  height: 100vh;
}

.loader {
  display: flex;
  width: 100%;
  height: 100%;
  justify-content: center;
  align-items: center;
}

.title {
  text-align: center;
}

.cards {
  display: flex;
  justify-content: space-between;
  flex-wrap: wrap;
  gap: 5px;
}

.card {
  list-style-type: none;
  padding: 0;
  margin: 0;
  flex: 1 1 40%;
}

.card-body {
  display: flex;
  flex-direction: column;
}

.card-text {
  flex-grow: 1;
}

.card-subtitle {
  text-align: right;
  margin: 0;
}

.scroll-to-top {
  position: fixed;
  bottom: 20px;
  right: 20px;
  background-color: #333;
  color: #fff;
  width: 40px;
  height: 40px;
  border-radius: 50%;
  display: flex;
  justify-content: center;
  align-items: center;
  cursor: pointer;
}

@media(max-width: 500px) {
  .card {
    flex: 1 1 100%;
  }
}
</style>