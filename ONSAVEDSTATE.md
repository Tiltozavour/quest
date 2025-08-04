
# onSaveInstanceState/ onRestoreInstanceState

<details>
  <summary> <h2> 🌱 Junior </h2> </summary>

<details>
  <summary> Что за методы, для чего нужны? </summary>

**`onSaveInstanceState()`** — вызывается перед уничтожением Activity/Fragment, чтобы сохранить **временные данные** (например, состояние UI).

**`onRestoreInstanceState()`** — вызывается при пересоздании, чтобы восстановить эти данные.

Используются для сохранения:
- Текст в EditText
- Позиция в списке
- Выделенные вкладки и т.п.

Пример (Activity):
```kotlin
override fun onSaveInstanceState(outState: Bundle) {
    outState.putString("key", "value")
    super.onSaveInstanceState(outState)
}

override fun onRestoreInstanceState(savedInstanceState: Bundle) {
    super.onRestoreInstanceState(savedInstanceState)
    val value = savedInstanceState.getString("key")
}
```

> 💡 Вызывается только при **конфигурационных изменениях** (поворот, смена языка), но не при завершении приложения.

</details>

<details> 
  <summary> На каком этапе жизненного цикла активити вызывается onSaveInstanceState? </summary>

`onSaveInstanceState()` вызывается **после `onPause()` и перед `onStop()`**.

Порядок:
```
onPause() → onSaveInstanceState() → onStop()
```

> 🔹 Вызывается только если Activity может быть уничтожена (например, при повороте экрана).  
> 🔹 Не вызывается, если пользователь закрывает Activity кнопкой "Назад" или через `finish()`.
  
</details>

<details> 
  <summary> на каком этапе жизненного цикла фрагмента вызывается onRestoreInstanceState?</summary>

`onRestoreInstanceState()` у фрагмента вызывается в **`onActivityCreated()`**, но **после** восстановления состояния Activity.

Более точно:
- Вызывается **после `onCreate()` и `onCreateView()`**, но **до `onStart()`**.
- Обычно — внутри `onActivityCreated()`, когда состояние уже доступно.

Порядок у фрагмента:
```
onCreate() → onCreateView() → onViewCreated() → onActivityCreated() → [onRestoreInstanceState()] → onStart()
```

> 💡 Используйте `onViewStateRestored()` — более явное место для реакции на восстановление UI-состояния.

</details>
  

<details> 
  <summary> В каких случаях onRestoreInstanceState не вызывается? </summary>

`onRestoreInstanceState()` **не вызывается**, если:

1. **Состояние не было сохранено** — например, `onSaveInstanceState()` не вызывался (при `finish()` или завершении приложения).
2. **Activity/фрагмент уничтожается штатно** — пользователь нажал "Назад" или вызван `finish()`.
3. **Приложение убито системой** и запущено заново через launcher (а не из recents) — тогда нет `savedInstanceState`.
4. **Первый запуск Activity/фрагмента** — нет данных для восстановления.

> 🔹 `savedInstanceState` в `onCreate()` и `onRestoreInstanceState()` будет `null` в этих случаях.  
> 🔹 Всегда проверяй: `if (savedInstanceState != null)` перед восстановлением.
  
</details>

</details>

<details>
  <summary> <h2> 🌿 Middle </h2> </summary>

<details>
  <summary> Как можно сохранять сложные объекты?(не базовые типы) </summary>
  
### 1. **Объект должен реализовывать `Parcelable`** (предпочтительно)
```kotlin
class User(val name: String, val age: Int) : Parcelable {
    constructor(parcel: Parcel) : this(
        parcel.readString()!!,
        parcel.readInt()
    )

    override fun writeToParcel(parcel: Parcel, flags: Int) {
        parcel.writeString(name)
        parcel.writeInt(age)
    }

    override fun describeContents() = 0

    companion object : Parcelable.Creator<User> {
        override fun createFromParcel(parcel: Parcel) = User(parcel)
        override fun newArray(size: Int) = arrayOfNulls<User>(size)
    }
}
```

Использование:
```kotlin
override fun onSaveInstanceState(outState: Bundle) {
    outState.putParcelable("user", user)
    super.onSaveInstanceState(outState)
}

override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    val user = savedInstanceState?.getParcelable<User>("user")
}
```

---

### ⚠️ Не используй:
- `Serializable` — медленнее, грузит main thread.
- `Bitmap`, `List<Any>` и другие не-Parcelable объекты напрямую.

---

### Альтернативы (лучше):
- **ViewModel + SavedStateHandle** — современный способ (Jetpack):
```kotlin
class MyViewModel(savedState: SavedStateHandle) : ViewModel() {
    var user by savedState.getLiveData<User>("user")
}
```
> ✅ Сохраняет состояние между пересозданиями, работает с процессами, проще и безопаснее.

---

**Вывод:**  
Для `onSaveInstanceState` — только `Parcelable`.  
Но лучше использовать **SavedStateHandle + ViewModel** для сложных данных.

</details>
  
</details>

-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
**Ранее**

- [Fragment](FRAGMENT.md)
- [FragmentManager](FRAGMENT_MANAGER.md)

**Далее**



**Оглавление**
- [Readme](README.md)
