# project_aluna_school


 # README.md: Платформа для Уроков в Школе

## Обзор

Это полноценное веб-приложение для учителей, позволяющее создавать интерактивные уроки. Сайт позволяет учителям регистрироваться, входить в систему и генерировать уроки с 8 слайдами на основе темы. Интегрируется ИИ для генерации текста (через Aitunnel.ru), изображений (Unsplash/Pexels) и озвучки текста (ElevenLabs). Уроки сохраняются и просматриваются с автоматическим переключением слайдов, синхронизированным с аудио.

**Ключевые возможности:**
- Регистрация и аутентификация пользователей.
- Мастер создания урока: Тема → Текст от ИИ → Изображения → Аудио → Сохранение и просмотр.
- Хостинг на бесплатном тарифе InfinityFree (PHP 8.2/8.3, MySQL, cURL включён; ограничения: без Composer, без ffmpeg, ~45 сек на выполнение скрипта, 5 ГБ диск, 10 МБ размер файла).
- Frontend: HTML/CSS/JS (Vanilla + опционально Tailwind CDN).
- Backend: Чистый PHP + PDO.
- Хранение: MySQL для метаданных; папки на сервере для аудио (lessons/temp и published).

**Предположения:**
- Ключи API: Aitunnel.ru (совместим с OpenAI), ElevenLabs, Unsplash/Pexels (опциональные ключи).
- Безопасность: Базовая (хэшированные пароли, проверка сессий); добавить CSRF позже.
- Дизайн: Простой, responsive; фокус на функциональности.

## Структура Проекта

Ниже приведено дерево файлов проекта. Используйте эту структуру строго при разработке.

```markdown
school-lessons-platform/                  # ← Корневая папка сайта (public_html на InfinityFree)
├── index.php                             # Главная: Форма входа/регистрации
├── config.php                            # Конфиг БД, ключи API (не коммитить в git!)
├── .htaccess                             # Защита папок, запрет прямого доступа к includes/api/lessons

├── assets/                               # Статические файлы (CSS, JS, изображения)
│   ├── css/
│   │   └── main.css                      # Глобальные стили (или Tailwind CDN в HTML)
│   ├── js/
│   │   ├── main.js                       # Основной JS (помощники fetch, утилиты)
│   │   ├── auth.js                       # Логика входа/регистрации
│   │   ├── dashboard.js                  # Взаимодействия в личном кабинете
│   │   ├── create-lesson.js              # Мастер создания урока (multi-step)
│   │   └── view-lesson.js                # Просмотр урока с синхронизацией аудио
│   └── img/
│       ├── logo.svg                      # Логотип сайта
│       └── icons/                        # Иконки (play.svg, pause.svg и т.д.)

├── includes/                             # Переиспользуемые PHP-куски (без прямого доступа)
│   ├── header.php                        # Шапка сайта (навигация, логотип)
│   ├── footer.php                        # Подвал сайта
│   ├── auth.php                          # Функции аутентификации: login(), logout(), is_logged_in()
│   ├── db.php                            # Подключение PDO
│   └── helpers.php                       # Утилиты: sanitize(), json_response(), обработка ошибок()

├── api/                                  # AJAX-эндпоинты (только POST, требуется авторизация)
│   ├── generate-text.php                 # Aitunnel.ru: Генерация текста 8 слайдов
│   ├── suggest-images.php                # Unsplash/Pexels: Предложения URL изображений
│   ├── generate-audio.php                # ElevenLabs: TTS для одного слайда (возвращает путь MP3)
│   ├── save-draft.php                    # Сохранение черновика урока (в сессию/БД)
│   └── finalize-lesson.php               # Финальное сохранение: Перемещение в published, создание URL просмотра

├── lessons/                              # Хранение данных уроков (защитить .htaccess)
│   ├── temp/                             # Черновики во время создания (очищать периодически)
│   │   └── {teacher_id}_{unix_ts}/       # Пример: 7_1738765432/
│   │       ├── slide-1.mp3
│   │       ├── slide-2.mp3
│   │       └── ... (до slide-8.mp3)
│   └── published/                        # Готовые уроки
│       └── {lesson_id}/                  # Пример: 000123/
│           ├── slide-1.mp3
│           ├── slide-2.mp3
│           └── ... 
│           └── meta.json                 # Опционально: Резервная копия метаданных урока

├── pages/                                # Основные страницы пользователей
│   ├── dashboard.php                     # Личный кабинет учителя
│   ├── create-lesson.php                 # Мастер создания урока
│   ├── edit-lesson.php                   # Редактирование существующего урока (опционально)
│   ├── view.php                          # Страница просмотра урока (?id=123&key=abc)
│   ├── register.php                      # Форма регистрации
│   └── login.php                         # Форма входа

└── logout.php                            # Обработчик выхода
```

