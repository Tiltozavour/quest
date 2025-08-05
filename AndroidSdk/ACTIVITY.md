<details>
  <summary> <h2> 🌱 Junior </h2> </summary>

<details>
<summary> Какими способами можно запустить активити (implicit/explicit интенты)? </summary>

В Android **Activity** можно запустить двумя способами с помощью **Intent**:

---

### 1. **Explicit Intent (явный)**  
Указывает **точное имя класса** Activity, которую нужно открыть.  
Используется **внутри одного приложения**.

```kotlin
val intent = Intent(this, MainActivity::class.java)
startActivity(intent)
```

✅ Когда использовать:  
— Переход между экранами в своём приложении.

---

### 2. **Implicit Intent (неявный)**  
Описывает **действие (action)** и **данные (data)**, а система сама выбирает подходящую Activity.  
Может запускать экраны **в других приложениях**.

```kotlin
val intent = Intent(Intent.ACTION_VIEW, Uri.parse("https://google.com"))
startActivity(intent)
```

✅ Примеры:
- Открыть ссылку — браузер.
- Поделиться текстом — выбор приложения (Telegram, WhatsApp).
- Выбрать фото — галерея.

---

### Важно:
- Для **своих активити** — всегда **explicit**.
- Для **внешних действий** — **implicit**, но перед запуском стоит проверить, есть ли обработчик:
  ```kotlin
  if (intent.resolveActivity(packageManager) != null) {
      startActivity(intent)
  }
  ```

> ⚠️ Implicit-интенты работают через **фильтры намерений (intent-filters)** в `AndroidManifest.xml` других приложений.
  
</details>

<details> 
<summary> Что такое жизненный цикл в общих словах?</summary>

**Жизненный цикл** — это последовательность состояний, через которые проходит компонент (например, Activity или Fragment) от создания до уничтожения.

В Android он определяется системой и зависит от действий пользователя и состояния приложения (например, переход на другой экран, сворачивание, пересоздание при повороте).

### Зачем нужен:
- Правильно управлять ресурсами (запуск/остановка потоков, подписок).
- Сохранять состояние при пересоздании.
- Избегать утечек памяти и ошибок.

### Пример (Activity):
```
onCreate() → onStart() → onResume() → [работа] → onPause() → onStop() → onDestroy()
```

- `onCreate()` — инициализация UI.
- `onResume()` — активность видима и активна.
- `onPause()` / `onStop()` — при переходе в фон.

> **Ключевое:** разработчик должен корректно обрабатывать каждый этап, чтобы приложение работало стабильно и экономно.

</details>

<details> 
<summary> Перечислить основные методы жц </summary>
Вот основные методы **жизненного цикла Activity** в порядке вызова:

```kotlin
onCreate()        // Создание Activity: инициализация UI, данных
onStart()         // Activity становится видимой (но ещё не активной)
onResume()        // Activity готова к взаимодействию с пользователем
onPause()         // Activity теряет фокус (частично перекрыта) — сохранить данные, остановить анимации
onStop()          // Activity больше не видима — освободить ресурсы
onDestroy()       // Activity уничтожается (вызван finish() или система)
onRestart()       // Вызывается перед onStart(), если Activity возвращается из остановленного состояния
```

---

### Дополнительно (для Fragment):
```kotlin
onAttach()        // Привязка к Activity
onCreate()        // Создание фрагмента
onCreateView()    // Создание View
onViewCreated()   // View создана, можно инициализировать
onStart()         // Фрагмент становится видимым
onResume()        // Фрагмент активен
onPause()         // Потеря фокуса
onStop()          // Перестал быть видимым
onDestroyView()   // View уничтожена (но фрагмент жив)
onDestroy()       // Фрагмент уничтожается
onDetach()        // Отвязка от Activity
```

> ⚠️ Все операции должны соответствовать этапу:  
> — Инициализация — в `onCreate()`,  
> — Работа с UI — после `onResume()`,  
> — Освобождение — в `onPause()`/`onStop()`/`onDestroyView()`.
</details>

