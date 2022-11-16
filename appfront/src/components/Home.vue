<template>

  <div class="home">
    <ul>
      <li v-for="(book, index) in books" :key="index" style="display:block">
        {{ index }}-{{ book.name }}-{{ book.author }}
      </li>
    </ul>
    <form action="">
      Book Name：<input type="text" placeholder="book name" v-model="inputBook.name"><br>
      Author：<input type="text" placeholder="book author" v-model="inputBook.author"><br>
    </form>
    <button type="submit" @click="bookSubmit()">submit</button>
  </div>
</template>

<script>
import Axios from 'axios'

export default {
  name: 'home',
  data() {
    return {
      // books list data
      books: [
        {name: 'test', author: 't'},
        {name: 'test2', author: 't2'}
      ],
      // book data in the form
      inputBook: {
        "name": "",
        "author": "",
      }
    }
  },
  methods: {
    loadBooks() {
      Axios('http://localhost:8000/api/books/')
          .then((response) => this.books = response.data)
          .catch((error) => console.log(error));
    },
    bookSubmit() {
      Axios.post(`http://localhost:8000/api/books/`, {
        'name': this.inputBook.name,
        'author': this.inputBook.author
      }).then(response => {
        console.log(response)
        this.loadBooks()
      })

    } // add a book to backend when click the button
  },
  created: function () {
    this.loadBooks()
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1, h2 {
  font-weight: normal;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  display: inline-block;
  margin: 0 10px;
}

a {
  color: #42b983;
}
</style>