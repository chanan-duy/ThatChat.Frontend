# ThatChat.Frontend

Фронтовая часть

Без запущенного бэка будет выдавать ошибки

Структура:

- `src/components` - компоненты
- `src/components/ui` - компоненты shadcn-vue (отдельная ui ~библиотека)
- `src/components/chat` - vue компоненты чата
- `src/composables` - composables (отделённая логика от vue компонентов, помогает при тестировании)
- `src/lib` - разное
- `src/router` - роуты (vue-router)
- `src/services` - сервисы. Тут api и auth api
- `src/stores` - stores
- `src/views` - vue компоненты страниц

По тестам. Они находятся прямо с компонентами тестирования в папках `*/tests/*.spec.ts`

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