<details> 
  <summary>Почему нужно прописывать активити в манифесте? </summary>

**Activity нужно прописывать в `AndroidManifest.xml`**, потому что:

1. **Система Android должна знать о её существовании** — иначе не сможет её запустить (даже с `Intent`).
2. **Только объявленные Activity доступны для запуска** — система проверяет манифест перед созданием.
3. Можно задать **intent-filters** — например, для обработки ссылок или `ACTION_SEND`.
4. Указывается **launchMode**, разрешения, тема (`theme`), экземплярность.
5. Обеспечивается **безопасность** — система контролирует, какие компоненты могут быть вызваны извне.

> 🔒 Если Activity **не в манифесте** — при запуске будет `ActivityNotFoundException`.

### Пример:
```xml
<activity
    android:name=".MainActivity"
    android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity>
```

> ⚠️ С Android 12+ (`API 31`) **все Activity, используемые в приложении, обязательно должны быть объявлены в манифесте**.
 
</details>


<details> 
<summary>Какая активити считается стартовой и должна запускаться при клике на иконку приложения? (ACTION_MAIN, CATEGORY_LAUNCHER)</summary>

Стартовой считается **Activity, у которой в `AndroidManifest.xml` указан фильтр намерений**:

```xml
<intent-filter>
    <action android:name="android.intent.action.MAIN" />
    <category android:name="android.intent.category.LAUNCHER" />
</intent-filter>
```

### Что это значит:
- `ACTION_MAIN` — точка входа в приложение.
- `CATEGORY_LAUNCHER` — отображается в лаунчере (списке приложений).

Такая Activity **появляется в списке приложений** и запускается при клике на иконку.

### Важно:
- Должна быть **только одна** такая Activity в приложении (иначе — несколько иконок).
- Обычно это `MainActivity` или `SplashActivity`.
- Атрибут `android:exported="true"` обязателен для запуска извне (например, лаунчера).

```xml
<activity
    android:name=".MainActivity"
    android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity>
```

> ⚠️ Без `exported="true"` Activity **не запустится** из лаунчера, даже с правильным intent-filter.

</details>

<details> 
  <summary> Можно ли изменить стартовую активность программно? </summary>

  **Нет, стартовую Activity нельзя изменить программно в runtime.**

Причина: система определяет стартовую Activity **на этапе установки приложения**, на основе `AndroidManifest.xml`. Только одна Activity с `ACTION_MAIN` + `CATEGORY_LAUNCHER` становится точкой входа.

---

### Но можно **управлять логикой запуска**:

#### ✅ Решение: использовать **SplashActivity** или **RouterActivity**
Создать временную стартовую Activity, которая **программно решает**, какую Activity показать дальше.

Пример:
```kotlin
class SplashActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        val intent = if (UserPrefs.isLoggedIn(this)) {
            Intent(this, MainActivity::class.java)
        } else {
            Intent(this, LoginActivity::class.java)
        }
        startActivity(intent)
        finish()
    }
}
```

И в манифесте:
```xml
<activity
    android:name=".SplashActivity"
    android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity>
```

---

### Вывод:
- **Прямое изменение стартовой Activity — невозможно.**
- **Косвенное управление** — легко реализуется через промежуточную Activity (сплэш или роутер).

> Это стандартный паттерн в Android-приложениях.

</details>

<details> 
  <summary> Чем отличаются Activity и Fragment? </summary>

**Activity** и **Fragment** — ключевые UI-компоненты в Android, но с разными целями и возможностями:

| Характеристика | **Activity** | **Fragment** |
|----------------|-------------|--------------|
| **Уровень абстракции** | Полный экран (или окно) | Часть экрана (UI-модуль) |
| **Жизненный цикл** | Управляется системой напрямую | Зависит от родительской Activity |
| **Переиспользование** | Сложно переиспользовать | Легко использовать в разных Activity (например, на телефоне и планшете) |
| **Навигация** | Переключение между экранами | Упрощает навигацию внутри Activity (через `FragmentManager`) |
| **Разделение логики** | Всё в одном классе — может быть громоздко | Позволяет разделять UI на независимые блоки |
| **Состояние при повороте** | Пересоздаётся полностью | Может сохранять состояние через `ViewModel` или `onSaveInstanceState()` |
| **Зависимость** | Самостоятельный компонент | Существует **только внутри Activity** |

