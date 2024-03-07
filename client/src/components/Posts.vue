<template>
  <div>
    <h1>Book shop</h1>

    <!-- Create or Post -->
    <input type="text" v-model="title" placeholder="Title" class="title-input" />
    <input type="text" v-model="body" placeholder="Body" class="body-input" />

    <!-- render when editing -->
    <button v-if="isEditing" @click="updatePost()" class="update-button">Update</button>
    <button v-if="isEditing" @click="cancelEdit()" class="cancel-edit-button">Cancel</button>

    <!-- render when creating -->
    <button v-else @click="createPost" class="create-button">Create</button>

    <!-- List of posts -->
    <div v-for="post in posts" :key="post.id">
      <h5>[{{post.id}}] {{ post.title }}</h5>
      <p>{{ post.body }}</p>

      <button @click="editPost(post.id)" class="edit-button">Edit</button>
      <button @click="deletePost(post.id)" class="delete-button">Delete</button>
    </div>
  </div>
</template>

<script setup>
  import { ref, onMounted} from "vue";

  const posts = ref([])
  const title = ref('')
  const body = ref('')
  const post_id = ref(0)
  const isEditing = ref(false)
  const API_URL = "http://localhost:3000/posts";

  onMounted(async() => {
    const res = await fetch(API_URL)
    posts.value = await res.json()
  })

  const createPost = async() => {
    const res = await fetch(API_URL, {
      method: 'POST',
      headers: {
        'Content-type': 'application/json'
      },
      body: JSON.stringify({
        title: title.value,
        body: body.value
      })
    })

    const data = await res.json()

    posts.value.push(data)
    title.value = ''
    body.value = ''
    post_id.value = 0;
  }

  const updatePost = async() => {
    const res = await fetch (`${API_URL}/${post_id.value}`, {
      method: 'PUT',
      headers: {
        'Content-type': 'application/json'
      },
      body: JSON.stringify({
        title: title.value,
        body: body.value,
        id: post_id.value
      })
    })

    const data = await res.json()
    const index = posts.value.findIndex(post => post.id === data.id)
    posts.value[index] = data
    title.value = ''
    body.value = ''
    post_id.value = 0
    isEditing.value = false;
  }

  const cancelEdit = async() => {
    title.value = ''
    body.value = ''
    post_id.value = 0;
    isEditing.value = false
  }

  const deletePost = async(id) => {
    await fetch (`${API_URL}/${id}`, {
      method: 'DELETE'
    })
    posts.value = posts.value.filter(post => post.id != id)
  }

  const editPost = async(id) => {
    const post = posts.value.find(post => post.id === id)
    title.value = post.title
    body.value = post.body
    post_id.value = post.id
    isEditing.value = true

    window.scrollTo({
      top: 0,
      behavior: 'smooth'
    })
  }
</script>

<style scoped>
.title-input {
  width: 100%;
  padding: 12px 20px;
  margin: 8px 0;
  box-sizing: border-box;
  border: 2px solid #ccc;
  background-color: #f8f8f8;
  color: #111;
  border-radius: 4px;
  resize: vertical;
}

.body-input {
  width: 100%;
  padding: 12px 20px;
  margin: 8px 0;
  box-sizing: border-box;
  border: 2px solid #ccc;
  background-color: #f8f8f8;
  color: #111;
  border-radius: 4px;
  resize: vertical;
}

.update-button {
  margin-right: 10px;
  margin-top: 10px;
  border-color: green
}

.cancel-edit-button {
  margin-top: 10px; border-color: crimson
}

.create-button {
  margin-top: 10px;
  background: green
}

.edit-button {
  margin-right: 10px;
  background: gray
}

.delete-button {
  background: crimson
}
</style>