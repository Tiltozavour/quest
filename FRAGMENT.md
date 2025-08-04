# Fragments

<details>
  <summary> <h2> 🌱 Junior </h2> </summary>

<details> 
  <summary> Почему не стоит создавать конструкторы с параметрами для фрагментов? </summary>

**Создавать конструкторы с параметрами для `Fragment` не рекомендуется**, потому что **фрагмент может быть пересоздан системой** (например, при повороте экрана), и тогда он будет создан **через пустой (дефолтный) конструктор**.

Если у фрагмента нет пустого конструктора — приложение **упадёт с исключением**:
```
Can't instantiate fragment: java.lang.InstantiationException
```

---

### Почему так происходит?
Android **сериализует аргументы фрагмента** в `Bundle` и должен иметь возможность **воссоздать фрагмент** без участия разработчика — например, после убийства процесса.

---

### 🔁 Правильный способ передачи данных — `setArguments()`:

```kotlin
class UserFragment : Fragment() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        val userId = arguments?.getString("user_id")
    }

    companion object {
        fun newInstance(userId: String): UserFragment {
            return UserFragment().apply {
                arguments = Bundle().apply {
                    putString("user_id", userId)
                }
            }
        }
    }
}
```

### Преимущества:
- Аргументы сохраняются автоматически при пересоздании.
- Работает корректно с `ViewModel`, `NavController`, `FragmentManager`.
- Поддерживает безопасную передачу примитивов, `Parcelable`, `Serializable`.

---

### Вывод:
> ❌ Нельзя: `UserFragment("123")` — если нет пустого конструктора, будет ошибка.  
> ✅ Нужно: использовать **фабричный метод `newInstance()`** и **`arguments`**.

</details>

<details> 
<summary>  Как управлять жизненным циклом Fragment? / Перечислить основные методы жц </summary>

**Управление жизненным циклом Fragment** — это корректная реакция на его методы в нужный момент, с учётом связи с родительской Activity.

---

### 🔁 Основные методы жизненного цикла Fragment (в порядке вызова):

| Метод | Когда вызывается | Что делать |
|------|-------------------|-----------|
| `onAttach()` | Привязка к Activity | Инициализация, получение ссылки на Activity |
| `onCreate()` | Создание фрагмента (до UI) | Инициализация данных, ViewModel |
| `onCreateView()` | Создание View | `inflate` разметки, **не инициализировать логику** |
| `onViewCreated()` | View создана | Настроить UI: адаптеры, клики, `ViewModel` observers |
| `onStart()` | Фрагмент видим | Начать обновление данных (если нужно) |
| `onResume()` | Фрагмент активен | Запустить сенсоры, анимации, подписки |
| `onPause()` | Потеря фокуса | Остановить тяжёлые операции |
| `onStop()` | Не виден | Освободить ресурсы |
| `onDestroyView()` | View уничтожена | Очистить View-ссылки (например, `recyclerView = null`) |
| `onDestroy()` | Фрагмент уничтожается | Освободить ресурсы, отписаться от событий |
| `onDetach()` | Отвязка от Activity | Обнулить ссылку на Activity |

---

### ✅ Ключевые правила управления:

1. **Не создавать UI-логику в `onCreateView()`** — только инфлейт.
2. **В `onViewCreated()`** — инициализировать адаптеры, `LiveData`, обработчики кликов.
3. **В `onDestroyView()`** — **обязательно очищать ссылки на View**, чтобы избежать утечек памяти:
   ```kotlin
   override fun onDestroyView() {
       binding = null // если используется ViewBinding
       super.onDestroyView()
   }
   ```
4. **Работать с `ViewModel`** — он переживает пересоздание фрагмента.
5. **Не хранить тяжёлые ресурсы (камера, сенсоры) после `onPause()`** — освобождать в `onPause()` или `onStop()`.

---