### Когда что использовать:
- **Activity** — для независимых экранов (например, вход, главный экран, настройки).
- **Fragment** — для гибкой верстки, особенно под планшеты и сложные интерфейсы (например, мастер-деталь).

> 💡 **Современный подход**: один `MainActivity` + множество `Fragment` + `NavController` — гибкая и поддерживаемая архитектура.

**Коротко**:  
Activity — **экран**, Fragment — **блок интерфейса**.
  
</details>

<details> 
  <summary>Как управлять жизненным циклом Activity?</summary>

**Управлять жизненным циклом Activity** — значит корректно реагировать на его методы, чтобы приложение работало стабильно и эффективно.

### Как управлять:
1. **Переопределяй методы жизненного цикла** в нужных местах:
   ```kotlin
   override fun onCreate(savedInstanceState: Bundle?) {
       super.onCreate(savedInstanceState)
       // Инициализация: UI, данные, ViewModel
   }

   override fun onResume() {
       super.onResume()
       // Начать обновление UI, запустить таймеры, подписаться на события
   }

   override fun onPause() {
       super.onPause()
       // Остановить анимации, отписаться от событий, сохранить данные
   }
   ```

2. **Не выполняй тяжелые операции в главном потоке**, особенно в `onCreate()` и `onResume()`.

3. **Освобождай ресурсы** в соответствующих методах:
   - Камера, сенсоры — в `onPause()` или `onStop()`.
   - Локальные подписки (например, `LocationManager`) — в `onPause()`.

4. **Используй `ViewModel` и `LiveData`** — они переживают пересоздание Activity и помогают не терять данные.

5. **Сохраняй состояние**:
   - Кратковременное — `onSaveInstanceState()` / `onRestoreInstanceState()`.
   - Долгосрочное — `ViewModel`, `SharedPreferences`, БД.

6. **Подписывайся/отписывайся** от событий (RxJava, Flow, BroadcastReceiver) в правильных этапах, чтобы избежать утечек памяти.

---

### Пример:
```kotlin
override fun onResume() {
    super.onResume()
    locationManager.requestLocationUpdates(listener)
}

override fun onPause() {
    super.onPause()
    locationManager.removeUpdates(listener)
}
```

> ✅ **Главное**: не "управлять" вмешательством, а **реагировать на системные вызовы** — Android сам управляет жизненным циклом. Твоя задача — корректно на него реагировать.
  
</details>

<details>
  <summary> Как сохранить состояние Activity при повороте экрана?</summary>

  При повороте экрана **Activity пересоздаётся**, поэтому важно сохранить состояние. Есть несколько способов:

---

### 1. **`onSaveInstanceState()` и `onRestoreInstanceState()`**  
Для **временных данных** (например, текст в поле, позиция скролла).

```kotlin
override fun onSaveInstanceState(outState: Bundle) {
    super.onSaveInstanceState(outState)
    outState.putString("key", editText.text.toString())
}

override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    val text = savedInstanceState?.getString("key", "")
    editText.setText(text)
}
```

✅ Подходит: для UI-состояния.  
❌ Не подходит: для больших объектов, потоков, ресурсов.

---

### 2. **ViewModel** (рекомендуется)  
`ViewModel` **переживает пересоздание Activity** и идеален для логики и данных.

```kotlin
class MyViewModel : ViewModel() {
    val userData = MutableLiveData<String>()
}
```

В Activity:
```kotlin
val viewModel = ViewModelProvider(this)[MyViewModel::class.java]
viewModel.userData.observe(this) { /* обновить UI */ }
```

✅ Главный способ в современной разработке.

---

### 3. **Сохранение в постоянное хранилище**  
Для данных, которые нужно сохранить даже после закрытия приложения:
- `SharedPreferences`
- Локальная БД (Room)
- Файлы

