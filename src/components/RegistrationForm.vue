vue.js
<template>
  <div class="app-container">
    <!-- Приветственное сообщение -->
    <transition name="fade" @after-leave="showRegistrationText = true">
      <div v-if="showGreeting" class="greeting-message">
        <h2>Привет!</h2>
      </div>
    </transition>

    <!-- Сообщение о регистрации -->
    <transition name="fade" @after-leave="showRegistrationForm = true">
      <div v-if="showRegistrationText" class="registration-text">
        <h3>Чтобы продолжить, пройди регистрацию.</h3>
      </div>
    </transition>

    <!-- Форма регистрации -->
    <transition name="slide-up">
      <div v-if="showRegistrationForm" class="registration-container">
        <h2>Регистрация</h2>
        <form @submit.prevent="submitRegistration">
          <div class="form-group">
            <label for="lastName">Фамилия:</label>
            <input type="text" id="lastName" v-model="lastName" required />
          </div>
          <div class="form-group">
            <label for="firstName">Имя:</label>
            <input type="text" id="firstName" v-model="firstName" required />
          </div>
          <div class="form-group">
            <label for="middleName">Отчество:</label>
            <input type="text" id="middleName" v-model="middleName" />
          </div>
          <div class="form-group">
            <label for="birthDate">Дата рождения:</label>
            <input type="date" id="birthDate" v-model="birthDate" required />
          </div>
          <div class="form-group">
            <label for="birthTime">Время рождения:</label>
            <input type="time" id="birthTime" v-model="birthTime" required />
          </div>
          <button type="submit" class="submit-button">Зарегистрироваться</button>
        </form>
      </div>
    </transition>
  </div>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      showGreeting: true,            // Показываем "Привет!"
      showRegistrationText: false,   // Показываем "Пройди регистрацию"
      showRegistrationForm: false,   // Показываем форму регистрации
      telegramId: null,

      // Данные формы
      lastName: "",
      firstName: "",
      middleName: "",
      birthDate: "",
      birthTime: "",
    };
  },
  watch: {
    // Когда появляется текст "Чтобы продолжить...", через 3 секунды скрываем его и показываем форму
    showRegistrationText(newVal) {
      if (newVal) {
        setTimeout(() => {
          this.showRegistrationText = false;
        }, 3000);
      }
    },
  },
  methods: {
    // Инициализация пользователя из Telegram WebApp
    async initializeTelegramUser() {
      if (window.Telegram?.WebApp) {
        const tg = window.Telegram.WebApp;
        const initData = tg.initDataUnsafe;
        this.telegramId = initData?.user?.id || null;
        tg.expand();
      } else {
        alert("Telegram Web App не поддерживается.");
      }
    },

    // Отправка данных на сервер
    async submitRegistration() {
      const userData = {
        tg_id: this.telegramId,
        surname: this.lastName,
        name: this.firstName,
        patronymic: this.middleName,
        birth_date: this.birthDate,
        birth_time: this.birthTime,
      };

      try {
        const response = await axios.post("https://uniback-1.onrender.com/api/register", userData);
        if (response.data.status === "success") {
          alert("Регистрация прошла успешно!");
        } else {
          alert("Не удалось зарегистрироваться. Попробуйте снова.");
        }
      } catch (error) {
        console.error("Ошибка при регистрации:", error);
        alert("Не удалось зарегистрироваться. Попробуйте снова.");
      }
    },
  },
  mounted() {
    // Через 3 секунды убираем приветствие
    setTimeout(() => {
      this.showGreeting = false;
    }, 3000);

    // Инициализируем пользователя из Telegram
    this.initializeTelegramUser();
  },
};
</script>

<style scoped>
/* Сброс и базовые стили */
* {
  font-family: "Montserrat", sans-serif;
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

:root {
  background: #fff;
}

html,
body {
  width: 100%;
  height: 100%;
  line-height: 1.6;
  background: #fff;
  /* Убираем overflow:hidden, чтобы страница могла прокручиваться, если контент не влезает */
  overflow: auto;
}

/* Контейнер приложения */
.app-container {
  display: flex;
  justify-content: center;
  align-items: center;
  position: relative;
  min-height: 100vh;
  max-width: 100vw;
  max-height: 100vh;
  width: 100%;
  padding: 20px;
  background: #fff;
  overflow: hidden; /* Чтобы избежать горизонтального скролла */
}

/* Приветствие и текст о регистрации */
.greeting-message h2,
.registration-text h3 {
  font-size: 1.5rem;
  background: linear-gradient(45deg, #f70eff, #7700ff, #750cff);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  -webkit-text-fill-color: transparent;
  white-space: normal;
  word-wrap: break-word;
  padding: 0 10px;
  text-align: center;
}

/* Анимации переходов */
.fade-enter-active,
.fade-leave-active {
  transition: opacity 1s ease;
}
.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

.slide-up-enter-active,
.slide-up-leave-active {
  transition: transform 0.5s ease, opacity 0.5s ease;
}
.slide-up-enter-from {
  transform: translateY(100%);
  opacity: 0;
}
.slide-up-leave-to {
  transform: translateY(-100%);
  opacity: 0;
}
.slide-up-enter-to,
.slide-up-leave-from {
  transform: translateY(0);
  opacity: 1;
}

/* Контейнер формы */
.registration-container {
  width: 90%;
  max-width: 400px;
  min-height: 100vh;
  min-width: 280px;
  padding: 20px;
  background: linear-gradient(45deg, #1f5bfe, #741efe, #6c11ff);
  background-size: 400% 400%;
  animation: gradient 4s ease infinite;
  border-radius: 10px;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.2);
  display: flex;
  flex-direction: column;
  align-items: center;
  position: fixed;
}

/* Поля формы */
.form-group {
  margin-bottom: 15px;
  width: 100%;
}
.form-group label {
  display: block;
  margin-bottom: 5px;
  font-weight: bold;
  color: #fff;
  font-size: 0.9rem;
}
.form-group input {
  width: 100%;
  padding: 8px;
  border: 1px solid rgba(255, 255, 255, 0.3);
  border-radius: 4px;
  background: rgba(255, 255, 255, 0.1);
  color: #fff;
  font-size: 0.9rem;
  box-sizing: border-box;
}
.form-group input::placeholder {
  color: rgba(255, 255, 255, 0.7);
}

/* Кнопка */
.submit-button {
  width: 100%;
  padding: 10px;
  background: #fb0eff;
  color: #fff;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: background 0.3s ease;
  font-size: 0.9rem;
  margin-top: 5px;
}
.submit-button:hover {
  background: #e62ee6;
}

/* Анимация градиента фона */
@keyframes gradient {
  0% {
    background-position: 0% 50%;
  }
  50% {
    background-position: 100% 50%;
  }
  100% {
    background-position: 0% 50%;
  }
}

/* Адаптив */
@media (max-width: 500px) {
  .registration-container {
    .app-container {
      text-align: center;
    }
    width: 100%;
    padding: 15px;
  }

  .greeting-message h2,
  .registration-text h3 {
    font-size: 1rem;
  }
}
</style>
