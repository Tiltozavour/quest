
# Firebase Services

<details>
  <summary> <h2> 🌱 Junior </h2> </summary>

<details>
  <summary> Расскажите про сервисы в Firebase </summary>

**Firebase Services** — набор готовых бэкенд-сервисов от Google для мобильных и веб-приложений.

---

## **Основные сервисы:**

### **1. Базы данных**
- **Cloud Firestore** (рекомендуется) — документная БД
- **Realtime Database** — JSON-дерево, проще для простых случаев

### **2. Аутентификация (Auth)**
- Готовые провайдеры: Email/пароль, Google, Facebook и др.
- Управление пользователями

### **3. Хранилище (Storage)**
- Cloud Storage для файлов (фото, видео)
- Автоматическое масштабирование

### **4. Push-уведомления (Cloud Messaging - FCM)**
- Отправка уведомлений на устройства

### **5. Аналитика (Analytics)**
- Бесплатная аналитика поведения пользователей

### **6. Краш-репорты (Crashlytics)**
- Отслеживание ошибок и крашей

### **7. Конфигурация (Remote Config)**
- Изменение настроек приложения без публикации обновлений

### **8. Cloud Functions**
- Серверный код без серверов (бессерверные функции)
- Запуск по событиям (Auth, Database, Storage)

### **9. Производительность (Performance Monitoring)**
- Мониторинг скорости работы приложения

---

## **Ключевые преимущества:**

✅ **Нет серверного кода** — готовые решения  
✅ **Быстрый старт** — минуты вместо недель  
✅ **Автомасштабирование** — не нужно админить  
✅ **Интеграция** — все сервисы работают вместе  
✅ **Бесплатный тариф** для старта  

---

## **Типичный стек:**

**Для MVP:** Auth + Firestore + Analytics + Crashlytics  
**Для соцсети:** Auth + Firestore + Storage + FCM  
**Для e-commerce:** Auth + Firestore + Analytics + Remote Config  

---

## **Как подключить:**

