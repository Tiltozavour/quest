
<details> 
  <summary>  Junior </summary>

  <details>
  <summary> Каковы основные компоненты Android и для чего они нужны? </summary>

> Activity, Services, Broadcast Receiver, Content Provider

</details>

 
</details>







### **Расскажи что происходит при запуске приложения**

<details>
  <summary> Ответ </summary>

> 1. Запуск процесса
```
Когда пользователь запускает приложение, операционная система Android создает новый процесс (если он еще не существует) и выделяет для него ресурсы.
Каждое приложение в Android работает в изолированном процессе с собственной виртуальной машиной (ART/Dalvik).
```
> 2. Загрузка приложения
```
Система загружает код приложения из APK-файла.
Загружаются ресурсы приложения (изображения, строки, макеты и т.д.).
```
> 3. Создание объекта Application
```
Если в приложении есть пользовательский класс, унаследованный от Application, система создает его экземпляр.
В этом классе можно выполнить инициализацию глобальных переменных или библиотек (например, Firebase, аналитика и т.д.).
```
> 4. Запуск стартовой Activity
```
Система определяет, какая Activity должна быть запущена первой (указана в манифесте в теге <intent-filter> с действием MAIN и категорией LAUNCHER).
Создается экземпляр этой Activity.
```
> 5. Жизненный цикл Activity
```
OnCreate() - onStart() - OnResume()
```
> 6.  Отображение интерфейса\Работа приложения\Фоновые процессы
> 7.  Заверншение работы

</details>

### Как работает Intent и какие типы Intent существуют? 

<details>
  <summary> Ответ </summary>

> Intent работает как сообщение, которое передается системе Android, чтобы выполнить какое-либо действие. Система анализирует Intent и находит подходящий компонент (Activity, Service, BroadcastReceiver) для его обработки.

<details>
  <summary> Виды интентов </summary>

<details>
  <summary> Явный Intent (Explicit Intent) </summary>
 
- Указывает конкретный компонент (класс), который должен быть запущен.
- Используется для взаимодействия внутри приложения.

Пример:
```
val intent = Intent(this, SecondActivity::class.java)
startActivity(intent)

```

</details>

<details>
  <summary> Неявный Intent (Implicit Intent) </summary>

> Описывает действие, которое нужно выполнить, и система сама находит подходящий компонент (например, браузер, галерею, другое приложение).

**Может содержать:**

Действие (Action) – ACTION_VIEW, ACTION_SEND и т. д.\
Данные (Data) – URI (например, http://, tel:, content://).\
Тип данных (Type) – MIME-тип (text/plain, image/jpeg).\
Категория (Category) – CATEGORY_BROWSABLE, CATEGORY_LAUNCHER.\

Пример:
```
kotlin
val intent = Intent(Intent.ACTION_VIEW, Uri.parse("https://google.com"))
startActivity(intent)
```

</details>

<details>
  <summary> Broadcast Intent  </summary>

> Используется для отправки широковещательных сообщений через sendBroadcast().

Пример:
```
java
Intent intent = new Intent("com.example.CUSTOM_ACTION");
sendBroadcast(intent);
```

</details>

<details>
  <summary> PendingIntent  </summary>

> Это Intent, который может быть выполнен другим приложением от имени вашего приложения (например, уведомления или AlarmManager).

Пример:
```
java
PendingIntent pendingIntent = PendingIntent.getActivity(this, 0, intent, PendingIntent.FLAG_IMMUTABLE);
```

</details>

</details>

<details>
  <summary> Основные действия (Actions) в Intent  </summary>
 
- Intent.ACTION_VIEW – открыть данные (веб-страницу, карту, изображение).
- Intent.ACTION_SEND – отправить данные (текст, изображение).
- Intent.ACTION_DIAL – набрать номер в телефонном приложении)
- Intent.ACTION_MAIN – главная точка входа (используется в манифесте для LAUNCHER).

</details>

<details>
  <summary> Как система находит подходящий компонент?  </summary>

> Для неявных Intent система использует Intent Filter, объявленные в AndroidManifest.xml. Например:

```
xml
<activity android:name=".ShareActivity">
    <intent-filter>
        <action android:name="android.intent.action.SEND" />
        <category android:name="android.intent.category.DEFAULT" />
        <data android:mimeType="text/plain" />
    </intent-filter>
</activity>
```

</details>

</details>


### **Как реализовать глубокие ссылки (Deep Links) в Android?**

<details>
  <summary> Ответ </summary>
  
> Глубокие ссылки (Deep Links) позволяют открывать определенные экраны или контент в приложении из внешних источников (веб).\
> Подключаем manifest -  добавляем intent-фильтры\
> Обрабатываем в коде

</details>

 Что такое Android Manifest и для чего он нужен? 
• Объясните, что такое ViewModel и как его использовать. 
• Что такое LiveData и как его применять? 
• Что такое RecyclerView и как он отличается от ListView? 
• Как происходит взаимодействие между Activity и Fragment? 
• Как работает система разрешений в Android? 
• Как работает back stack в Android? 
• Чем отличаются Parcelable и Serializable? Какой способ предпочтительнее и 
почему? 
• Какие бывают способы межпроцессного взаимодействия (IPC) в Android? 
• Как реализовать глубокие ссылки (Deep Links) в Android? 

</details>

