
# Notifications

<details>
  <summary> <h2> 🌱 Junior </h2> </summary>

<details>
  <summary> Что такое notification в Android? </summary>

Уведомление (notification) — это сообщение, которое Android отображает вне пользовательского интерфейса вашего приложения. Его цель — предоставить пользователю краткую информацию о событиях (сообщениях, напоминаниях и т.д.) и быстрый доступ к приложению.

</details>

<details>
  <summary> Как создать notification? </summary>

 Для создания уведомления используется класс NotificationCompat.Builder (для обратной совместимости) и NotificationManager.

1. Cоздаем канал для уведомлений
```
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
    val channel = NotificationChannel(
        "channel_id",
        "Channel Name",
        NotificationManager.IMPORTANCE_DEFAULT
    ).apply {
        description = "Channel Description"
    }
    val notificationManager: NotificationManager = getSystemService()
    notificationManager.createNotificationChannel(channel)
}
```
2. Создаем уведомление
```
val notification = NotificationCompat.Builder(this, "channel_id")
    .setSmallIcon(R.drawable.ic_notification)
    .setContentTitle("Title")
    .setContentText("Text")
    .setPriority(NotificationCompat.PRIORITY_DEFAULT)
    .build()
```
3. Показываем уведомление
```
notificationManager.notify(notificationId, notification)
```

</details>

</details>

<details> 
  <summary> <h2> 🌿 Middle </h2> </summary>

  <details>
  <summary> Расскажите из чего состоит уведомление? Какие параметры можно настроить? </summary>

Уведомление состоит из:

- **Small icon** (обязательно) — маленькая иконка в статус-баре.
- **Title & Text** — заголовок и основной текст.
- **Content Intent** — PendingIntent для перехода в приложение по тапу.
- **Channel ID** (API 26+) — обязательный, определяет поведение уведомления (звук, вибрация и т.д.).
- **Priority / Importance** — приоритет отображения (до/после API 26).
- **Large icon** — большая иконка слева (например, аватар).
- **Actions** — кнопки действий (до 3).
- **Grouping** — группировка уведомлений.
- **Badge / Number** — счетчик на иконке приложения.
- **Auto-cancel** — скрыть после тапа.
- **Sound / Vibration / Lights** — настраивается через канал.
- **Custom layout** — через `RemoteViews` (осторожно с производительностью).

Настройка — через `NotificationCompat.Builder`, каналы — через `NotificationChannel`.

  </details>


   <details>
  <summary> Как создать каналов уведомлений и управлять ими? Какие уровни важности можно установить? </summary>


**Создание канала:**

```kotlin
val channel = NotificationChannel(
    "channel_id",
    "Channel Name",
    NotificationManager.IMPORTANCE_DEFAULT
).apply {
    description = "Channel description"
    enableVibration(true)
    setSound(soundUri, audioAttributes)
    // другие настройки: свет, вибрация, блокировка на экране и т.д.
}

val manager = getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager
manager.createNotificationChannel(channel)
```

**Управление каналами:**
- Изменить нельзя — только **пересоздать** (удалить → создать заново), но пользовательские настройки сохраняются.
- Удалить: `manager.deleteNotificationChannel("channel_id")`
- Получить список: `manager.notificationChannels`

**Уровни важности (IMPORTANCE):**

- `IMPORTANCE_NONE` — нет уведомлений (тихо).
- `IMPORTANCE_MIN` — без звука, в статус-баре.
- `IMPORTANCE_LOW` — без звука, в шторке.
- `IMPORTANCE_DEFAULT` — звук по умолчанию.
- `IMPORTANCE_HIGH` — звук + всплывающее уведомление (heads-up).
- `IMPORTANCE_MAX` — высший приоритет (редко используется).

> Важность задаётся **при создании канала** и влияет на поведение: звук, вибрация, heads-up, показ на экране блокировки.

  </details>  
  
</details>


<details> 
  <summary> <h2> 🌳 Senior </h2> </summary>

  <details>
  <summary> Как можно настроить группы уведомлений? </summary>

Группировка — объединение нескольких уведомлений в одно свёрнутое с раскрывающимся списком.

---

**Как настроить:**

1. **Задать группу** через `setGroup("group_key")` у каждого уведомления.
2. **Создать summary-уведомление** — родительское, которое отображается в шторке:

```kotlin
.setGroup("messages_group")
.setGroupSummary(true) // только для summary
```

3. **Для Android 7.0+ (API 24+)** — система сама делает свёртку, summary можно не показывать.
4. **Для старых версий** — summary обязателен, иначе группировка не работает.

---

**Дополнительно:**

- Можно использовать `InboxStyle`, `MessagingStyle` для красивого отображения внутри группы.
- `setGroupAlertBehavior()` — настраивает, какие уведомления в группе издают звук (только summary, все, или только не-summary).
- Группы работают **в пределах одного пакета приложения**.

---

**Важно:**
- Не перегружать группу — свыше 7–10 уведомлений система может обрезать.
- Summary-уведомление должно содержать агрегированную информацию (например, “5 новых сообщений”).

Готов к следующему.

  </details>

  <details>
  <summary> В чем заключается особенность push notifications в Foreground и Background. Как отобразить пуш уведомление если оно пришло в Foreground? </summary>

---

**Особенности:**

- **Background / Killed**:  
  Система автоматически показывает уведомление, если в payload есть `notification` (от FCM).  
  → **Нет контроля** — отображается как есть.

- **Foreground**:  
  Приходит только `onMessageReceived()`, **уведомление НЕ показывается автоматически** — нужно отобразить вручную через `NotificationManager`.

---

**Как отобразить пуш в Foreground:**

1. Унаследуй `FirebaseMessagingService`.
2. Переопредели `onMessageReceived(remoteMessage: RemoteMessage)`:

```kotlin
override fun onMessageReceived(remoteMessage: RemoteMessage) {
    if (remoteMessage.notification != null) {
        val title = remoteMessage.notification?.title
        val body = remoteMessage.notification?.body

        val notification = NotificationCompat.Builder(this, "channel_id")
            .setContentTitle(title)
            .setContentText(body)
            .setSmallIcon(R.drawable.ic_notification)
            .setContentIntent(pendingIntent)
            .build()

        val manager = getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager
        manager.notify(notificationId, notification)
    }
}
```

---

**Важно:**

- Если хочешь **единое поведение** — отправляй **только data-пуш**, и обрабатывай вручную везде.
- Для кастомизации (иконки, действия, группы) — только ручной показ.
- В Foreground можно показать Heads-up, даже если канал не High Importance — если приложение активно.

---

**Pro совет:**  
Используй `RemoteMessage.data` вместо `notification` — полный контроль на клиенте.


  </details>

  </details>
  
</details>

-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
**Ранее**

- [OnSavedState](ONSAVEDSTATE.md)
- 
**Далее**
- []()
