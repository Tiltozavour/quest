# FragmentManager

<details>
  <summary> <h2> 🌱 Junior </h2> </summary>

<details>
  <summary> Вы заменяете один фрагмент другим - как обеспечить, чтобы пользователь мог вернуться к предыдущему фрагменту, нажав кнопку «Назад»? </summary>

  Чтобы пользователь мог вернуться к предыдущему фрагменту по кнопке «Назад», нужно добавить транзакцию во **внутренний стек операций** (back stack):

```kotlin
supportFragmentManager.beginTransaction()
    .replace(R.id.container, newFragment)
    .addToBackStack(null) // Добавляет транзакцию в back stack
    .commit()
```

После `addToBackStack()` предыдущий фрагмент сохраняется в стеке, и при нажатии кнопки "Назад" произойдёт возврат к нему.
  
</details>
  
</details>

<details>
  <summary> <h2> 🌿 Middle </h2> </summary>

<details>
  <summary> Есть три фрагмента A B и C. Изначально в контейнере только фрагмент A, после чего происходит add B, а затем replace на С. Какие методы ЖЦ будут вызываться у фрагментов? </summary>
  
Разберём по шагам:

1. **Изначально: добавлен фрагмент A**  
   → `A.onCreate()`, `A.onCreateView()`, `A.onStart()`, `A.onResume()`

2. **Добавляем B через `add(B)`**  
   → `B.onCreate()`, `B.onCreateView()`, `B.onStart()`, `B.onResume()`  
   (A остаётся в фоне, его ЖЦ не меняется)

3. **Заменяем на C через `replace(C)`**  
   `replace()` удаляет **все фрагменты в контейнере** и добавляет C:  
   → `A.onPause()`, `A.onStop()`, `A.onDestroyView()`  
   → `B.onPause()`, `B.onStop()`, `B.onDestroyView()`  
   → `C.onCreate()`, `C.onCreateView()`, `C.onStart()`, `C.onResume()`

**Итог:** A и B теряют свои view, но остаются вFragmentManager (если были добавлены с `addToBackStack`). C — в состоянии resumed.
  
</details>
  
</details>

-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
**Ранее**

- [FragmentManager](FRAGMENT_MANAGER.md)
- [Fragment](FRAGMENT.md)

**Далее**
- [](.md)


**Оглавление**
- [Readme](README.md)
