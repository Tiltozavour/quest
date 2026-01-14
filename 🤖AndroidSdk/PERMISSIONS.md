
# Permissions

<details>
  <summary> <h2> 🌱 Junior </h2> </summary>

<details>
  <summary> Какие типы разрешений ты знаешь? как их запрашивать? </summary>
## Типы разрешений:

1. **Обычные (Normal)** – автоматически выдаются при установке (интернет, вибрация).
2. **Опасные (Dangerous)** – требуют явного запроса пользователя во время работы приложения (камера, геолокация, контакты).
3. **Особые (Special)** – например, `SYSTEM_ALERT_WINDOW` (поверх других окон) или `WRITE_SETTINGS` — запрашиваются через Intent отдельно.

## Как запрашивать опасные разрешения:

1. **Объявить в манифесте:**
```xml
<uses-permission android:name="android.permission.CAMERA" />
```

2. **Проверить наличие разрешения во время выполнения:**
```kotlin
if (ContextCompat.checkSelfPermission(this, Manifest.permission.CAMERA) 
    != PackageManager.PERMISSION_GRANTED) {
    // Разрешение не дано
}
```

3. **Запросить разрешение:**
```kotlin
ActivityCompat.requestPermissions(
    this, 
    arrayOf(Manifest.permission.CAMERA), 
    REQUEST_CODE
)
```

4. **Обработать ответ пользователя:**
```kotlin
override fun onRequestPermissionsResult(...) {
    if (grantResults.isNotEmpty() && grantResults[0] == PERMISSION_GRANTED) {
        // Разрешение дано
    }
}
```

</details>

<details>
  <summary> Какие разрешения для телефонов в которых нет камеры?/ как проверить если на телефоне камера? </summary>

  ## Проверка наличия камеры

**1. Проверить в манифесте:**
```xml
<uses-feature 
    android:name="android.hardware.camera"
    android:required="false" />
```
- `android:required="false"` — приложение можно устанавливать на устройства без камеры
- `android:required="true"` — установка только на устройства с камерой

**2. Проверить программно:**
```kotlin
fun hasCamera(context: Context): Boolean {
    return context.packageManager.hasSystemFeature(PackageManager.FEATURE_CAMERA_ANY)
    // или FEATURE_CAMERA для задней камеры
    // или FEATURE_CAMERA_FRONT для фронтальной
}
```

**3. Условная обработка:**
```kotlin
if (hasCamera(this)) {
    // Запрашиваем разрешение CAMERA
    requestCameraPermission()
} else {
    // Показываем сообщение или скрываем функционал камеры
    showNoCameraMessage()
}
```

**Важно:** Даже если камеры нет, разрешение `CAMERA` в манифесте может блокировать установку на некоторых устройствах, если не указано `android:required="false"`.


</details>

<details>
  <summary> Расскажи как правильно запрашивать runtime permissions? </summary>

  ## Правильный алгоритм запроса runtime permissions:

**1. Проверить, нужно ли объяснение:**
```kotlin
if (ContextCompat.checkSelfPermission(context, permission) 
    != PackageManager.PERMISSION_GRANTED) {
    
    if (ActivityCompat.shouldShowRequestPermissionRationale(activity, permission)) {
        // Пользователь уже отказывал или просит объяснить "зачем"
        showExplanationDialog()
    } else {
        // Запрашиваем впервые
        requestPermission()
    }
} else {
    // Разрешение уже есть
    executeFeature()
}
```

**2. Показывать диалог объяснения (если нужно):**
Объяснять, зачем нужно разрешение, только если `shouldShowRequestPermissionRationale` возвращает `true`.

**3. Запрашивать разрешение:**
```kotlin
ActivityCompat.requestPermissions(
    activity,
    arrayOf(Manifest.permission.CAMERA, Manifest.permission.RECORD_AUDIO),
    REQUEST_CODE
)
```

**4. Обрабатывать результат:**
```kotlin
override fun onRequestPermissionsResult(requestCode: Int, 
                                        permissions: Array<String>,
                                        grantResults: IntArray) {
    if (requestCode == REQUEST_CODE) {
        if (grantResults.isNotEmpty() && grantResults[0] == PERMISSION_GRANTED) {
            // Разрешение получено
            executeFeature()
        } else {
            // Пользователь отказал
            handleDenial()
        }
    }
}
```

**5. Обрабатывать "Навсегда запретить":**
Если пользователь выбрал "Don't ask again" (больше не спрашивать):
- `shouldShowRequestPermissionRationale()` вернет `false`
- `checkSelfPermission()` вернет `DENIED`
В этом случае нужно перенаправить в настройки:
```kotlin
val intent = Intent(Settings.ACTION_APPLICATION_DETAILS_SETTINGS)
intent.data = Uri.fromParts("package", packageName, null)
startActivity(intent)
```

