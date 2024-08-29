<script setup>
import axios from 'axios'
import { onMounted, ref } from 'vue'

const apiBaseUrl = 'https://todolist-api.hexschool.io'
const tokenValue = ref('')
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
    const res = await axios.post(`${apiBaseUrl}/users/sign_up`, signUpFields.value)
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
    const res = await axios.post(`${apiBaseUrl}/users/sign_in`, signInFields.value)
    signInMsg.value = '登入成功!'
    signInFields.value = {
      email: '',
      password: ''
    }

    // 設定token到期時間
    const now = new Date()
    const expire = now.setMinutes(now.getMinutes() + 30)

    // 將token存在cookie
    document.cookie = `todoToken=${res.data.token}; expires=${new Date(expire).toUTCString()}`

    // 取得所有待辦事項
    getAllTodos()

    // 用於顯示token值
    tokenValue.value = getToken()
  } catch (error) {
    signInMsg.value = error.response.data.message
  }
}

// 驗證
const checkoutMsg = ref('')
const checkout = async () => {
  try {
    const token = getToken()
    if (token) {
      const res = await axios.get(`${apiBaseUrl}/users/checkout`, {
        headers: {
          Authorization: token
        }
      })
      checkoutMsg.value = '驗證成功!'
    } else {
      checkoutMsg.value = '請重新登入!'
    }
  } catch (error) {
    checkoutMsg.value = error.response.data.message
  }
}

// 登出
const signOutMsg = ref('')

const signOut = async () => {
  try {
    const token = getToken()

    if (token) {
      const res = await axios.post(`${apiBaseUrl}/users/sign_out`, null, {
        headers: {
          Authorization: token
        }
      })
      signOutMsg.value = res.data.message

      // 設定token到期時間
      const now = new Date()
      const expire = now.setMinutes(now.getMinutes() - 1)

      // 刪除token
      document.cookie = `todoToken=${token}; expires=${new Date(expire).toUTCString()}`

      // 移除顯示token
      tokenValue.value = ''
    } else {
      signOutMsg.value = '請重新登入!'
    }
  } catch (error) {
    signOutMsg.value = error.response.data.message
  }
}

// 取得token值
const getToken = () => {
  const token = document.cookie
    .split('; ')
    .find((row) => row.startsWith('todoToken='))
    ?.split('=')[1]
  return token
}

onMounted(() => {
  const token = getToken()
  if (token) {
    getAllTodos()
  }
})

// 取得所有待辦事項
const getAllTodos = async () => {
  try {
    await checkout()
    const token = getToken()
    if (token) {
      const res = await axios.get(`${apiBaseUrl}/todos/`, {
        headers: {
          Authorization: token
        }
      })
      data.value = res.data.data
    } else {
      alert('請重新登入!')
    }
  } catch (error) {
    console.log(error)
  }
}

// 新增一筆待辦事項
const newTodoItem = ref('')

const addNewTodoItem = async () => {
  if (!newTodoItem.value) {
    return
  }
  try {
    const token = getToken()
    if (token) {
      const res = await axios.post(
        `${apiBaseUrl}/todos/`,
        {
          content: newTodoItem.value
        },
        {
          headers: {
            Authorization: token
          }
        }
      )
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
const editTodoItem = ref({
  id: '',
  content: ''
})

const resetEditTodoItem = () => {
  editTodoItem.value = {
    id: '',
    content: ''
  }
}

const updateTodoItem = (id) => {
  const index = data.value.findIndex((item) => item.id === id)
  editTodoItem.value.id = id
  editTodoItem.value.content = data.value[index].content
}

const cancelUpdateTodoItem = () => {
  resetEditTodoItem()
}

const confirmUpdateTodoItem = async () => {
  try {
    const token = getToken()
    if (token) {
      const res = await axios.put(
        `${apiBaseUrl}/todos/${editTodoItem.value.id}`,
        {
          content: editTodoItem.value.content
        },
        {
          headers: {
            Authorization: token
          }
        }
      )
      getAllTodos()
      resetEditTodoItem()
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
    const token = getToken()
    if (token) {
      const res = await axios.delete(`${apiBaseUrl}/todos/${id}`, {
        headers: {
          Authorization: token
        }
      })
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
    const token = getToken()
    const res = await axios.patch(`${apiBaseUrl}/todos/${id}/toggle`, null, {
      headers: {
        Authorization: token
      }
    })
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
  <input type="text" id="validationToken" placeholder="Token" :value="tokenValue" disabled />
  <br />
  <button type="button" @click="checkout">驗證</button>
  <br />
  {{ checkoutMsg }}

  <h2>登出</h2>
  <label for="signOutToken">Token: </label>
  <input type="text" id="signOutToken" placeholder="Token" :value="tokenValue" disabled />
  <br />
  <button type="button" @click="signOut">登出</button>
  <br />
  {{ signOutMsg }}

  <hr />
  <div v-if="tokenValue">
    <h2>Todo List</h2>
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
          <td>{{ item.status ? '完成' : '未完成' }}</td>
          <td>{{ item.content }}</td>
          <td>
            <span v-if="editTodoItem.id === item.id">
              <input type="text" v-model="editTodoItem.content" />
              <button type="button" @click="cancelUpdateTodoItem()">取消</button>
              <button type="button" @click="confirmUpdateTodoItem()">確認</button>
            </span>
            <span v-else>
              <button type="button" @click="updateTodoItem(item.id)">更新待辦事項</button>
            </span>
            <button type="button" @click="deleteTodoItem(item.id)">刪除待辦事項</button>
            <button type="button" @click="updateTodoItemStatus(item.id)">修改待辦事項狀態</button>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<style scoped></style>