### 🔄 Связь с Activity:
- Жизненный цикл Fragment **вложен** в жизненный цикл Activity.
- `onCreate()` Fragment вызывается **после** `onCreate()` Activity.
- `onResume()` Activity вызывается **после** `onResume()` всех фрагментов.

---

### Вывод:
> Управлять — значит **реагировать на этапы**, а не вмешиваться.  
> Главное:  
> - Инициализация в правильных методах,  
> - Очистка в `onDestroyView()` и `onDestroy()`,  
> - Безопасная работа с UI и ресурсами.

</details>

<details>
  <summary> Как добавить фрагмент в активность? </summary>

  Фрагмент добавляется в Activity с помощью **`FragmentManager`** и **`FragmentTransaction`**.

---

### ✅ Основные способы:

#### 1. **Добавление (add) — для стека фрагментов**
```kotlin
supportFragmentManager
    .beginTransaction()
    .add(R.id.container, MyFragment())
    .commit()
```
> Используется, когда нужно **сохранять предыдущие фрагменты** (например, при навигации).

---

#### 2. **Замена (replace) — стандартный способ**
```kotlin
supportFragmentManager
    .beginTransaction()
    .replace(R.id.container, MyFragment())
    .commit()
```
> Удаляет текущий фрагмент в контейнере и добавляет новый.

---

#### 3. **С сохранением в бэк-стек (для "Назад")**
```kotlin
supportFragmentManager
    .beginTransaction()
    .replace(R.id.container, MyFragment())
    .addToBackStack(null) // можно указать имя
    .commit()
```
> Позволяет возвращаться к предыдущему фрагменту по кнопке "Назад".

---

### 📌 Где размещать:
- Контейнер (`FrameLayout`, `FragmentContainerView`) в разметке Activity:
  ```xml
  <FrameLayout
      android:id="@+id/container"
      android:layout_width="match_parent"
      android:layout_height="match_parent" />
  ```

---

### 🔄 Дополнительно:
- **`commit()`** — применяет изменения асинхронно.
- **`commitNow()`** — синхронно (редко, может блокировать UI).
- Используй **фабричный метод `newInstance()`**, если фрагменту нужны аргументы.

---

### 💡 Современный подход:
Использовать **`NavController`** из **Navigation Component** — он сам управляет стеком:
```kotlin
findNavController().navigate(R.id.action_to_myFragment)
```

---

### Вывод:
> Основной способ — `replace()` + `commit()`.  
> Для навигации назад — добавляй в бэк-стек.  
> В новых проектах — лучше использовать **Navigation Component**.

</details>

</details>

<details> 
  <summary> <h2> 🌿 Middle </h2> </summary>

<details> 
<summary> Вызывается ли onPause без вызова onStop? Привести пример если да </summary>

  **Да, `onPause()` может вызываться без `onStop()`.**

### Когда это происходит:
Когда Activity **частично перекрывается другим окном**, но **остаётся частично видимой**.

В этом случае:
- `onPause()` — вызывается (потеряла фокус),
- `onStop()` — **не вызывается**, потому что Activity всё ещё **частично видна**.

---

### ✅ Пример:
Открытие **диалогового окна** (например, `DialogFragment` или системный диалог) **поверх Activity**.

```kotlin
// В Activity
val dialog = AlertDialog.Builder(this)
    .setTitle("Внимание")
    .show()
```

#### Что происходит:
1. `onPause()` — вызывается (Activity теряет фокус).
2. Диалог отображается — Activity приглушена, но **не скрыта полностью**.
3. `onStop()` — **не вызывается**.
4. Закрытие диалога → `onResume()` (минуя `onStart()` и `onStop()`).

---

### 🔁 Порядок вызова:
```
onResume() → onPause() → onResume()
```
(без `onStop()` и `onStart()`)

---

### Важно:
- `onPause()` **всегда** вызывается перед потерей фокуса.
- `onStop()` — **только когда Activity становится полностью невидимой**.