**Важные моменты:**
- Запрашивайте только необходимые разрешения
- Группируйте логически связанные разрешения
- Не запрашивайте несколько раз подряд
- Предоставляйте fallback-функционал при отказе

</details>

<details>
  <summary> Что нужно делать если пользователь отменяет или отклоняет разрешение? </summary>

  ## Действия при отказе в разрешении:

**1. При первом отказе:**
- `shouldShowRequestPermissionRationale()` вернет `true`
- Показать объяснение, зачем нужно разрешение
- Предложить запросить снова (но не сразу)

**2. При повторном отказе или "Навсегда запретить":**
- `shouldShowRequestPermissionRationale()` вернет `false`
- Нельзя запрашивать снова автоматически
- Нужно направить в настройки приложения:

```kotlin
private fun showSettingsDialog() {
    AlertDialog.Builder(this)
        .setTitle("Нужно разрешение")
        .setMessage("Разрешение отклонено навсегда. Включите в настройках")
        .setPositiveButton("Настройки") { _, _ ->
            val intent = Intent(Settings.ACTION_APPLICATION_DETAILS_SETTINGS)
            intent.data = Uri.fromParts("package", packageName, null)
            startActivity(intent)
        }
        .setNegativeButton("Отмена", null)
        .show()
}
```

**3. Общий алгоритм обработки:**
```kotlin
fun handlePermissionDenied() {
    if (!ActivityCompat.shouldShowRequestPermissionRationale(this, permission)) {
        // "Навсегда запретить" - показываем диалог с настройками
        showSettingsDialog()
    } else {
        // Просто отклонил - показываем объяснение
        showRationaleDialog()
    }
}
```

**4. Что НЕ делать:**
- Не запрашивать разрешение сразу после отказа
- Не блокировать весь функционал приложения
- Не показывать агрессивные сообщения

**5. Что делать вместо этого:**
- Предоставить альтернативный функционал
- Скрыть элементы, требующие разрешения
- Показать кнопку "повторить попытку" в удобном месте
- Вежливо объяснить последствия отказа

**Пример fallback:**
```kotlin
if (!hasCameraPermission()) {
    cameraButton.isVisible = false
    uploadFromGalleryButton.isVisible = true
    showInfoMessage("Можно загрузить фото из галереи")
}
```


</details>

<details>
  <summary> Уменьшает ли группа разрешений количество системных диалогов?  </summary>

Начиная с Android **6.0 (API 23)** и выше:

1. **Группы разрешений** существуют только на уровне документации для пользователя
2. **Система показывает один диалог на запрос**, независимо от количества разрешений в группе
3. Если запрашиваете несколько разрешений из одной группы одновременно — они все покажутся в одном диалоге

**Пример:**
```kotlin
// Один диалог с двумя разрешениями
requestPermissions(
    arrayOf(
        Manifest.permission.READ_CONTACTS,
        Manifest.permission.WRITE_CONTACTS
    ), 
    REQUEST_CODE
)
```
Пользователь увидит **один диалог** с двумя переключателями (для Android 10 и ниже) или описанием доступа к контактам (Android 11+).

**Важно:**
- На **Android 10 и ниже**: показываются отдельные переключатели для каждого разрешения в группе
- На **Android 11 (API 30) и выше**: система сама определяет группу и показывает общее описание доступа (например, "Доступ к контактам")
- **Количество системных диалогов = 1** на вызов `requestPermissions()`, а не на количество разрешений

**Вывод:** Группировка разрешений в одном вызове `requestPermissions()` действительно уменьшает количество диалогов до одного.

</details>

<details>
  <summary> Какие методы в активити для получения результата разрешения </summary>

  ## Методы для работы с runtime permissions в Activity:

**1. `requestPermissions()`** — запрос разрешений:
```kotlin
requestPermissions(
    arrayOf(Manifest.permission.CAMERA),
    REQUEST_CODE
)
// Или через ActivityCompat для обратной совместимости
ActivityCompat.requestPermissions(...)
```

**2. `onRequestPermissionsResult()`** — обработка ответа пользователя:
```kotlin
override fun onRequestPermissionsResult(
    requestCode: Int,
    permissions: Array<String>,
    grantResults: IntArray
) {
    super.onRequestPermissionsResult(requestCode, permissions, grantResults)
    
    if (requestCode == REQUEST_CODE) {
        if (grantResults.isNotEmpty() && grantResults[0] == 
            PackageManager.PERMISSION_GRANTED) {
            // Разрешение получено
        } else {
            // Разрешение отклонено
        }
    }
}
```