1. Добавить проект в [Firebase Console](https://console.firebase.google.com)
2. Скачать `google-services.json`
3. Добавить зависимости в `build.gradle`
4. Инициализировать в коде

---

**Итог:** Firebase — это готовый бэкенд для быстрого запуска и масштабирования приложений без серверной разработки.

</details>

<details>
  <summary> Как добавить Firebase в проект?  </summary>

  **Добавление Firebase в Android проект:**

## **Шаг 1: Создать проект в Firebase Console**
1. Зайти на [console.firebase.google.com](https://console.firebase.google.com)
2. Нажать "Создать проект"
3. Ввести название проекта
4. Выключить Google Analytics (или оставить)
5. Создать проект

## **Шаг 2: Добавить приложение Android**
1. В проекте нажать `+ Добавить приложение`
2. Выбрать Android (символ Android)
3. **Указать:**
   - **Имя пакета** (из `build.gradle` → `applicationId`)
   - **Никнейм приложения** (опционально)
   - **SHA-1** (для Auth с Google Sign-In, опционально)
4. Нажать "Зарегистрировать приложение"

## **Шаг 3: Скачать конфиг файл**
1. Скачать `google-services.json`
2. Положить в **корень модуля app** проекта:
```
project/
├── app/
│   ├── google-services.json  ← ВОТ СЮДА
│   ├── build.gradle
│   └── src/
```

## **Шаг 4: Настроить Gradle**

### **project-level `build.gradle`**:
```gradle
// Top-level build.gradle
buildscript {
    dependencies {
        // Добавить этот classpath
        classpath 'com.google.gms:google-services:4.4.0'
    }
}
```

### **app-level `build.gradle`**:
```gradle
// app/build.gradle
plugins {
    id 'com.android.application'
    id 'com.google.gms.google-services'  // Добавить этот plugin
}

dependencies {
    // Firebase BoM (Bill of Materials) - управление версиями
    implementation platform('com.google.firebase:firebase-bom:32.4.0')
    
    // Добавить нужные Firebase зависимости:
    implementation 'com.google.firebase:firebase-analytics'
    implementation 'com.google.firebase:firebase-auth'
    implementation 'com.google.firebase:firebase-firestore'
    implementation 'com.google.firebase:firebase-storage'
    // Добавлять только нужные!
}
```

## **Шаг 5: Синхронизировать проект**
1. Нажать "Sync Now" в Android Studio
2. Или выполнить `./gradlew clean build`

## **Шаг 6: Проверить подключение** (опционально)
```kotlin
// В MainActivity или Application
class MyApp : Application() {
    override fun onCreate() {
        super.onCreate()
        FirebaseApp.initializeApp(this)
        Log.d("Firebase", "Initialized: ${FirebaseApp.getInstance().name}")
    }
}
```

## **Важные моменты:**

### **1. Минимальные зависимости:**
```gradle
// Минимум для старта:
dependencies {
    implementation platform('com.google.firebase:firebase-bom:32.4.0')
    implementation 'com.google.firebase:firebase-analytics' // Обязательно для сбора данных
}
```

### **2. Для разных сервисов:**
```gradle
// Добавлять по необходимости:
implementation 'com.google.firebase:firebase-auth'        // Аутентификация
implementation 'com.google.firebase:firebase-firestore'   // База данных
implementation 'com.google.firebase:firebase-storage'     // Файлы
implementation 'com.google.firebase:firebase-messaging'   // Push
implementation 'com.google.firebase:firebase-crashlytics' // Краши
implementation 'com.google.firebase:firebase-config'      // Remote Config
```

### **3. Для Kotlin:**
```gradle
plugins {
    id 'com.android.application'
    id 'kotlin-android'
    id 'com.google.gms.google-services'  // plugin В КОНЦЕ списка
    id 'kotlin-kapt'  // Если нужно
}
```

### **4. Если возникает ошибка:**
- Проверить версию `google-services` в classpath
- Убедиться, что `google-services.json` в правильной папке
- Проверить package name в файле и в build.gradle

## **Пример минимальной конфигурации:**

**project/build.gradle:**
```gradle
buildscript {
    dependencies {
        classpath 'com.android.tools.build:gradle:8.1.0'
        classpath 'com.google.gms:google-services:4.4.0'
    }
}
```

**app/build.gradle:**
```gradle
plugins {
    id 'com.android.application'
    id 'com.google.gms.google-services'
}

android {
    namespace 'com.example.myapp'
    // остальная конфигурация
}

dependencies {
    implementation platform('com.google.firebase:firebase-bom:32.4.0')
    implementation 'com.google.firebase:firebase-analytics'
}
```

## **Проверка успешного подключения:**

1. Запустить приложение
2. В Firebase Console → Project Overview
3. Должна появиться статистика за 1 активного пользователя
4. В логах Android Studio: `FirebaseApp initialization successful`

## **Итог:**
1. **Создать** проект в консоли Firebase
2. **Добавить** Android приложение
3. **Скачать** `google-services.json` → в `app/`
4. **Добавить** classpath и plugin в Gradle
5. **Добавить** нужные Firebase зависимости
6. **Синхронизировать** проект

**Важно:** Добавлять только те сервисы Firebase, которые реально нужны в приложении!

  </details>

<details>
  <summary> Для чего нужен файл google-services.xml </summary>

  **`google-services.json` (или `.xml` для старых проектов)** — это конфигурационный файл Firebase для конкретного приложения.

---

## **Для чего нужен:**

### **1. Содержит настройки Firebase-проекта:**
```json
{
  "project_info": {
    "project_number": "123456789012",      // Номер проекта (Sender ID)
    "firebase_url": "https://...",         // URL базы данных
    "project_id": "myapp-12345",          // ID проекта
    "storage_bucket": "myapp-12345.appspot.com"
  },
  "client": [
    {
      "client_info": {
        "mobilesdk_app_id": "1:123456789012:android:abcdef", // App ID
        "android_client_info": {
          "package_name": "com.example.myapp"               // Ваш package name
        }
      },
      "oauth_client": [],          // Для Google Sign-In
      "api_key": [                 // API ключи
        {
          "current_key": "AIzaSyB...123"  // Главный ключ
        }
      ],
      "services": {
        "analytics_service": {...},
        "appinvite_service": {...}
      }
    }
  ]
}
```

### **2. Что хранит:**
- **Project ID** — идентификатор вашего Firebase-проекта
- **API Keys** — ключи для доступа к Firebase API
- **Package name** — имя пакета вашего приложения
- **App ID** — уникальный ID приложения в Firebase
- **Database URL** — адрес Realtime Database
- **Storage bucket** — адрес Cloud Storage
- **Sender ID** — для FCM (push-уведомлений)
- **OAuth clients** — для Google Sign-In

---

## **Как работает:**

### **При сборке проекта:**
1. **Gradle plugin** `com.google.gms.google-services` читает `google-services.json`
2. **Генерирует** XML/Java ресурсы с настройками:
   - `values/values.xml` с конфигурацией
   - `AndroidManifest.xml` добавляет метаданные
3. **Инжектирует** настройки в приложение

### **Сгенерированные файлы:**
```
app/build/generated/res/google-services/{buildType}/values/values.xml
```
Содержит:
```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string name="firebase_database_url" 
            translatable="false">https://myapp-12345.firebaseio.com</string>
    <string name="gcm_defaultSenderId" 
            translatable="false">123456789012</string>
    <string name="google_api_key" 
            translatable="false">AIzaSyB...123</string>
    <string name="google_app_id" 
            translatable="false">1:123456789012:android:abcdef</string>
    <string name="google_crash_reporting_api_key"
            translatable="false">AIzaSyB...123</string>
    <string name="google_storage_bucket"
            translatable="false">myapp-12345.appspot.com</string>
    <string name="project_id" 
            translatable="false">myapp-12345</string>
</resources>
```

---

## **Почему не хардкодить настройки:**

### **❌ ПЛОХО (хардкод в коде):**
```kotlin
class FirebaseConfig {
    companion object {
        const val API_KEY = "AIzaSyB...123"
        const val PROJECT_ID = "myapp-12345"
        const val DATABASE_URL = "https://myapp-12345.firebaseio.com"
        // Проблемы:
        // 1. Ключи в коде (security risk)
        // 2. Разные настройки для разных билдов
        // 3. Сложно менять
    }
}
```

### **✅ ХОРОШО (через google-services.json):**
- **Безопасность** — ключи не в исходном коде
- **Разные конфиги** для разных билдов (debug/release)
- **Автоматическое обновление** при изменении в Firebase Console
- **Централизованное управление** — один файл для всех настроек

---

## **Работа с разными окружениями:**

### **Разные файлы для debug/release:**
```
app/
├── google-services.json          // Production (release)
└── src/
    ├── debug/
    │   └── google-services.json  // Debug (тестовый проект)
    └── release/
        └── google-services.json  // Release (продакшен)
```

### **Или через flavors:**
```
app/
└── src/
    ├── dev/
    │   └── google-services.json  // Development
    ├── staging/
    │   └── google-services.json  // Staging
    └── prod/
        └── google-services.json  // Production
```

---

## **Важные моменты:**

### **1. Не коммитить в git?**
- **НЕ коммитить** если проект публичный
- **МОЖНО коммитить** для приватных репозиториев
- **Лучше:** Использовать CI/CD переменные

### **2. Если файл потерян:**
1. Зайти в Firebase Console
2. Project Settings → Your apps
3. Скачать `google-services.json` заново

### **3. Для нескольких приложений:**
- Каждому приложению в Firebase — свой файл
- Можно несколько в одном проекте (com.myapp, com.myapp.debug)

### **4. Миграция проекта:**
При смене package name:
1. Добавить новое приложение в Firebase Console
2. Скачать новый `google-services.json`
3. Заменить старый файл

---

## **Пример использования в коде:**

```kotlin
// Firebase автоматически читает настройки из google-services.json
// Вам не нужно делать это вручную!

class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate()
        
        // Firebase уже знает все настройки из google-services.json
        val db = Firebase.firestore // URL уже настроен
        val auth = Firebase.auth    // API key уже есть
        val storage = Firebase.storage // Bucket уже указан
        
        // Все работает без хардкода настроек!
    }
}
```

---

## **Итог для Junior:**

**`google-services.json` нужен для:**
1. **Хранения** всех настроек Firebase в одном месте
2. **Безопасности** — ключи не в исходном коде
3. **Упрощения** — не нужно конфигурировать Firebase вручную
4. **Гибкости** — разные настройки для разных билдов
5. **Синхронизации** — изменения в Firebase Console автоматически применяются

**Просто положите файл в `app/` и добавьте Gradle plugin — Firebase сам все настроит!**


</details>

<details>
  <summary> Какая зависимость необходима для подключения пушей? Как реализовать обработку пуша? </summary>

  ## **Для подключения пушей (FCM) нужны зависимости:**

### **1. Основные зависимости в `build.gradle`:**
```gradle
dependencies {
    // Firebase BoM для управления версиями
    implementation platform('com.google.firebase:firebase-bom:32.4.0')
    
    // Обязательные для FCM:
    implementation 'com.google.firebase:firebase-messaging'
    implementation 'com.google.firebase:firebase-analytics' // Рекомендуется
    
    // Для работы с уведомлениями (опционально):
    implementation 'androidx.core:core-ktx:1.10.1'
}
```

---

## **Реализация обработки пушей:**

### **Шаг 1: Создать сервис для обработки сообщений**
```kotlin
import com.google.firebase.messaging.FirebaseMessagingService
import com.google.firebase.messaging.RemoteMessage

class MyFirebaseMessagingService : FirebaseMessagingService() {

    // 1. Вызывается при получении нового FCM токена
    override fun onNewToken(token: String) {
        Log.d("FCM", "New token: $token")
        // Отправить токен на ваш сервер
        sendTokenToServer(token)
    }

    // 2. Вызывается при получении push-уведомления
    override fun onMessageReceived(message: RemoteMessage) {
        Log.d("FCM", "Message received")
        
        // Проверяем тип сообщения
        message.notification?.let { notification ->
            // Уведомление с заголовком и телом
            showNotification(
                notification.title ?: "Новое уведомление",
                notification.body ?: "",
                message.data // дополнительные данные
            )
        }
        
        // Или данные без уведомления
        message.data.let { data ->
            if (data.isNotEmpty()) {
                handleDataMessage(data)
            }
        }
    }
    
    private fun showNotification(title: String, body: String, data: Map<String, String>) {
        // Создаем канал уведомлений (для Android 8.0+)
        createNotificationChannel()
        
        // Создаем Intent для открытия приложения
        val intent = Intent(this, MainActivity::class.java).apply {
            flags = Intent.FLAG_ACTIVITY_NEW_TASK or Intent.FLAG_ACTIVITY_CLEAR_TASK
            putExtra("data", data["key"]) // передаем данные
        }
        
        val pendingIntent = PendingIntent.getActivity(
            this, 0, intent,
            PendingIntent.FLAG_UPDATE_CURRENT or PendingIntent.FLAG_IMMUTABLE
        )
        
        // Строим уведомление
        val notification = NotificationCompat.Builder(this, "fcm_channel")
            .setContentTitle(title)
            .setContentText(body)
            .setSmallIcon(R.drawable.ic_notification)
            .setContentIntent(pendingIntent)
            .setAutoCancel(true)
            .setPriority(NotificationCompat.PRIORITY_HIGH)
            .build()
        
        // Показываем уведомление
        NotificationManagerCompat.from(this)
            .notify(System.currentTimeMillis().toInt(), notification)
    }
    
    private fun createNotificationChannel() {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            val channel = NotificationChannel(
                "fcm_channel",
                "Уведомления",
                NotificationManager.IMPORTANCE_HIGH
            ).apply {
                description = "Канал для push-уведомлений"
            }
            
            val manager = getSystemService(NotificationManager::class.java)
            manager.createNotificationChannel(channel)
        }
    }
    
    private fun handleDataMessage(data: Map<String, String>) {
        // Обработка данных (без показа уведомления)
        when (data["type"]) {
            "sync" -> syncData()
            "update" -> checkForUpdates()
        }
    }
    
    private fun sendTokenToServer(token: String) {
        // Отправка токена на ваш бэкенд
        // Например через Retrofit или Firestore
        Log.d("FCM", "Token to server: $token")
    }
}
```

### **Шаг 2: Объявить сервис в `AndroidManifest.xml`**
```xml
<service
    android:name=".MyFirebaseMessagingService"
    android:exported="false">
    <intent-filter>
        <action android:name="com.google.firebase.MESSAGING_EVENT" />
    </intent-filter>
</service>

<!-- Для Android 13+ (API 33) нужно разрешение -->
<uses-permission android:name="android.permission.POST_NOTIFICATIONS" />
```

### **Шаг 3: Запросить разрешения (для Android 13+)**
```kotlin
class MainActivity : AppCompatActivity() {
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        // Запрос разрешения на уведомления (Android 13+)
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU) {
            if (ContextCompat.checkSelfPermission(
                    this,
                    Manifest.permission.POST_NOTIFICATIONS
                ) != PackageManager.PERMISSION_GRANTED
            ) {
                ActivityCompat.requestPermissions(
                    this,
                    arrayOf(Manifest.permission.POST_NOTIFICATIONS),
                    REQUEST_NOTIFICATION_PERMISSION
                )
            }
        }
        
        // Получить текущий FCM токен
        FirebaseMessaging.getInstance().token.addOnCompleteListener { task ->
            if (task.isSuccessful) {
                val token = task.result
                Log.d("FCM", "Current token: $token")
            }
        }
    }
}
```

### **Шаг 4: Подписаться на темы (опционально)**
```kotlin
// Подписаться на тему
FirebaseMessaging.getInstance().subscribeToTopic("news")
    .addOnCompleteListener { task ->
        if (task.isSuccessful) {
            Log.d("FCM", "Subscribed to news topic")
        }
    }

// Отписаться от темы
FirebaseMessaging.getInstance().unsubscribeFromTopic("news")
```

---

## **Типы push-сообщений:**

### **1. Notification messages (авто-уведомления):**
```json
{
  "to": "token...",
  "notification": {
    "title": "Заголовок",
    "body": "Текст уведомления",
    "icon": "ic_notification"
  }
}
```
- **Показываются автоматически** если приложение в бэкграунде
- **`onMessageReceived` вызывается** если приложение на переднем плане

### **2. Data messages (данные):**
```json
{
  "to": "token...",
  "data": {
    "type": "message",
    "user": "Иван",
    "text": "Привет!"
  }
}
```
- **Не показываются автоматически**
- **Всегда попадают в `onMessageReceived`**
- **Нужно обрабатывать вручную**

### **3. Комбинированные:**
```json
{
  "to": "token...",
  "notification": {
    "title": "Новое сообщение"
  },
  "data": {
    "chat_id": "123",
    "message_id": "456"
  }
}
```

---

## **Обработка нажатия на уведомление:**

### **В Activity:**
```kotlin
class MainActivity : AppCompatActivity() {
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        // Проверяем, открыли ли приложение по нажатию на уведомление
        if (intent.extras != null) {
            for (key in intent.extras!!.keySet()) {
                val value = intent.extras!![key]
                Log.d("FCM", "Key: $key Value: $value")
            }
            
            // Получаем данные из уведомления
            val data = intent.getStringExtra("data")
            if (data != null) {
                // Показываем соответствующий экран
                showNotificationScreen(data)
            }
        }
    }
}
```

---

## **Важные моменты:**

### **1. Проверка разрешений:**
```kotlin
// Проверяем, включены ли уведомления
val areNotificationsEnabled = NotificationManagerCompat
    .from(context)
    .areNotificationsEnabled()

if (!areNotificationsEnabled) {
    // Показать диалог о включении уведомлений
    showEnableNotificationsDialog()
}
```

### **2. Обработка в бэкграунде:**
- Если приложение **закрыто** → уведомление показывается в статус-баре
- Если приложение **в бэкграунде** → уведомление показывается в статус-баре
- Если приложение **на переднем плане** → `onMessageReceived` вызывается сразу

### **3. Лимиты FCM:**
- Размер сообщения: **4KB** для всех данных
- Частота отправки: лимиты для бесплатного тарифа

---

## **Итог для Junior:**

### **Что нужно сделать:**
1. **Добавить зависимости** `firebase-messaging` и `firebase-analytics`
2. **Создать сервис**, наследующий `FirebaseMessagingService`
3. **Реализовать** `onNewToken()` и `onMessageReceived()`
4. **Объявить сервис** в манифесте
5. **Запросить разрешения** для Android 13+
6. **Получить токен** и отправить на сервер

### **Ключевые методы:**
- `onNewToken()` — для получения/обновления токена устройства
- `onMessageReceived()` — для обработки пришедших сообщений
- `FirebaseMessaging.getInstance().token` — получить текущий токен

**Простой пример работает из коробки, кастомизация — в `onMessageReceived()`.**

</details>  

<details>
  <summary>Как осуществляется отправка ошибок в консоль Firebase? </summary>

  ## **Для отправки ошибок в Firebase Crashlytics:**

### **1. Добавить зависимость:**
```gradle
dependencies {
    implementation platform('com.google.firebase:firebase-bom:32.4.0')
    implementation 'com.google.firebase:firebase-crashlytics'
}
```

### **2. Инициализация (автоматическая):**
Crashlytics включается автоматически при добавлении зависимости и `google-services.json`.

---

## **Отправка ошибок:**

### **Автоматически (краши):**
```kotlin
// Все необработанные исключения автоматически отправляются
fun riskyFunction() {
    // Если произойдет исключение - Crashlytics отправит его автоматически
    val result = 10 / 0 // ArithmeticException
}
```

### **Вручную (нефатальные ошибки):**
```kotlin
try {
    // Код, который может вызвать ошибку
    processUserData()
} catch (e: Exception) {
    // Отправляем ошибку вручную
    FirebaseCrashlytics.getInstance().recordException(e)
    
    // Можно добавить дополнительную информацию
    FirebaseCrashlytics.getInstance().log("Ошибка при обработке данных пользователя")
    
    // Показываем пользователю сообщение
    showErrorToast("Что-то пошло не так")
}
```

---

## **Добавление дополнительной информации:**

### **1. Пользовательские ключи:**
```kotlin
// Устанавливаем ключи для контекста ошибки
val crashlytics = FirebaseCrashlytics.getInstance()

crashlytics.setCustomKey("user_id", userId)
crashlytics.setCustomKey("app_version", BuildConfig.VERSION_NAME)
crashlytics.setCustomKey("android_version", Build.VERSION.SDK_INT)
crashlytics.setCustomKey("device_model", Build.MODEL)

// В случае ошибки эти ключи будут прикреплены к отчету
```

### **2. Логирование:**
```kotlin
// Добавляем логи для отслеживания потока выполнения
FirebaseCrashlytics.getInstance().log("Начало загрузки данных")
// ... код ...
FirebaseCrashlytics.getInstance().log("Данные загружены успешно")
// ... код ...
FirebaseCrashlytics.getInstance().log("Ошибка парсинга JSON")
```

---

## **Полный пример:**
```kotlin
class DataProcessor {
    fun processData(userId: String, json: String) {
        val crashlytics = FirebaseCrashlytics.getInstance()
        
        // Устанавливаем контекст
        crashlytics.setCustomKey("user_id", userId)
        crashlytics.setCustomKey("data_size", json.length)
        crashlytics.log("Начало обработки данных для пользователя: $userId")
        
        try {
            // Парсим JSON
            val data = JSONObject(json)
            crashlytics.log("JSON успешно распарсен")
            
            // Обрабатываем данные
            val result = complexProcessing(data)
            crashlytics.log("Обработка завершена успешно")
            
        } catch (e: JSONException) {
            // Ошибка парсинга
            crashlytics.log("Ошибка парсинга JSON: ${e.message}")
            crashlytics.setCustomKey("json_error", "invalid_format")
            crashlytics.recordException(e)
            
        } catch (e: IOException) {
            // Сетевая ошибка
            crashlytics.log("IO ошибка: ${e.message}")
            crashlytics.recordException(e)
            
        } catch (e: Exception) {
            // Любая другая ошибка
            crashlytics.log("Неизвестная ошибка: ${e.message}")
            crashlytics.recordException(e)
            throw e // Бросаем дальше для автоматического краш-репорта
        }
    }
}
```

---

## **Настройка для разных сборок:**

### **В `build.gradle`:**
```gradle
android {
    buildTypes {
        debug {
            // Отключаем Crashlytics для debug сборок
            firebaseCrashlytics {
                mappingFileUploadEnabled false
            }
        }
        release {
            // Включаем для release
            firebaseCrashlytics {
                nativeSymbolUploadEnabled true
                unstrippedNativeLibsDir 'path/to/unstripped/libs'
            }
        }
    }
}
```

### **Программное отключение:**
```kotlin
// Отключить Crashlytics (например, для debug)
FirebaseCrashlytics.getInstance().setCrashlyticsCollectionEnabled(false)

// Включить обратно
FirebaseCrashlytics.getInstance().setCrashlyticsCollectionEnabled(true)
```

---

## **Просмотр ошибок в консоли:**

1. **Зайти в** [Firebase Console](https://console.firebase.google.com)
2. **Выбрать проект** → **Crashlytics** в меню слева
3. **Видеть:**
   - Количество крашей
   - Стек трейсы
   - Пользовательские ключи и логи
   - Устройства и версии ОС

---

## **Важные моменты:**

### **1. Для NDK (нативные краши):**
```gradle
android {
    buildTypes {
        release {
            firebaseCrashlytics {
                nativeSymbolUploadEnabled true
                unstrippedNativeLibsDir 'build/intermediates/merged_native_libs/release/out/lib'
            }
        }
    }
}
```

### **2. Проверка работы:**
```kotlin
// Тестовая ошибка для проверки
fun testCrashlytics() {
    // Можно добавить кнопку для теста
    FirebaseCrashlytics.getInstance().log("Тест Crashlytics")
    throw RuntimeException("Тестовая ошибка для Crashlytics")
}
```

### **3. Ограничения:**
- Данные об ошибках появляются через **5-10 минут**
- Бесплатный тариф: **без ограничений** по количеству отчетов
- Сохранение: **90 дней** истории

---

## **Итог для Junior:**

### **Что нужно сделать:**
1. **Добавить** `firebase-crashlytics` зависимость
2. **Автоматически** отправляются все необработанные исключения
3. **Вручную** отправлять через `recordException()`
4. **Добавлять контекст** через `setCustomKey()` и `log()`

### **Ключевые методы:**
- `recordException(e)` — отправить ошибку
- `setCustomKey(key, value)` — добавить контекст
- `log(message)` — добавить лог
- `setCrashlyticsCollectionEnabled()` — включить/выключить

**Просто добавьте зависимость — краши начнут отправляться автоматически!** 

  <details>
  <summary> Что содержит в себе google-services.xml </summary>

  **В Android проектах используется `google-services.json`, а не `.xml`. XML версия — устаревший формат для очень старых проектов.**

---

## **Что содержит `google-services.json`:**

### **1. Основные разделы файла:**
```json
{
  "project_info": {
    "project_number": "123456789012",      // Номер проекта (Sender ID для FCM)
    "firebase_url": "https://myapp-12345.firebaseio.com", // Realtime DB URL
    "project_id": "myapp-12345",          // Уникальный ID проекта
    "storage_bucket": "myapp-12345.appspot.com" // Cloud Storage bucket
  },
  
  "client": [
    {
      "client_info": {
        "mobilesdk_app_id": "1:123456789012:android:abcdef123456", // App ID
        "android_client_info": {
          "package_name": "com.example.myapp" // Ваш package name
        }
      },
      "oauth_client": [ // Для Google Sign-In
        {
          "client_id": "123456-abcdef.apps.googleusercontent.com",
          "client_type": 3
        }
      ],
      "api_key": [ // API ключи
        {
          "current_key": "AIzaSyB...123" // Главный API ключ
        }
      ],
      "services": { // Конфигурация сервисов
        "analytics_service": {
          "status": 1
        },
        "appinvite_service": {
          "status": 2,
          "other_platform_oauth_client": []
        },
        "ads_service": {
          "status": 2
        }
      }
    }
  ],
  
  "configuration_version": "1" // Версия конфигурации
}
```

---

## **Ключевые поля:**

### **Для разработчика важны:**
1. **`project_number`** (Sender ID) — для FCM push-уведомлений
2. **`project_id`** — уникальный ID Firebase проекта
3. **`mobilesdk_app_id`** — ID приложения в Firebase
4. **`package_name`** — имя пакета вашего приложения
5. **`current_key`** — API ключ для доступа к Firebase
6. **`firebase_url`** — URL Realtime Database
7. **`storage_bucket`** — URL Cloud Storage

---

## **Как используется Gradle plugin'ом:**

При сборке проекта plugin:
1. **Читает** `google-services.json`
2. **Генерирует** `values.xml` в `build/generated/res/`:
```xml
<resources>
    <string name="firebase_database_url">https://myapp-12345.firebaseio.com</string>
    <string name="gcm_defaultSenderId">123456789012</string>
    <string name="google_api_key">AIzaSyB...123</string>
    <string name="google_app_id">1:123456789012:android:abcdef</string>
    <string name="google_storage_bucket">myapp-12345.appspot.com</string>
    <string name="project_id">myapp-12345</string>
</resources>
```

3. **Добавляет** в `AndroidManifest.xml`:
```xml
<meta-data
    android:name="com.google.android.gms.version"
    android:value="@integer/google_play_services_version" />
```

---

## **Для разных сборок (debug/release):**

Можно иметь разные файлы:
```
app/
├── src/
│   ├── debug/google-services.json     // Для debug
│   └── release/google-services.json   // Для release
```

Или один файл, но с разными конфигурациями для разных приложений в Firebase Console.

---

## **Итог для Junior:**

**`google-services.json` содержит:**
1. **Идентификаторы** проекта и приложения
2. **API ключи** для доступа к Firebase
3. **URL адреса** сервисов (DB, Storage)
4. **Конфигурацию** OAuth для Google Sign-In
5. **Настройки** подключенных сервисов

**Не храните его в публичных репозиториях** — содержит API ключи!

  </details

  <details>
  <summary> Для чего необходим Channel в пушах? </summary>  

  ## **Notification Channels (каналы уведомлений) нужны для Android 8.0 (API 26) и выше.**

---

## **Зачем нужны:**

### **1. Управление пользователем:**
Пользователь может **настроить каждую категорию уведомлений отдельно**:
- Звук/вибрация
- Важность (priority)
- Показ на экране блокировки
- Badge icon (цифра на иконке)

### **2. Пример из настроек телефона:**
```
Уведомления приложения "МоеПриложение"
├── 📢 Новости (можно отключить)
├── 🔔 Сообщения (можно изменить звук)
├── 💰 Акции (можно отключить вибрацию)
└── 🎯 Важные (показывать на заблокированном экране)
```

---

## **Как создать канал:**

### **Обязательно для Android 8.0+:**
```kotlin
fun createNotificationChannel() {
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
        // 1. Создаем канал
        val channel = NotificationChannel(
            "news_channel",          // ID канала (уникальный)
            "Новости",               // Название (видит пользователь)
            NotificationManager.IMPORTANCE_HIGH // Важность
        ).apply {
            description = "Новости и обновления" // Описание
            enableLights(true)       // Светодиод
            lightColor = Color.RED   // Цвет светодиода
            enableVibration(true)    // Вибрация
            vibrationPattern = longArrayOf(100, 200, 300) // Паттерн вибрации
        }
        
        // 2. Регистрируем канал в системе
        val manager = getSystemService(NotificationManager::class.java)
        manager.createNotificationChannel(channel)
    }
}
```

---

## **Типы важности (importance):**

| IMPORTANCE | Описание | Поведение |
|------------|----------|-----------|
| **HIGH** | Высокая | Показывается везде, может прерывать |
| **DEFAULT** | Средняя | Звук, но не прерывает |
| **LOW** | Низкая | Без звука, в шторке |
| **MIN** | Минимальная | Только в шторке, без звука/вибрации |
| **NONE** | Без уведомления | Скрыто |

---

## **Использование в push-уведомлении:**

```kotlin
fun showNotification(title: String, body: String) {
    // Создаем канал (если еще не создан)
    createNotificationChannel()
    
    // Строим уведомление с указанием канала
    val notification = NotificationCompat.Builder(this, "news_channel") // ← ID канала
        .setContentTitle(title)
        .setContentText(body)
        .setSmallIcon(R.drawable.ic_notification)
        .setPriority(NotificationCompat.PRIORITY_HIGH)
        .setAutoCancel(true)
        .build()
    
    // Показываем
    NotificationManagerCompat.from(this).notify(1, notification)
}
```

---

## **Для FCM (Firebase Cloud Messaging):**

### **В сервисе обработки пушей:**
```kotlin
class MyFirebaseMessagingService : FirebaseMessagingService() {
    
    override fun onMessageReceived(message: RemoteMessage) {
        // Определяем тип уведомления для выбора канала
        val channelId = when (message.data["type"]) {
            "news" -> "news_channel"
            "message" -> "messages_channel"
            "promo" -> "promo_channel"
            else -> "default_channel"
        }
        
        showNotification(channelId, message)
    }
    
    private fun showNotification(channelId: String, message: RemoteMessage) {
        // Канал должен быть создан заранее!
        val notification = NotificationCompat.Builder(this, channelId)
            .setContentTitle(message.notification?.title)
            .setContentText(message.notification?.body)
            .setSmallIcon(R.drawable.ic_notification)
            .build()
        
        NotificationManagerCompat.from(this).notify(1, notification)
    }
}
```

---

## **Лучшие практики:**

### **1. Создавать каналы при запуске приложения:**
```kotlin
class MyApp : Application() {
    override fun onCreate() {
        super.onCreate()
        createNotificationChannels()
    }
    
    private fun createNotificationChannels() {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            // Основные каналы
            createChannel("messages", "Сообщения", 
                "Уведомления о новых сообщениях", 
                NotificationManager.IMPORTANCE_HIGH)
            
            createChannel("news", "Новости",
                "Новости и обновления приложения",
                NotificationManager.IMPORTANCE_DEFAULT)
            
            createChannel("promo", "Акции",
                "Специальные предложения",
                NotificationManager.IMPORTANCE_LOW)
        }
    }
}
```

### **2. Не создавать слишком много каналов:**
- **Оптимально:** 3-5 каналов
- **Максимум:** 8-10 (больше — пользователь запутается)

### **3. Не менять настройки каналов после создания:**
После создания канала **нельзя изменить** важность и другие настройки.
Можно только удалить и создать заново.

---

## **Проверка и управление:**

### **Проверить существование канала:**
```kotlin
fun isChannelExists(channelId: String): Boolean {
    val manager = getSystemService(NotificationManager::class.java)
    
    return if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
        manager.getNotificationChannel(channelId) != null
    } else {
        true // Для старых версий каналов нет
    }
}
```

### **Удалить канал:**
```kotlin
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
    val manager = getSystemService(NotificationManager::class.java)
    manager.deleteNotificationChannel("old_channel_id")
}
```

---

## **Для старых версий Android (< 8.0):**

Для API < 26 каналы **не нужны**, но код должен быть обратно совместимым:
```kotlin
fun showNotification(title: String, body: String) {
    val builder = NotificationCompat.Builder(this, 
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            "channel_id" // Для Android 8.0+
        } else {
            "" // Для старых версий
        }
    )
    // остальной код...
}
```

---

## **Итог для Junior:**

**Notification Channels нужны для:**
1. **Управления** — пользователь настраивает разные типы уведомлений
2. **Организации** — разделение уведомлений по категориям
3. **Кастомизации** — разный звук/вибрация для разных типов
4. **Требование Android** — начиная с API 26 **обязательны**

**Обязательно:**
- Создавать каналы при первом запуске
- Указывать channel ID при показе уведомления
- Для Android < 8.0 — код должен работать без каналов

**Без канала на Android 8.0+ уведомления НЕ ПОКАЖУТСЯ!**

  </details


</details> 

</details>

<details> 
  <summary> <h2> 🌿 Middle </h2> </summary>

  <details>
  <summary> </summary>

  </details>


   <details>
  <summary>  </summary>

  </details>

  
</details>


<details> 
  <summary> <h2> 🌳 Senior </h2> </summary>

  <details>
  <summary> </summary>

  </details>

  <details>
  <summary>  </summary>

  </details>

  </details>
  
</details>

-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
**Ранее**

- []()
- 
**Далее**
- []()