### Примечания к папкам
- **assets/**: Доступны публично; держите файлы маленькими.
- **includes/**: Подключайте через `require_once`; запретите доступ в .htaccess.
- **api/**: Обработка AJAX; проверяйте `$_SERVER['HTTP_X_REQUESTED_WITH'] == 'XMLHttpRequest'`.
- **lessons/**: Тяжёлая по аудио; мониторьте использование диска (5 ГБ лимит). .htaccess: `Deny from all`.
- **pages/**: Основные представления; требуйте авторизацию где нужно.

## Схема Базы Данных

Используйте MySQL (создайте через phpMyAdmin на InfinityFree).

```sql
CREATE DATABASE school_lessons;

USE school_lessons;

-- Таблица пользователей
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Таблица уроков
CREATE TABLE lessons (
    id INT AUTO_INCREMENT PRIMARY KEY,
    teacher_id INT NOT NULL,
    title VARCHAR(255) NOT NULL,
    status ENUM('draft', 'published') DEFAULT 'draft',
    view_key VARCHAR(32) UNIQUE NOT NULL,  -- Случайный токен для view.php
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (teacher_id) REFERENCES users(id)
);

-- Таблица слайдов (одна строка на слайд)
CREATE TABLE slides (
    id INT AUTO_INCREMENT PRIMARY KEY,
    lesson_id INT NOT NULL,
    slide_num TINYINT NOT NULL,  -- 1-8
    title VARCHAR(255) NOT NULL,
    text TEXT NOT NULL,
    image_url VARCHAR(512),  -- Ссылка на Unsplash
    audio_path VARCHAR(255),  -- Относительный путь: lessons/published/{id}/slide-{num}.mp3
    duration_sec FLOAT DEFAULT NULL,  -- Опционально для лучшей синхронизации
    FOREIGN KEY (lesson_id) REFERENCES lessons(id)
);
```

## Установка на InfinityFree

1. Зарегистрируйтесь на бесплатном аккаунте infinityfree.com.
2. Создайте поддомен (например, lessons.example.com).
3. Загрузите файлы через File Manager или FTP.
4. Создайте MySQL БД и пользователя; обновите `config.php`.
5. Выполните схему SQL в phpMyAdmin.
6. Установите ключи API в `config.php` (Aitunnel.ru, ElevenLabs).
7. Тестируйте: Посетите index.php → зарегистрируйтесь → dashboard.

**Пример config.php:**
```php
<?php
define('DB_HOST', 'sqlXXX.infinityfree.com');
define('DB_NAME', 'if0_XXXX_school_lessons');
define('DB_USER', 'if0_XXXX_user');
define('DB_PASS', 'your_password');

define('AITUNNEL_API_KEY', 'sk-aitunnel-yourkey');
define('ELEVENLABS_API_KEY', 'your-elevenlabs-key');
define('UNSPLASH_API_KEY', 'optional-unsplash-key');  // Если используете полный API

// Подключение PDO в db.php
```
**Примечание:** Игнорируйте `config.php` в git.

## Эндпоинты API

Все в папке `/api/`. Используйте POST, ответы в JSON. Требуйте авторизацию (сессия).

- **generate-text.php**
  - Метод: POST
  - Параметры: `theme` (строка)
  - Ответ: JSON {status: 'ok', slides: [{slide:1, title:'', text:''}, ...]}
  - Использует cURL к Aitunnel.ru (/v1/chat/completions).

- **suggest-images.php**
  - Метод: POST
  - Параметры: `keyword` (строка, на слайд)
  - Ответ: JSON {status: 'ok', images: [{url: 'https://unsplash.com/...', title: ''}, ...]}
  - Получает из Unsplash (random или search).

- **generate-audio.php**
  - Метод: POST
  - Параметры: `text` (строка), `slide_num` (int), `lesson_id` (temp строка)
  - Ответ: JSON {status: 'ok', audio_path: 'lessons/temp/.../slide-X.mp3'}
  - Использует cURL к ElevenLabs (/v1/text-to-speech/{voice}).

- **save-draft.php**
  - Метод: POST
  - Параметры: `slides` (JSON-массив)
  - Ответ: JSON {status: 'ok', draft_id: 'temp_folder_name'}

- **finalize-lesson.php**
  - Метод: POST
  - Параметры: `draft_id` (строка), `title` (строка)
  - Ответ: JSON {status: 'ok', lesson_id: 123, view_url: '/view.php?id=123&key=abc'}
  - Перемещает файлы в published, обновляет БД.

**Обработка ошибок:** Все возвращают {status: 'error', message: '...'}

## Этапы Разработки

Проект разделён на 5 этапов. Реализуйте последовательно, тестируя каждый.

### Этап 1: Базовая Структура, Регистрация и Личный Кабинет (MVP Аутентификации)

**Цель:** Настроить скелет с аутентификацией пользователей.

- Создайте папки/файлы по дереву.
- Реализуйте register.php, login.php, logout.php.
- Dashboard.php: Показать кнопку "Создать урок" (ссылка на create-lesson.php).
- Используйте сессии для аутентификации.
- SQL: Создайте таблицу users.
- JS: auth.js для отправки форм (AJAX опционально).

**Изменённые файлы:** index.php, config.php, includes/*, pages/register.php, pages/login.php, dashboard.php, .htaccess.

### Этап 2: Форма Урока + Генерация Текста через Aitunnel.ru

**Цель:** Ввод темы → Генерация текста 8 слайдов.

- create-lesson.php: Форма для темы.
- api/generate-text.php: cURL к Aitunnel.ru.
- Промпт: Строгий вывод в JSON.
- Сохраните в $_SESSION['temp_slides'].
- Отобразите редактируемые textarea.
- Кнопка "Далее → Изображения".

**Изменённые файлы:** pages/create-lesson.php, api/generate-text.php, assets/js/create-lesson.js.

### Этап 3: Добавление Изображений к Слайдам (Unsplash/Pexels, только URL)

**Цель:** Предложить/выбрать изображения на слайд.

- В create-lesson.php (или multi-step JS): 8 блоков с вводом ключевого слова.
- api/suggest-images.php: Получить URL из Unsplash.
- Сохраните image_url в сессии.
- Предпросмотр <img src={url}>.
- Кнопка "Далее → Аудио".

**Изменённые файлы:** api/suggest-images.php, обновите create-lesson.js.

### Этап 4: Генерация Аудио через ElevenLabs + Базовая Синхронизация (8 Отдельных Файлов)

**Цель:** TTS на слайд, сохранение MP3.

- Кнопка "Генерировать аудио": Цикл 8 вызовов api/generate-audio.php.
- Каждый: cURL к ElevenLabs, сохранение в lessons/temp/{draft_id}/slide-X.mp3.
- Сохраните пути в сессии/БД черновика.
- JS: Обработка последовательной генерации (чтобы избежать таймаута).
- Кнопка "Сохранить урок".

**Изменённые файлы:** api/generate-audio.php, api/save-draft.php, обновите create-lesson.js.
**Примечание:** По одному слайду, если таймауты.

### Этап 5: Сохранение Урока, Страница Просмотра + Авто-Переключение Слайдов

**Цель:** Финализировать и просматривать с синхронизацией.

- api/finalize-lesson.php: Вставка в БД, перемещение файлов в published/{id}.
- Сгенерируйте view_key (md5 random).
- view.php: Загрузка слайдов из БД, 8 <div class="slide">.
- JS: Массив Audio-объектов; onended → следующее аудио + показ следующего слайда.
- Управление: Play/pause, prev/next.

**Изменённые файлы:** api/finalize-lesson.php, pages/view.php, assets/js/view-lesson.js, SQL для таблиц lessons/slides.

## Тестирование и Отладка

- Локально: Используйте XAMPP (соответствующие версии PHP/MySQL).
- На InfinityFree: Загружайте поэтапно; проверяйте логи ошибок.
- Ограничения: Разделяйте тяжёлые задачи (например, генерацию аудио).
- Очистка: Ручное удаление старых папок temp/.

## Будущие Улучшения

- Токены CSRF.
- Редактирование уроков.
- Обмен ссылками.
- Роли пользователей (ученики).
- Лучшая синхронизация: Wavesurfer.js для единого MP3 (на клиенте).

Контакты по вопросам: [Ваш Email].

Последнее обновление: 5 февраля 2026 года.
 