**3. `shouldShowRequestPermissionRationale()`** — проверка, нужно ли объяснять:
```kotlin
if (shouldShowRequestPermissionRationale(Manifest.permission.CAMERA)) {
    // Пользователь уже отказывал - показать объяснение
    showExplanation()
}
```

**4. Вспомогательные методы из `ActivityCompat`:**
```kotlin
// Проверка разрешения
ActivityCompat.checkSelfPermission(this, permission)

// Запрос с учетом обратной совместимости
ActivityCompat.requestPermissions(...)

// Проверка needShowRequestPermissionRationale с совместимостью
ActivityCompat.shouldShowRequestPermissionRationale(...)
```

**5. Пример полной цепочки:**
```kotlin
class MainActivity : AppCompatActivity() {
    private val REQUEST_CODE = 100
    
    fun checkCameraPermission() {
        if (checkSelfPermission(Manifest.permission.CAMERA) != 
            PackageManager.PERMISSION_GRANTED) {
            
            requestPermissions(arrayOf(Manifest.permission.CAMERA), REQUEST_CODE)
        } else {
            // Разрешение уже есть
        }
    }
    
    override fun onRequestPermissionsResult(requestCode: Int, 
                                            permissions: Array<String>,
                                            grantResults: IntArray) {
        // Обработка результата
    }
}
```

**Важно:** Для Fragment используется почти такой же API, но вызывается через `fragment.requestPermissions()` и обрабатывается во фрагменте.

</details>


</details>

<details> 
  <summary> <h2> 🌿 Middle </h2> </summary>

  <details>
  <summary>  Как отключить функцию, которая используется в приложении, но приложение не зависит от нее при помощи разрешений? </summary> 

  ## Использовать `<uses-permission>` с `android:maxSdkVersion`

В **AndroidManifest.xml** можно ограничить запрос разрешения для определенных версий Android:

```xml
<uses-permission 
    android:name="android.permission.WRITE_EXTERNAL_STORAGE"
    android:maxSdkVersion="18" />
```

## Как это работает:

**1. Для старых устройств (API ≤ 18):**
- Разрешение запрашивается как обычно
- Функция работает

**2. Для новых устройств (API > 18):**
- Разрешение **не запрашивается** вообще
- Система игнорирует это `<uses-permission>`
- Функция, требующая разрешения, становится недоступной

## Пример отключения устаревшей функции:

```xml
<!-- Функция нужна только для API 28 и ниже -->
<uses-permission 
    android:name="android.permission.READ_CALL_LOG"
    android:maxSdkVersion="28" />
```

## Альтернативный подход — условная логика в коде:

```kotlin
fun useLegacyFeature() {
    // Проверяем версию Android
    if (Build.VERSION.SDK_INT <= Build.VERSION_CODES.P) {
        // Используем старую функцию с разрешением
        if (hasPermission(Manifest.permission.READ_CALL_LOG)) {
            readCallLogLegacy()
        }
    } else {
        // Используем новую функцию без разрешения
        readCallLogModern()
    }
}
```

## Преимущества подхода с `maxSdkVersion`:

1. **Не загромождает диалоги разрешений** на новых устройствах
2. **Упрощает код** — не нужно проверять версию SDK перед запросом
3. **Соответствует рекомендациям Google** для обратной совместимости
4. **Пользователи видят только актуальные разрешения**

## Важные моменты:

- `android:maxSdkVersion` работает только с `<uses-permission>`, не с `<uses-feature>`
- При обновлении приложения старое разрешение удаляется с устройств, где оно больше не нужно
- Всегда предоставляйте альтернативную реализацию без разрешения для новых API

  </details>
</details>


<details> 
  <summary> <h2> 🌳 Senior </h2> </summary>

  <details>
  <summary> Как создать custom разрешения? и как его объявить? </summary>

  ## Создание custom разрешений в Android

### 1. **Объявление в AndroidManifest.xml:**

```xml
<permission
    android:name="com.example.app.CUSTOM_PERMISSION"
    android:description="@string/custom_permission_description"
    android:icon="@drawable/ic_permission_icon"
    android:label="@string/custom_permission_label"
    android:protectionLevel="dangerous" />
```

### 2. **Ключевые атрибуты:**

- **`android:name`** — уникальный идентификатор (рекомендуется prefix с именем пакета)
- **`android:protectionLevel`** — определяет политику доступа:
  - `normal` — автоматически выдаётся
  - `dangerous` — требует явного согласия пользователя
  - `signature` — только для приложений, подписанных тем же ключом
  - `signatureOrSystem` — только системные приложения или с тем же ключом (устарело)
