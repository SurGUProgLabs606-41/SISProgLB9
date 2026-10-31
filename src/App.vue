<script setup>
import { ref, computed } from 'vue'

// Учётные записи пользователей
const users = [
  {
    login: 'admin',
    password: 'admin123',
    name: 'Александр',
    role: 'Администратор',
    information: [
      'Управление пользователями',
      'Просмотр всех данных системы',
      'Настройка системы'
    ]
  },
  {
    login: 'teacher',
    password: 'teacher123',
    name: 'Елена',
    role: 'Преподаватель',
    information: [
      'Просмотр списка студентов',
      'Выставление оценок',
      'Просмотр учебных материалов'
    ]
  },
  {
    login: 'student',
    password: 'student123',
    name: 'Иван',
    role: 'Студент',
    information: [
      'Просмотр своих оценок',
      'Просмотр расписания',
      'Доступ к учебным материалам'
    ]
  }
]

// Данные формы
const login = ref('')
const password = ref('')

// Состояние программы
const currentUser = ref(null)
const errorMessage = ref('')
const attempts = ref(0)
const isBlocked = ref(false)

// Проверка возможности входа
const canLogin = computed(() => {
  return login.value.trim() !== '' &&
         password.value.trim() !== '' &&
         !isBlocked.value
})

// Функция проверки логина и пароля
function authenticate(userLogin, userPassword) {
  return users.find(user =>
    user.login === userLogin &&
    user.password === userPassword
  )
}

// Функция входа
function handleLogin() {
  errorMessage.value = ''

  // Проверка пустых полей
  if (!login.value.trim() || !password.value.trim()) {
    errorMessage.value = 'Заполните все поля'
    return
  }

  // Проверка блокировки
  if (isBlocked.value) {
    errorMessage.value = 'Вход заблокирован'
    return
  }

  // Проверка учётных данных
  const user = authenticate(
    login.value.trim(),
    password.value
  )

  if (user) {
    currentUser.value = user
    attempts.value = 0
    errorMessage.value = ''
  } else {
    attempts.value++

    if (attempts.value >= 3) {
      isBlocked.value = true
      errorMessage.value = 'Вход заблокирован после 3 неудачных попыток'
    } else {
      errorMessage.value = 'Неверный логин или пароль'
    }

    password.value = ''
  }
}

// Выход из аккаунта
function logout() {
  currentUser.value = null
  login.value = ''
  password.value = ''
  errorMessage.value = ''
}
</script>

<template>
  <main class="page">
    <section class="login-card">

      <!-- Экран входа -->
      <template v-if="!currentUser">

        <div class="logo">
          <span>LS</span>
        </div>

        <h1>Вход в систему</h1>
        <p class="subtitle">
          Введите свои учётные данные
        </p>

        <form @submit.prevent="handleLogin">

          <div class="form-group">
            <label for="login">Логин</label>
            <input
              id="login"
              v-model="login"
              type="text"
              placeholder="Введите логин"
              :disabled="isBlocked"
            >
          </div>

          <div class="form-group">
            <label for="password">Пароль</label>
            <input
              id="password"
              v-model="password"
              type="password"
              placeholder="Введите пароль"
              :disabled="isBlocked"
            >
          </div>

          <p v-if="errorMessage" class="error">
            {{ errorMessage }}
          </p>

          <p v-if="!isBlocked && attempts > 0" class="attempts">
            Осталось попыток: {{ 3 - attempts }}
          </p>

          <button
            type="submit"
            :disabled="!canLogin"
          >
            Войти
          </button>

        </form>

        <div class="test-accounts">
          <p>Тестовые учётные записи:</p>
          <span>Администратор: admin / admin123</span>
          <span>Преподаватель: teacher / teacher123</span>
          <span>Студент: student / student123</span>
        </div>

      </template>

      <!-- Экран после успешного входа -->
      <template v-else>

        <div class="logo success-logo">
          <span>✓</span>
        </div>

        <h1>Добро пожаловать!</h1>

        <p class="subtitle">
          Вы успешно вошли в систему
        </p>

        <div class="user-info">
          <p class="user-name">
            {{ currentUser.name }}
          </p>

          <span class="role">
            {{ currentUser.role }}
          </span>
        </div>

        <div class="information">
          <h2>Доступная информация</h2>

          <ul>
            <li
              v-for="(item, index) in currentUser.information"
              :key="index"
            >
              {{ item }}
            </li>
          </ul>
        </div>

        <button @click="logout" class="logout-button">
          Выйти из аккаунта
        </button>

      </template>

    </section>
  </main>
</template>