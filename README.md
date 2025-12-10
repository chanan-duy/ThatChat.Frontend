# ThatChat.Frontend

## Фронтовая часть

Бэк [тут](https://github.com/chanan-duy/ThatChat.Backend)

Без запущенного бэка будет выдавать ошибки

### Структура:

- `src/components` - компоненты
- `src/components/ui` - компоненты shadcn-vue (отдельная ui ~библиотека)
- `src/components/chat` - vue компоненты чата
- `src/composables` - composables (отделённая логика от vue компонентов, помогает при тестировании)
- `src/lib` - разное
- `src/router` - роуты (vue-router)
- `src/services` - сервисы. Тут api и auth api
- `src/stores` - stores
- `src/views` - vue компоненты страниц

### По Тестам

Они находятся прямо с компонентами тестирования в папках `*/tests/*.spec.ts`

Используется `allure-report` и в [CI/CD](.github/workflows/cicd.yaml) тесты выгружаются на https://chanan-duy.github.io/ThatChat.Frontend

### Запуска/Установка

Установить зависимости (npm должен быть установлен):

```bash
npm install
```

Secretes (`.env`) (пример в `.env.example`, можно переименовать в `.env` и запускать):

```dotenv
VITE_API_URL=http://localhost:5042
```

Запустить тесты ручками:

```bash
npm run test:run
```

Запустить dev проект:

```bash
npm run dev
```

Запустить build:

```bash
npm run build
```

Ну и тд, и тп.

### Скрины

![demo_1](/public/demo_1.png)
![demo_2](/public/demo_2.png)
![demo_3](/public/demo_3.png)
![demo_4](/public/demo_4.png)
![demo_5](/public/demo_5.png)