- **`android:permissionGroup`** — группа для системного диалога (опционально)

### 3. **Запрос custom разрешения:**

Аналогично системным разрешениям:
```kotlin
if (checkSelfPermission("com.example.app.CUSTOM_PERMISSION") 
    != PackageManager.PERMISSION_GRANTED) {
    
    requestPermissions(
        arrayOf("com.example.app.CUSTOM_PERMISSION"),
        REQUEST_CODE
    )
}
```

### 4. **Проверка в коде перед вызовом защищённого компонента:**

```kotlin
fun performSecureAction() {
    if (checkSelfPermission("com.example.app.CUSTOM_PERMISSION") 
        == PackageManager.PERMISSION_GRANTED) {
        // Выполняем действие
    } else {
        // Запрашиваем разрешение
    }
}
```

### 5. **Защита компонентов приложения custom разрешением:**

В манифесте на компоненте:
```xml
<activity
    android:name=".SecureActivity"
    android:permission="com.example.app.CUSTOM_PERMISSION" />

<receiver
    android:name=".SecureBroadcastReceiver"
    android:permission="com.example.app.CUSTOM_PERMISSION" />

<service
    android:name=".SecureService"
    android:permission="com.example.app.CUSTOM_PERMISSION" />

<provider
    android:name=".SecureContentProvider"
    android:permission="com.example.app.CUSTOM_PERMISSION"
    android:exported="true" />
```

### 6. **Особенности и best practices:**

**a) Наследование разрешений:**
```xml
<!-- Разрешение с более строгими требованиями -->
<permission
    android:name="com.example.app.ADVANCED_PERMISSION"
    android:protectionLevel="signature"
    android:permission="com.example.app.CUSTOM_PERMISSION" />
```

**b) Группировка custom разрешений:**
```xml
<permission-group
    android:name="com.example.app.CUSTOM_PERMISSION_GROUP"
    android:label="@string/permission_group_label"
    android:description="@string/permission_group_description" />

<permission
    android:name="com.example.app.CUSTOM_PERMISSION"
    android:permissionGroup="com.example.app.CUSTOM_PERMISSION_GROUP"
    android:protectionLevel="dangerous" />
```

**c) Проверка перед использованием:**
```kotlin
fun isCustomPermissionGranted(): Boolean {
    return try {
        checkSelfPermission("com.example.app.CUSTOM_PERMISSION") == 
            PackageManager.PERMISSION_GRANTED
    } catch (e: SecurityException) {
        false // Разрешение не объявлено в манифесте вызывающего приложения
    }
}
```

### 7. **Ограничения и предупреждения:**

⚠️ **Системные ограничения:**
- Custom разрешения удаляются при удалении приложения, которое их объявило
- На Android 8.0+ нельзя изменить `protectionLevel` после публикации приложения
- Система кэширует разрешения — изменения могут требовать перезагрузки

⚠️ **Безопасность:**
- Не полагайтесь только на custom разрешения для критической безопасности
- Используйте `signature` уровень для межпроцессного взаимодействия между своими приложениями
- Всегда проверяйте разрешения на стороне защищаемого компонента

### 8. **Полный пример использования:**

**App A (объявляет разрешение):**
```xml
<permission
    android:name="com.app.a.SHARE_DATA"
    android:label="Access to shared data"
    android:protectionLevel="dangerous" />

<provider
    android:name=".DataProvider"
    android:authorities="com.app.a.provider"
    android:permission="com.app.a.SHARE_DATA"
    android:exported="true" />
```

**App B (использует разрешение):**
```xml
<uses-permission android:name="com.app.a.SHARE_DATA" />
```

```kotlin
// В App B запрашиваем разрешение
if (checkSelfPermission("com.app.a.SHARE_DATA") 
    != PackageManager.PERMISSION_GRANTED) {
    requestPermissions(arrayOf("com.app.a.SHARE_DATA"), REQUEST_CODE)
} else {
    // Доступ к DataProvider App A
    contentResolver.query(Uri.parse("content://com.app.a.provider/data"))
}
```

### 9. **Альтернативы custom разрешениям:**
- Использовать `android:exported="false"` для внутренних компонентов
- Проверка подписи приложения для доверенного взаимодействия
- ContentProvider с dynamic permission checks

**Custom разрешения полезны для:**
- Межпроцессного взаимодействия между своими приложениями
- Создания security-focused SDK
- Защиты внутренних API от несанкционированного доступа

  </details>


  </details>

  </details>
  
</details>

-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
**Ранее**

- []()
- 
**Далее**
- []()
