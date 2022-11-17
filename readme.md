### Vue3 and vite on django

Set the vue development environment to django

1. install npm 
```
$ brew install node
```

2. create django project
```
$ pip install django
$ django-admin startproject book_demo
```

3. create vue3 project 
```
$ npm install -g @vue/cli  #vue3 using vite 
```
&emsp;&emsp;✔ Project name: … appfront

&emsp;&emsp;✔ Package name: … appfront

&emsp;&emsp;✔ Add TypeScript? … **No** / Yes

&emsp;&emsp;✔ Add JSX Support? … **No** / Yes

&emsp;&emsp;✔ Add Vue Router for Single Page Application development? … **No** / Yes

&emsp;&emsp;✔ Add Pinia for state management? … **No** / Yes

&emsp;&emsp;✔ Add Vitest for Unit Testing? … **No** / Yes

&emsp;&emsp;✔ Add an End-to-End Testing Solution? › No

&emsp;&emsp;✔ Add ESLint for code quality? … **No** / Yes

**<br>** 

4. install front end package
```
$ npm install
$ npm install axios # replace ajax
```

5. development mode (it will run a node server )
```
$ npm run dev
```

6. modify vite config 

```
# vite.config.js

import {fileURLToPath, URL} from 'node:url'

const {resolve} = require('path');
import {defineConfig} from 'vite'
import vue from '@vitejs/plugin-vue'


// https://vitejs.dev/config/
export default defineConfig({
    plugins: [vue()],
    resolve: {
        alias: {
            '@': fileURLToPath(new URL('./src', import.meta.url))
        },
        extensions: ['.js', '.json'],
    },
    build: {
        outDir: resolve('./static/dist'),
        assetsDir: '',
        manifest: true,
        emptyOutDir: true,
        target: 'es2015',
        rollupOptions: {
            input: {
                main: resolve('./src/main.js'),
            },
            output: {
                entryFileNames: `[name].js`,
                chunkFileNames: `[name].js`,
                assetFileNames: `[name].[ext]`
            },
        }
    }
})

```

7. Add django_vite config to settings.py
```
# Where ViteJS assets are built.
DJANGO_VITE_ASSETS_PATH = BASE_DIR / "appfront" / "static" / "dist"

# If use HMR or not.
DJANGO_VITE_DEV_MODE = DEBUG


# Include DJANGO_VITE_ASSETS_PATH into STATICFILES_DIRS to be copied inside
# when run command python manage.py collectstatic
STATICFILES_DIRS = [DJANGO_VITE_ASSETS_PATH]

```

8. add index.html in "templates" folder
```
# index.html
{% load django_vite %}

<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Vite App</title>
  </head>
  <body>
    <div id="app"></div>

    {% vite_hmr_client %}
    {% vite_asset 'main.js' %}
  </body>
</html>
```
9. add "appfront/src/components/Home.vue"
```
<template>

  <div class="home">
    <ul>
      <li v-for="(book, index) in books" :key="index" style="display:block">
        {{ index }}-{{ book.name }}-{{ book.author }}
      </li>
    </ul>
    <form action="">
      Book Name2：<input type="text" placeholder="book name" v-model="inputBook.name"><br>
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
      Axios('http://127.0.0.1:8000/api/books/')
          .then((response) => this.books = response.data)
          .catch((error) => console.log(error));
    },
    bookSubmit() {
      Axios.post(`http://127.0.0.1:8000/api/books/`, {
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
```

10. modify "/appfront/src/App.vue"
```
import HelloWorld from './components/HelloWorld.vue'
import TheWelcome from './components/TheWelcome.vue'
import Home from "./components/Home.vue";
</script>

<template>
  <main>
    <Home />
  </main>
</template>

<style scoped>
header {
  line-height: 1.5;
}

.logo {
  display: block;
  margin: 0 auto 2rem;
}

@media (min-width: 1024px) {
  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }

  .logo {
    margin: 0 2rem 0 0;
  }

  header .wrapper {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
  }
}
</style>
```


11. create django application "books" (You can refer to the django rest framework example )

12.add variable to "setting.py" (it can get settings.py "debug" value in template )
```
INTERNAL_IPS = ["127.0.0.1"]
```

13.modify "templates/index.html"
```
  <body>
    <div id="app"></div>
        {%  if debug  %}
            {% vite_hmr_client %}
            {% vite_asset 'main.js' %}
        {% else %}
             <script type="module" src="{% static "dist/main.js" %}"></script>
             <link rel="stylesheet" href="{% static "dist/main.css" %}">
        {% endif %}
  </body>
```
14.set the production mode configuration (you should modify "book_demo/urls.py" ,if you want to use production mode ,
you need to run `npm run build` and `python manage.py collectionstatic` )
```
from django.urls import path, include, re_path
from django.views import static

urlpatterns = [
    re_path(r'^static/(?P<path>.*)$', static.serve, {'document_root': settings.STATIC_ROOT}, name='static'),
    ...
]
```