---

### ⚠️ Что НЕ использовать:
- Статические поля — могут вызвать утечки, не гарантируют сохранность.
- `onRetainNonConfigurationInstance()` — устарело, заменено на `ViewModel`.

---

### Итог:
- **UI-состояние** → `onSaveInstanceState()`
- **Бизнес-данные и логика** → **`ViewModel`**
- **Долгосрочные данные** → `SharedPreferences`, `Room`

> ✅ Лучшая практика: **ViewModel + onSaveInstanceState при необходимости**.
  
</details>

<details>
  <summary>Как обработать нажатие кнопки "Назад" в Activity/Fragment</summary>

Чтобы обработать нажатие кнопки "Назад", нужно переопределить метод `onBackPressed()`.

---

### 🔹 В **Activity**:
```kotlin
override fun onBackPressed() {
    // Своя логика: показ диалога, выход из режима и т.п.
    if (inSelectionMode) {
        exitSelectionMode()
    } else {
        super.onBackPressed() // стандартное поведение — выход
    }
}
```

> ✅ Используется для:
> - Подтверждения выхода.
> - Закрытия навигационного меню.
> - Выхода из полноэкранного режима.

---

### 🔹 В **Fragment**:
Fragment **не имеет** `onBackPressed()`, поэтому есть два способа:

#### 1. **Через Activity (рекомендуется):**
Fragment уведомляет Activity, а тот решает, как реагировать:
```kotlin
// Во фрагменте
(activity as? OnBackPressedListener)?.onBackPressed()

// Или через интерфейс
interface OnBackPressedListener {
    fun onBackPressed()
}
```

#### 2. **Использовать `OnBackPressedDispatcher` (современный способ):**
```kotlin
override fun onAttach(context: Context) {
    super.onAttach(context)
    requireActivity().onBackPressedDispatcher.addCallback(this) {
        // Своя логика
        if (shouldHandle) {
            // Обработать нажатие
        } else {
            // Разрешить стандартное поведение
            isEnabled = false
            requireActivity().onBackPressed()
        }
    }
}
```

> ✅ `OnBackPressedDispatcher` — часть **Navigation Component**, работает корректно с `NavController`.

---

### Итог:
- В **Activity** — `onBackPressed()`.
- Во **Fragment** — использовать `OnBackPressedDispatcher` или делегировать Activity.
- Всегда вызывай `super.onBackPressed()` или управляй `isEnabled`, чтобы не заблокировать навигацию.
  
</details>

</details>

<details>
  <summary> <h2> 🌿 Middle </h2> </summary>

<details>
  <summary>Жизненный цикл подробно (какие методы есть, в каком порядке и в каком случае вызываются)</summary>

  ### 🟦 Жизненный цикл **Activity** — подробно

Когда пользователь взаимодействует с приложением, система вызывает серию методов **в строгом порядке**. Вот основные состояния и методы:

---

#### 🔹 1. **Создание Activity**
Вызывается при первом запуске или пересоздании (например, при повороте).

```kotlin
onCreate()     → Инициализация: setContentView(), ViewModel, данные
onStart()      → Activity становится видимой (но не активной)
onResume()     → Activity активна, пользователь может с ней взаимодействовать
```

> ✅ Вызывается при старте или восстановлении из `onStop()`.

---

#### 🔹 2. **Переход в фон**
Пользователь ушёл в другое приложение или свернул текущее.

```kotlin
onPause()      → Activity теряет фокус (частично перекрыта)
onStop()       → Activity больше не видима
```

> ⚠️ `onPause()` должен быть быстрым — следующее приложение не запустится, пока он не завершится.

---

#### 🔹 3. **Возврат в Activity**
Пользователь вернулся к приложению.

```kotlin
onRestart()    → Вызывается только если был onStop()
onStart()      → Снова видима
onResume()     → Снова активна
```

> ❗ `onRestart()` вызывается **только** после `onStop()`, но **не после `onPause()`** (например, при вызове диалога).