> ✅ Это стандартное поведение Android — важно учитывать при управлении ресурсами (например, не останавливать сенсор в `onPause()`, если он нужен в фоне).
</details>

<details> 
  <summary> Как сохранить данные в Fragment при повороте экрана? (Без использования ViewModel и/или Moxy) </summary>

  Если **нельзя использовать `ViewModel` или Moxy**, данные во **Fragment** при повороте экрана можно сохранить с помощью **`onSaveInstanceState()`**.

---

### ✅ Способ: `onSaveInstanceState()` и восстановление в `onViewCreated()` / `onCreate()`

```kotlin
class MyFragment : Fragment() {

    private var userScore = 0

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        // Восстановить данные при пересоздании
        userScore = savedInstanceState?.getInt("SCORE", 0) ?: userScore
    }

    override fun onSaveInstanceState(outState: Bundle) {
        super.onSaveInstanceState(outState)
        // Сохранить данные перед уничтожением
        outState.putInt("SCORE", userScore)
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        // Обновить UI
        textView.text = userScore.toString()
    }
}
```

---

### 🔎 Как это работает:
- При повороте экрана Fragment пересоздаётся.
- `onSaveInstanceState()` вызывается **до `onDestroyView()`**, сохраняя данные в `Bundle`.
- При новом создании `savedInstanceState` передаётся в `onCreate()` и `onViewCreated()`.

---

### ⚠️ Ограничения:
- Поддерживает только **простые типы** и `Parcelable`, `Serializable`.
- Не подходит для больших объектов (ограничение `Bundle` по памяти).
- Не работает, если процесс убит (в отличие от `ViewModel`).

---

### 💡 Альтернатива (без ViewModel):
- Использовать **сохранённое состояние в Activity**, а Fragment получает данные через `arguments` при пересоздании.

---

### Вывод:
> При отсутствии `ViewModel` — **`onSaveInstanceState()`** — основной способ сохранения **временных UI-данных** при повороте.  
> Но **`ViewModel` — предпочтительный и современный способ**.

</details>

<details> 
  <summary> Преимущества DialogFragment перед Dialog </summary>

**`DialogFragment`** — предпочтительный способ показа диалогов в Android. Вот его **ключевые преимущества** перед обычным `Dialog`:

---

### ✅ 1. **Корректное управление жизненным циклом**
- `DialogFragment` интегрирован в жизненный цикл `Fragment` и `Activity`.
- Автоматически обрабатывает пересоздание (например, при повороте экрана).
- Обычный `Dialog` при пересоздании **исчезает**, если не управлять им вручную.

---

### ✅ 2. **Работает с `FragmentManager`**
- Диалог становится частью фрагментного стека.
- Можно добавить в бэк-стек:
  ```kotlin
  .addToBackStack("dialog")
  ```
- Поддерживает навигацию "назад" корректно.

---

### ✅ 3. **Сохранение состояния**
- При пересоздании Activity (поворот) `DialogFragment` восстанавливает диалог автоматически.
- Обычный `Dialog` нужно сохранять и пересоздавать вручную (например, в `onSaveInstanceState`).

---

### ✅ 4. **Гибкость и переиспользование**
- Легко передавать аргументы через `setArguments()` (как у обычного фрагмента).
- Можно переиспользовать в разных Activity.
- Поддерживает `ViewModel`, `LiveData`, `ViewBinding`.

---

### ✅ 5. **Поддержка планшетов и сложных макетов**
- Можно использовать как полноэкранный диалог или попап.
- Легко адаптировать под `Dialog` или встроенный `Fragment` на планшетах (мастер-деталь).

---

### ✅ 6. **Безопасная работа с конфигурациями**
- Не вызывает утечек памяти при уничтожении Activity (если используется правильно).
- Система сама закрывает диалог при уничтожении хоста.

---

### ❌ Обычный `Dialog`:
- Нужно управлять вручную.
- Легко «утечь» (например, показать после `onDestroy()` Activity).
- Не сохраняется при повороте.

