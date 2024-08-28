<script setup>
import axios from 'axios'
import { onMounted, ref } from 'vue'

const apiBase = 'https://todolist-api.hexschool.io'
const token = ref('')
const data = ref([])

// 註冊
const signUpFields = ref({
  email: '',
  password: '',
  nickname: ''
})

const signUpMsg = ref('')

const signUp = async () => {
  try {
    const res = await axios.post(`${apiBase}/users/sign_up`, signUpFields.value)
    signUpMsg.value = '註冊成功!'
    signUpFields.value = {
      email: '',
      password: '',
      nickname: ''
    }
  } catch (error) {
    signUpMsg.value = error.response.data.message
  }
}

// 登入
const signInFields = ref({
  email: '',
  password: ''
})

const signInMsg = ref('')

const signIn = async () => {
  try {
    const res = await axios.post(`${apiBase}/users/sign_in`, signInFields.value)
    signInMsg.value = '登入成功!'
    signInFields.value = {
      email: '',
      password: ''
    }

    token.value = res.data.token

    // 設定token到期時間
    const now = new Date()
    const expire = now.setMinutes(now.getMinutes() + 30)

    // 將token存在cookie
    document.cookie = `todoToken=${token.value}; expires=${new Date(expire).toUTCString()}`

    // 取得所有待辦事項
    getAllTodos()
  } catch (error) {
    signInMsg.value = error.response.data.message
  }
}

// 驗證
const checkoutMsg = ref('')
const checkout = async () => {
  try {
    checkToken()
    console.log('token', token.value)
    if (token.value) {
      const res = await axios.get(`${apiBase}/users/checkout`, {
        headers: {
          Authorization: token.value
        }
      })
      console.log(res)
      checkoutMsg.value = '驗證成功!'
    } else {
      checkoutMsg.value = '請重新登入!'
    }
  } catch (error) {
    console.log(error)
    checkoutMsg.value = error.response.data.message
  }
}

// 登出
const signOutMsg = ref('')

const signOut = async () => {
  try {
    checkToken()

    if (token.value) {
      const res = await axios.post(`${apiBase}/users/sign_out`, null, {
        headers: {
          Authorization: token.value
        }
      })
      signOutMsg.value = res.data.message
      token.value = ''
      document.cookie = `todoToken=${token.value}`
    }
  } catch (error) {
    signOutMsg.value = error.response.data.message
  }
}

const checkToken = () => {
  const tokenValue = document.cookie
    .split('; ')
    .find((row) => row.startsWith('todoToken='))
    ?.split('=')[1]

  if (tokenValue) {
    token.value = tokenValue
  } else {
    token.value = ''
  }
}

onMounted(() => {
  checkToken()
  if (token.value) {
    getAllTodos()
  }
})

// 取得所有待辦事項
const getAllTodos = async () => {
  try {
    checkToken()
    if (token.value) {
      const res = await axios.get(`${apiBase}/todos/`, {
        headers: {
          Authorization: token.value
        }
      })
      console.log(res)
      data.value = res.data.data
    } else {
      alert('請重新登入!')
    }
  } catch (error) {
    console.log(error)
  }
}

const newTodoItem = ref('')

const addNewTodoItem = async () => {
  try {
    checkToken()
    if (token.value) {
      const res = await axios.post(
        `${apiBase}/todos/`,
        {
          content: newTodoItem.value
        },
        {
          headers: {
            Authorization: token.value
          }
        }
      )
      console.log(res)
      newTodoItem.value = ''
      getAllTodos()
    } else {
      alert('請重新登入!')
    }
  } catch (error) {
    console.log(error)
  }
}

// 修改單筆待辦事項
const editTodoItem = ref('')
const updateTodoItem = async (id) => {
  try {
    checkToken()
    if (token.value) {
      const res = await axios.put(
        `${apiBase}/todos/${id}`,
        {
          content: editTodoItem.value
        },
        {
          headers: {
            Authorization: token.value
          }
        }
      )
      console.log(res)
      getAllTodos()
    } else {
      alert('請重新登入!')
    }
  } catch (error) {
    console.log(error)
  }
}

// 刪除單筆待辦事項
const deleteTodoItem = async (id) => {
  try {
    checkToken()
    if (token.value) {
      const res = await axios.delete(`${apiBase}/todos/${id}`, {
        headers: {
          Authorization: token.value
        }
      })
      console.log(res)
      getAllTodos()
    } else {
      alert('請重新登入!')
    }
  } catch (error) {
    console.log(error)
  }
}

// 更新單筆待辦事項狀態
const updateTodoItemStatus = async (id) => {
  try {
    const res = await axios.patch(`${apiBase}/todos/${id}/toggle`, null, {
      headers: {
        Authorization: token.value
      }
    })
    console.log(res)
    getAllTodos()
  } catch (error) {
    console.log(error)
  }
}
</script>

<template>
  <h2>註冊</h2>
  <label for="signUpEmail">Email: </label>
  <input type="email" id="signUpEmail" placeholder="Email" v-model="signUpFields.email" />
  <br />
  <label for="signUpPassword">Password: </label>
  <input type="password" id="signUpPassword" placeholder="Password" v-model="signUpFields.password" />
  <br />
  <label for="signUpNickname">Nickname: </label>
  <input type="text" id="signUpNickname" placeholder="Nickname" v-model="signUpFields.nickname" />
  <br />
  <button type="button" @click="signUp">註冊</button>
  <br />
  {{ signUpMsg }}

  <h2>登入</h2>
  <label for="signInEmail">Email: </label>
  <input type="email" id="signIpEmail" placeholder="Email" v-model="signInFields.email" />
  <br />
  <label for="signInPassword">Password: </label>
  <input type="password" id="signInPassword" placeholder="Password" v-model="signInFields.password" />
  <br />
  <button type="button" @click="signIn">登入</button>
  <br />
  {{ signInMsg }}

  <h2>驗證</h2>
  <label for="validationToken">Token: </label>
  <input type="text" id="validationToken" placeholder="Token" :value="token" disabled />
  <br />
  <button type="button" @click="checkout">驗證</button>
  <br />
  {{ checkoutMsg }}

  <h2>登出</h2>
  <label for="signOutToken">Token: </label>
  <input type="text" id="signOutToken" placeholder="Token" :value="token" disabled />
  <br />
  <button type="button" @click="signOut">登出</button>
  <br />
  {{ signOutMsg }}

  <hr />
  <h2>Todo List</h2>
  {{ data }}
  <br />
  <input type="text" placeholder="待辦事項" v-model="newTodoItem" />
  <button type="button" @click="addNewTodoItem">新增待辦事項</button>
  <br />
  <table>
    <thead>
      <tr>
        <th>狀態</th>
        <th>待辦事項</th>
        <th>動作</th>
      </tr>
    </thead>
    <tbody>
      <tr v-for="item in data" :key="item.id">
        <td>{{ item.status }}</td>
        <td>{{ item.content }}</td>
        <td>
          <input type="text" v-model="editTodoItem" />
          <button type="button" @click="updateTodoItem(item.id)">更新待辦事項</button>
          <button type="button" @click="deleteTodoItem(item.id)">刪除待辦事項</button>
          <button type="button" @click="updateTodoItemStatus(item.id)">修改待辦事項狀態</button>
        </td>
      </tr>
    </tbody>
  </table>
</template>

<style scoped></style>