---

#### 🔹 4. **Уничтожение Activity**
```kotlin
onPause() → onStop() → onDestroy()
```

Вызывается при:
- `finish()` (программное закрытие),
- системном уничтожении (нехватка памяти),
- пересоздании (например, смена языка/ориентации).

> 💡 При повороте: `onDestroy()` → `onCreate()` (если не настроен `configChanges`).

---

### 🟨 Жизненный цикл **Fragment** (связан с Activity)

```kotlin
onAttach()           → Привязка к Activity
onCreate()           → Создание фрагмента (до UI)
onCreateView()       → Создание View
onViewCreated()      → View создана, можно инициализировать (например, `findViewById`)
onStart()            → Фрагмент и View видимы
onResume()           → Фрагмент активен
onPause()            → Потеря фокуса
onStop()             → Перестал быть видимым
onDestroyView()      → View уничтожена (но фрагмент жив — например, при замене)
onDestroy()          → Фрагмент уничтожается
onDetach()           → Отвязка от Activity
```

> ⚠️ `onDestroyView()` — ключевой для очистки UI-ссылок (чтобы избежать утечек памяти).

---

### 📌 Когда что использовать:

| Метод | Назначение |
|------|-----------|
| `onCreate()` | Инициализация данных, ViewModel |
| `onCreateView()` | Инфлейт разметки |
| `onViewCreated()` | Настройка UI (кнопки, адаптеры) |
| `onResume()` | Начать обновление (например, сенсор, локация) |
| `onPause()` | Остановить ресурсы (камера, подписки) |
| `onDestroyView()` | Очистить View-ссылки (`view = null`) |
| `onDestroy()` / `onDetach()` | Освободить ресурсы, отписаться от глобальных событий |

---

### ✅ Важно:
- Все операции должны соответствовать этапу.
- Не делать тяжёлые операции в `onCreate()` и `onResume()`.
- Использовать `ViewModel` для сохранения данных между пересозданиями.

> Это база для стабильной и производительной работы Android-приложения.
  
</details>

<details>
  <summary>Почему не рекомендуется менять имя активити после публикации приложения?</summary>
  
  **Менять имя Activity (или удалять её) после публикации приложения не рекомендуется**, потому что это может **сломать работу приложения** для уже установленных пользователей.

---

### Основные причины:

#### 1. **Нарушение deep link / push-уведомлений**
Если на Activity ссылаются:
- **Deep link** (например, `intent-filter` для URL),
- **Push-уведомления** (от сервера),
- **Ярлыки на рабочем столе** (pinned shortcuts),

…то при переименовании Activity **интенты перестанут находить нужный экран** → `ActivityNotFoundException`.

#### 2. **Сломается task-стек**
Android сохраняет стек активити между сессиями. Если пользователь свернул приложение, а потом обновил его с переименованной Activity — система не сможет восстановить стек.

#### 3. **Нарушение экспортированных компонентов**
Если Activity была `exported=true` (доступна извне), и на неё ссылаются другие приложения (например, через intent), изменение имени **сломает интеграцию**.

#### 4. **Потеря сохранённого состояния**
Некоторые библиотеки или кэши могут ссылаться на Activity по полному имени (`package.ClassName`). Её изменение нарушит восстановление состояния.

---

### Что делать, если нужно изменить?
✅ **Правильный способ — оставить старую Activity как прокладку:**
```kotlin
class OldActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        // Перенаправляем на новую
        startActivity(Intent(this, NewActivity::class.java))
        finish()
    }
}
```

И только потом удалять её в следующей версии.

---

### Вывод:
> 🔒 **Имя Activity — это часть публичного API приложения.**  
> Его изменение может сломать ссылки, пуш-уведомления и пользовательский опыт.  
> Лучше **переименовывать с миграцией**, а не резко удалять.
</details>
  
</details>  

-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
**Ранее**
- [Base_Component](BASE_COMPONENT.md)

**Далее**
- [Fragment](FRAGMENT.md)

**Оглавление**
- [Readme](README.md)