---

### Вывод:
> ✅ **`DialogFragment` — стандарт де-факто** для диалогов в современном Android.  
> Он надёжнее, гибче и лучше интегрирован в архитектуру приложения.

</details>

</details>

<details> 
  <summary> <h2> Senior </h2> </summary>

<details> 
  <summary> Как гарантированно доставить данные в Fragment ? (commit не дает такой гарантии)  </summary> 
  
  На **senior уровне** важно понимать: метод `commit()` у `FragmentTransaction` **не выполняется сразу**, а ставит транзакцию в очередь, и её выполнение **не гарантировано мгновенно**. Это может привести к ситуации, когда **Fragment создаётся до того, как данные были переданы**.

---

### ✅ Как **гарантированно** доставить данные во Fragment?

#### ✔️ 1. **Использовать `setArguments(Bundle)` до `commit()`**
Это **самый надёжный и рекомендуемый способ**.

```kotlin
val fragment = MyFragment()
val args = Bundle().apply {
    putString("key", "value")
}
fragment.arguments = args

supportFragmentManager
    .beginTransaction()
    .add(fragment, "tag")
    .commit()
```

> ✅ `arguments` передаются до создания фрагмента, сохраняются при пересоздании и доступны в `onCreate()` / `onCreateView()`.

> ⚠️ Никогда не передавайте данные через кастомные конструкторы — это **нарушает жизненный цикл**.

---

#### ✔️ 2. **Дождаться завершения транзакции — `commitNow()`**
Если нужно **гарантированно выполнить транзакцию синхронно** (например, в `onCreate()` Activity):

```kotlin
supportFragmentManager
    .beginTransaction()
    .add(fragment, "tag")
    .commitNow() // выполняется сразу, блокирует поток
```

> ✅ Гарантирует, что фрагмент добавлен и его `onCreate()` уже вызван.  
> ❌ Не использовать в UI-потоке для тяжёлых операций — может вызвать ANR.

> Подходит для: `onCreate()`, `onAttach()`, `DialogFragment`, `NavigationUI`.

---

#### ✔️ 3. **Использовать `commitAllowingStateLoss()` с осторожностью**
Только если **не критично** потеря состояния (например, в фоне), но **не рекомендуется** для передачи данных.

---

#### ✔️ 4. **Современный способ — Navigation Component + Safe Args**
```kotlin
val action = HomeFragmentDirections.toDetailFragment("value")
findNavController().navigate(action)
```

> ✅ Данные передаются через типизированные аргументы, безопасно и надёжно.  
> ✅ Гарантируется доставка через `arguments`.

---

#### ✔️ 5. **Для сложных сценариев — `ViewModel` + `SharedViewModel`**
Если данные генерируются после создания фрагмента:

```kotlin
// Общий ViewModel через Activity
val viewModel by activityViewModels<SharedViewModel>()
```

> ✅ Данные доставляются асинхронно, но надёжно через `LiveData`/`StateFlow`.

---

### ❌ Что **не работает**:
- Передача данных **после** `commit()` через сеттеры — фрагмент может ещё не быть создан.
- Хранение данных в локальных переменных — не переживают пересоздание.

---

### ✅ Итог (senior уровень):

| Способ | Когда использовать |
|-------|---------------------|
| `setArguments()` + `commit()` | По умолчанию, для простых данных |
| `commitNow()` | Когда нужно синхронное выполнение (например, в `onResume()`) |
| `Navigation + Safe Args` | В современных приложениях с навигацией |
| `Shared ViewModel` | Для динамических или общих данных между фрагментами |

> 🔑 **Гарантия доставки = `arguments` + `commitNow()` или `Navigation Component`**.  
> Это обеспечивает **предсказуемость**, **сохранность при пересоздании** и **интеграцию с жизненным циклом**.
</details>

</details>

