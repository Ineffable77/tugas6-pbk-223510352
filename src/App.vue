<template>
  <q-layout view="hHh lpR fFf ">
    <q-header elevated class="bg-deep-purple-2 text-white ">
      <q-btn flat style="color: #004558" label="Login" />
      <q-btn flat style="color: #004558" label="About" />
      <q-btn flat style="color: #004558" label="Contact us" />
    </q-header>

    <q-page-container>
      <q-page class="container q-pa-md">
        <q-card class="q-pa-md q-mb-md rounded-borders">
          <q-card-section>
            <q-form @submit.prevent="save">
              <q-input
                v-model="form.title"
                label="title"
                outlined
                dense
                class="q-mb-md"
                :rules="[val => !!val || 'field tidak boleh kosong']"
              />
              <q-input
                v-model="form.content"
                label="Content"
                type="textarea"
                outlined
                dense
                class="q-mb-md"
                :rules="[val => !!val || 'field tidak boleh kosong']"
              />
              <q-btn type="submit" label="Save" color="negative" class="q-mb-md rounded-borders " />
            </q-form>
          </q-card-section>
        </q-card>

        <q-card class="q-pa-md q-mb-md rounded-borders">
          <q-card-section>
            <q-list bordered padding>
              <q-item
                v-for="article in articles"
                :key="article.id"
                clickable
                class="q-mb-xs rounded-borders q-gutter-xs"
              >
                <q-item-section>
                  <q-item-label>{{ article.title }}</q-item-label>
                  <q-item-label caption>{{ article.content }}</q-item-label>
                </q-item-section>
                <q-item-section side>
                  <q-btn dense round color="accent" icon="edit" @click.stop="edit(article)" />
                  <q-btn dense round color="warning" icon="delete" @click.stop="remove(article.id)" />
                </q-item-section>
              </q-item>
            </q-list>
          </q-card-section>
        </q-card>

        <q-btn label="Reload" color="negative" @click="load" class="rounded-borders" />
      </q-page>
    </q-page-container>

    <footer class="q-pa-md text-center bg-deep-purple-4 text-white">
      <div class="q-mb-md">© 2024 RandyIvando.com All Rights Reserved.</div>
      <div class="text-caption">JL. Kubang Raya | email@gmail.com | telpon 08********</div>
    </footer>>
  </q-layout>
</template>

<script setup>
import { reactive, ref, onMounted } from 'vue';
import axios from 'axios';

// JSONBin API details
const JSONBIN_API_KEY = '$2a$10$XVTGLAfQa.7V85WP/L9GtObAB8BdXzLtaGmmqvq7LvynjKwzvLuPe';
const BIN_ID = '6659f4f5acd3cb34a850b41a';

const leftDrawerOpen = ref(false);

const form = reactive({
  id: null,
  title: '',
  content: '',
});

const articles = ref([]);

async function load() {
  try {
    const response = await axios.get(`https://api.jsonbin.io/v3/b/${BIN_ID}/latest`, {
      headers: {
        'X-Master-Key': JSONBIN_API_KEY,
      },
    });
    articles.value = response.data.record.articles || [];
  } catch (error) {
    console.error('Error loading articles', error);
  }
}

async function save() {
  try {
    let updatedArticles;
    if (form.id) {
      // Update the existing article
      updatedArticles = articles.value.map((article) =>
        article.id === form.id ? { ...form } : article
      );
    } else {
      // Create a new article
      const newArticle = { ...form, id: Date.now() };
      updatedArticles = [...articles.value, newArticle];
    }
    await axios.put(
      `https://api.jsonbin.io/v3/b/${BIN_ID}`,
      { articles: updatedArticles },
      {
        headers: {
          'Content-Type': 'application/json',
          'X-Master-Key': JSONBIN_API_KEY,
        },
      }
    );
    form.id = null;
    form.title = '';
    form.content = '';
    await load();
  } catch (error) {
    console.error('Error saving article', error);
  }
}

function edit(article) {
  form.id = article.id;
  form.title = article.title;
  form.content = article.content;
}

async function remove(id) {
  try {
    const updatedArticles = articles.value.filter((article) => article.id !== id);
    await axios.put(
      `https://api.jsonbin.io/v3/b/${BIN_ID}`,
      { articles: updatedArticles },
      {
        headers: {
          'Content-Type': 'application/json',
          'X-Master-Key': JSONBIN_API_KEY,
        },
      }
    );
    await load();
  } catch (error) {
    console.error('Error deleting article', error);
  }
}

function toggleLeftDrawer() {
  leftDrawerOpen.value = !leftDrawerOpen.value;
}

function navigateTo(page) {
  console.log(`Navigating to ${page}`);
}

onMounted(load);
</script>

<style scoped>
.container {
  max-width: 800px;
  margin: auto;
}

.footer {
  background-color: #FFB6C1;
  color: white;
  width: 100%;
  font-size: medium;
  padding: 10px;
  text-align: center;
  position: relative;
  bottom: 0;
}

.footer-content {
  margin: auto;
}

.profile {
  border-radius: 50%;
  overflow: hidden;
}

.q-card {
  margin-bottom: 20px;
}

.bg-pink-9 {
  background-color: #FFB6C1 !important;
}

.bg-soft-pink {
  background-color: #FFB6C1 !important;
}

.rounded-borders {
  border-radius: 15px;
}

.q-btn {
  margin-left: 5px;
  margin-right: 5px;
}

.q-page-container {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
}

.q-page {
  flex: 1;
}
</style>
