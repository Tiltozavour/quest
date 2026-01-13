
# Broadcast Receive

<details>
  <summary> <h2> 🌱 Junior </h2> </summary>

<details>
  <summary> Для чего используется Broadcast Receiver? </summary>
  
**Broadcast Receiver (приемник широковещательных сообщений)** — это компонент Android, который позволяет приложению реагировать на системные или кастомные события (broadcasts), происходящие в системе.

---

## **Для чего используется:**

### 1. **Реакция на системные события:**
- 📱 Изменение состояния телефона (включение/выключение, перезагрузка)
- 🔋 Изменение уровня заряда батареи
- ✈️ Включение/выключение авиарежима
- 📶 Изменение состояния сети (подключение/отключение Wi-Fi, мобильной сети)
- 📷 Сделана фотография
- 📥 Установка/удаление приложений
- 🔔 Изменение режима "Не беспокоить"

### 2. **Получение сообщений от других приложений:**
- 📩 Получение SMS/MMS
- 📞 Входящие/исходящие вызовы
- 📢 Push-уведомления (через FCM)
- 🔄 Синхронизация данных

### 3. **Внутриприложенная коммуникация:**
- 📡 Отправка и получение кастомных событий
- 🔀 Обмен данными между компонентами приложения
- 🎯 Уведомления о завершении фоновых задач

---

## **Примеры использования:**

```java
// Получение уведомления о подключении к сети
public class NetworkReceiver extends BroadcastReceiver {
    @Override
    public void onReceive(Context context, Intent intent) {
        ConnectivityManager cm = (ConnectivityManager) 
            context.getSystemService(Context.CONNECTIVITY_SERVICE);
        
        if (cm.getActiveNetworkInfo() != null) {
            // Сеть доступна - начать синхронизацию
            startSync();
        } else {
            // Нет сети - показать уведомление
            showNoInternetNotification();
        }
    }
}
```

---

## **Ключевые особенности:**

### **Типы Broadcast Receiver:**

1. **Статический (Manifest-declared)**
   ```xml
   <receiver android:name=".MyReceiver"
             android:exported="true">
       <intent-filter>
           <action android:name="android.intent.action.BOOT_COMPLETED"/>
       </intent-filter>
   </receiver>
   ```
   - Регистрируется в манифесте
   - Работает даже когда приложение не запущено
   - **Ограничения:** Начиная с Android 8.0 не получает неявные интенты

2. **Динамический (Context-registered)**
   ```kotlin
   val receiver = object : BroadcastReceiver() {
       override fun onReceive(context: Context, intent: Intent) {
           // Обработка события
       }
   }
   
   val filter = IntentFilter().apply {
       addAction(Intent.ACTION_AIRPLANE_MODE_CHANGED)
   }
   
   // Регистрация
   context.registerReceiver(receiver, filter)
   
   // Обязательно отменить регистрацию!
   context.unregisterReceiver(receiver)
   ```
   - Регистрируется программно
   - Работает только когда приложение активно
   - Гибче в использовании

---

## **Практическое применение:**

```kotlin
// Пример: Контроль состояния зарядки
class PowerReceiver : BroadcastReceiver() {
    override fun onReceive(context: Context, intent: Intent) {
        when (intent.action) {
            Intent.ACTION_POWER_CONNECTED -> {
                // Зарядка подключена
                Toast.makeText(context, "Charging started", Toast.LENGTH_SHORT).show()
                // Начать тяжелые операции
                startHeavyTasks()
            }
            Intent.ACTION_POWER_DISCONNECTED -> {
                // Зарядка отключена
                Toast.makeText(context, "Charging stopped", Toast.LENGTH_SHORT).show()
                // Приостановить тяжелые операции
                pauseHeavyTasks()
            }
            Intent.ACTION_BATTERY_LOW -> {
                // Батарея разряжена
                saveCriticalData()
                disableBackgroundServices()
            }
        }
    }
}
```

---

## **Важные ограничения (Android 8.0+):**

1. **Неявные broadcast'ы** (implicit broadcasts) нельзя получать через манифест
2. **Исключения:** Некоторые системные события (загрузка системы, изменение локали) по-прежнему работают
3. **Решение:** Использовать динамическую регистрацию или JobScheduler/WorkManager

---

## **Когда использовать Broadcast Receiver:**

✅ **Для реакции на системные события** (сеть, зарядка, загрузка)  
✅ **Для внутриприложенной коммуникации** между компонентами  
✅ **Для получения явных broadcast'ов** от других приложений  
❌ **Не использовать** для длительных операций (максимум 10 секунд)  
❌ **Не использовать** для частых событий (лучше использовать LiveData, EventBus, RxJava)

---

## **Краткий итог:**

Broadcast Receiver — это **"слушатель системных и прикладных событий"**, который позволяет вашему приложению реагировать на изменения в системе и получать сообщения от других компонентов без постоянного опроса или активной работы.

</details>

<details>
  <summary> Когда используется Local Broadcast? </summary>
  
**Local Broadcast (локальный broadcast)** — это механизм для отправки и получения broadcast'ов **только внутри вашего приложения**.

---

## **Когда используется Local Broadcast:**

### **1. Внутриприложенная коммуникация между компонентами:**
```kotlin
// Activity отправляет событие
val intent = Intent("com.example.MY_ACTION")
intent.putExtra("data", "Hello from Activity")
LocalBroadcastManager.getInstance(this).sendBroadcast(intent)

// Fragment принимает событие
val receiver = object : BroadcastReceiver() {
    override fun onReceive(context: Context, intent: Intent) {
        val data = intent.getStringExtra("data")
        // Обновляем UI во Fragment
        updateUI(data)
    }
}

LocalBroadcastManager.getInstance(requireContext())
    .registerReceiver(receiver, IntentFilter("com.example.MY_ACTION"))
```

### **2. Обмен данными между независимыми компонентами:**
- **Activity ↔ Fragment**
- **Service ↔ Activity**  
- **Вложенные Fragment'ы** друг с другом
- **Виджеты ↔ Основное приложение**

### **3. Уведомление о завершении фоновых задач:**
```kotlin
class DownloadService : Service() {
    fun downloadComplete(filePath: String) {
        val intent = Intent("DOWNLOAD_COMPLETE").apply {
            putExtra("file_path", filePath)
        }
        LocalBroadcastManager.getInstance(this).sendBroadcast(intent)
    }
}

// Activity показывает результат
class MainActivity : AppCompatActivity() {
    private val receiver = object : BroadcastReceiver() {
        override fun onReceive(context: Context, intent: Intent) {
            val filePath = intent.getStringExtra("file_path")
            showDownloadComplete(filePath)
        }
    }
    
    override fun onCreate(savedInstanceState: Bundle?) {
        LocalBroadcastManager.getInstance(this)
            .registerReceiver(receiver, IntentFilter("DOWNLOAD_COMPLETE"))
    }
}
```

---

## **Преимущества Local Broadcast перед глобальным:**

| Параметр | Global Broadcast | Local Broadcast |
|----------|------------------|-----------------|
| **Безопасность** | Доступен другим приложениям | Только внутри вашего приложения |
| **Производительность** | Проходит через системный процесс | Прямая отправка в приложении |
| **Конфиденциальность** | Данные могут быть перехвачены | Данные защищены |
| **Системная нагрузка** | Нагружает систему | Минимальная нагрузка |

---

## **Типичные сценарии использования:**

### **Сценарий 1: Обновление UI из Service**
```kotlin
// Сервис выполняет задачу и сообщает о прогрессе
class ProgressService : Service() {
    fun updateProgress(progress: Int) {
        val intent = Intent("PROGRESS_UPDATE").apply {
            putExtra("progress", progress)
        }
        LocalBroadcastManager.getInstance(this).sendBroadcast(intent)
    }
}

// Activity показывает прогресс
class MainActivity : AppCompatActivity() {
    private val progressReceiver = object : BroadcastReceiver() {
        override fun onReceive(context: Context, intent: Intent) {
            val progress = intent.getIntExtra("progress", 0)
            progressBar.progress = progress
        }
    }
}
```

### **Сценарий 2: Синхронизация между Fragment'ами**
```kotlin
// Fragment A отправляет событие
class FragmentA : Fragment() {
    fun onItemSelected(itemId: String) {
        val intent = Intent("ITEM_SELECTED").apply {
            putExtra("item_id", itemId)
        }
        LocalBroadcastManager.getInstance(requireContext())
            .sendBroadcast(intent)
    }
}

// Fragment B реагирует на выбор
class FragmentB : Fragment() {
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        LocalBroadcastManager.getInstance(requireContext())
            .registerReceiver(receiver, 
                IntentFilter("ITEM_SELECTED"))
    }
    
    private val receiver = object : BroadcastReceiver() {
        override fun onReceive(context: Context, intent: Intent) {
            val itemId = intent.getStringExtra("item_id")
            loadDetails(itemId)
        }
    }
}
```

### **Сценарий 3: Авторизация/выход из системы**
```kotlin
// После успешного входа
fun onLoginSuccess(user: User) {
    val intent = Intent("USER_LOGIN").apply {
        putExtra("user_name", user.name)
        putExtra("user_email", user.email)
    }
    LocalBroadcastManager.getInstance(this).sendBroadcast(intent)
}

// Все Activity/Fragment'ы обновляются
class ProfileFragment : Fragment() {
    private val authReceiver = object : BroadcastReceiver() {
        override fun onReceive(context: Context, intent: Intent) {
            val userName = intent.getStringExtra("user_name")
            userNameTextView.text = userName
        }
    }
}
```

---

## **Как использовать:**

### **1. Отправка Local Broadcast:**
```kotlin
// Создаем intent с действием
val intent = Intent("CUSTOM_ACTION_NAME")
// Добавляем данные (опционально)
intent.putExtra("key", "value")
// Отправляем
LocalBroadcastManager.getInstance(context).sendBroadcast(intent)
```

### **2. Регистрация и получение:**
```kotlin
// Создаем receiver
val myReceiver = object : BroadcastReceiver() {
    override fun onReceive(context: Context, intent: Intent) {
        // Обрабатываем событие
        when (intent.action) {
            "CUSTOM_ACTION_NAME" -> {
                val data = intent.getStringExtra("key")
                handleData(data)
            }
        }
    }
}

// Создаем фильтр
val filter = IntentFilter().apply {
    addAction("CUSTOM_ACTION_NAME")
    addAction("ANOTHER_ACTION") // Можно слушать несколько действий
}

// Регистрируем
LocalBroadcastManager.getInstance(context)
    .registerReceiver(myReceiver, filter)
```

### **3. Обязательная отмена регистрации:**
```kotlin
override fun onDestroy() {
    super.onDestroy()
    // Важно: всегда отменяем регистрацию!
    LocalBroadcastManager.getInstance(this)
        .unregisterReceiver(myReceiver)
}
```

---

## **Важные нюансы:**

1. **LocalBroadcastManager устарел** в AndroidX, но все еще работает
2. **Современная альтернатива:** Используйте:
   - `LiveData` + `ViewModel` для UI компонентов
   - `Kotlin Flow` или `RxJava` для реактивного программирования
   - `EventBus` библиотеки (GreenRobot EventBus)
   - **Однако Local Broadcast все еще полезен для простых сценариев**

3. **Преимущества над альтернативами:**
   - Не требует дополнительных зависимостей
   - Простая интеграция в существующий код
   - Знакомая парадигма (как обычный BroadcastReceiver)

---

## **Практический пример: Обновление корзины покупок**

```kotlin
// Когда товар добавлен в корзину (в ProductActivity)
fun addToCart(product: Product) {
    CartManager.addProduct(product)
    
    // Уведомляем другие компоненты
    val intent = Intent("CART_UPDATED").apply {
        putExtra("cart_count", CartManager.getCount())
        putExtra("product_name", product.name)
    }
    LocalBroadcastManager.getInstance(this).sendBroadcast(intent)
}

// CartFragment обновляет счетчик
class CartFragment : Fragment() {
    private val cartReceiver = object : BroadcastReceiver() {
        override fun onReceive(context: Context, intent: Intent) {
            val count = intent.getIntExtra("cart_count", 0)
            val productName = intent.getStringExtra("product_name")
            
            badgeView.text = count.toString()
            showToast("$productName добавлен в корзину")
        }
    }
    
    override fun onStart() {
        super.onStart()
        LocalBroadcastManager.getInstance(requireContext())
            .registerReceiver(cartReceiver, 
                IntentFilter("CART_UPDATED"))
    }
    
    override fun onStop() {
        LocalBroadcastManager.getInstance(requireContext())
            .unregisterReceiver(cartReceiver)
        super.onStop()
    }
}
```

---

## **Когда НЕ использовать Local Broadcast:**

❌ **Для частых событий** (чаще чем раз в секунду)  
❌ **Для передачи больших данных** (больше 1MB)  
❌ **Вместо прямого вызова методов** между связанными компонентами  
❌ **Для сложных сценариев** с множеством подписчиков и событий  
❌ **В новых проектах** без легаси кода (лучше использовать современные альтернативы)

---

## **Краткий итог:**

**Local Broadcast используется когда нужно:**
1. **Безопасно** обмениваться событиями внутри приложения
2. **Не загружать систему** глобальными broadcast'ами
3. **Связать независимые компоненты** без прямых ссылок
4. **Быстро реализовать** простую коммуникацию в legacy коде

**Идеально для:** уведомлений о изменениях состояния, обновления UI из фона, синхронизации между экранами одного приложения.

</details>

<details>
  <summary> В чем разница между sendStickyBroadcast и sendBroadcast?  </summary>

**Основная разница:** `sendStickyBroadcast` сохраняет intent в системе, и последний зарегистрированный receiver может получить его даже после отправки.

---

## **Сравнение:**

| Параметр | `sendBroadcast()` | `sendStickyBroadcast()` |
|----------|-------------------|--------------------------|
| **Доставка** | Только текущим receiver'ам | + Сохраняется в системе |
| **Получение** | Только в момент отправки | Можно получить позже |
| **Порядок** | Очередь событий | Последнее значение перезаписывает предыдущее |
| **Безопасность** | Обычная | Требует разрешения `BROADCAST_STICKY` |
| **Статус** | Актуальный | **Устарел (deprecated) с API 21** |

---

## **sendBroadcast() — обычная отправка:**

```kotlin
// Отправка
val intent = Intent("MY_ACTION")
intent.putExtra("data", "Hello")
context.sendBroadcast(intent)

// Receiver получает ТОЛЬКО если зарегистрирован в момент отправки
// Если receiver зарегистрируется позже - не получит это сообщение
```

---

## **sendStickyBroadcast() — "липкая" отправка:**

```kotlin
// 1. Отправка (intent сохраняется в системе)
val intent = Intent("MY_ACTION")
intent.putExtra("data", "Hello")
context.sendStickyBroadcast(intent) // Устарел!

// 2. Receiver может получить это сообщение ПОСЛЕ регистрации
val receiver = object : BroadcastReceiver() {
    override fun onReceive(context: Context, intent: Intent) {
        // Получит "Hello" даже если зарегистрировался через минуту
        val data = intent.getStringExtra("data")
    }
}

// 3. Можно получить последний sticky intent
val lastIntent = context.registerReceiver(
    receiver, 
    IntentFilter("MY_ACTION")
)
// lastIntent будет содержать последний отправленный sticky intent
```

---

## **Как работал Sticky Broadcast (до API 21):**

### **Пример: Мониторинг состояния батареи**
```java
// Система отправляет sticky broadcast при изменении заряда
sendStickyBroadcast(new Intent(Intent.ACTION_BATTERY_CHANGED));

// Приложение может получить последнее состояние в любой момент
IntentFilter filter = new IntentFilter(Intent.ACTION_BATTERY_CHANGED);
Intent batteryStatus = context.registerReceiver(null, filter);

// Извлекаем данные
int level = batteryStatus.getIntExtra(BatteryManager.EXTRA_LEVEL, -1);
int scale = batteryStatus.getIntExtra(BatteryManager.EXTRA_SCALE, -1);
float batteryPercent = level * 100 / (float)scale;
```

### **Сценарий использования:**
1. **Отправка:** `sendStickyBroadcast(intent)`
2. **Intent сохраняется** в системной памяти
3. **Новый receiver** регистрируется с `IntentFilter`
4. **Сразу получает** последний сохраненный sticky intent
5. **Можно получить** через `registerReceiver(null, filter)`

---

## **Проблемы Sticky Broadcast:**

1. **Утечки памяти** — intents постоянно хранятся в системе
2. **Безопасность** — любой receiver может получить sticky intent
3. **Нет контроля** — нельзя удалить sticky intent явно (только заменой)
4. **Производительность** — хранение всех sticky intents в памяти

---

## **Альтернативы для замены Sticky Broadcast:**

### **1. Local Storage (SharedPreferences, Room)**
```kotlin
// Вместо sticky broadcast
object AppState {
    private const val PREFS_NAME = "app_state"
    
    fun saveLastLocation(location: String) {
        getSharedPrefs().edit()
            .putString("last_location", location)
            .apply()
    }
    
    fun getLastLocation(): String {
        return getSharedPrefs()
            .getString("last_location", "")
    }
}

// Использование
AppState.saveLastLocation("Москва")
val location = AppState.getLastLocation() // Москва
```

### **2. LiveData/StateFlow (реактивное программирование)**
```kotlin
// ViewModel хранит состояние
class AppViewModel : ViewModel() {
    private val _lastEvent = MutableLiveData<String>()
    val lastEvent: LiveData<String> = _lastEvent
    
    fun updateLastEvent(event: String) {
        _lastEvent.value = event
    }
}

// Activity/Fragment наблюдает
viewModel.lastEvent.observe(this) { event ->
    // Получает последнее значение даже если было создано позже
    showEvent(event)
}
```

### **3. Системные sticky events через официальные API**
```kotlin
// Вместо sticky broadcast для батареи
val batteryStatus = context.registerReceiver(
    null,
    IntentFilter(Intent.ACTION_BATTERY_CHANGED)
)

// Или через BatteryManager
val bm = context.getSystemService(BATTERY_SERVICE) as BatteryManager
val batteryPercent = bm.getIntProperty(BatteryManager.BATTERY_PROPERTY_CAPACITY)
```

---

## **Практический пример перехода с Sticky:**

### **Было (устаревший код):**
```java
// Service отправляет sticky broadcast
public void onLocationChanged(Location location) {
    Intent intent = new Intent("LOCATION_UPDATE");
    intent.putExtra("lat", location.getLatitude());
    intent.putExtra("lng", location.getLongitude());
    sendStickyBroadcast(intent); // Deprecated!
}

// Activity получает последнюю локацию
IntentFilter filter = new IntentFilter("LOCATION_UPDATE");
Intent lastLocation = registerReceiver(null, filter); // Получаем sticky
```

### **Стало (современный подход):**
```kotlin
// 1. Храним в Repository
class LocationRepository {
    private val _lastLocation = MutableLiveData<Location>()
    val lastLocation: LiveData<Location> = _lastLocation
    
    fun updateLocation(location: Location) {
        _lastLocation.value = location
    }
}

// 2. Service обновляет Repository
class LocationService : Service() {
    private val repository = LocationRepository.getInstance()
    
    fun onLocationChanged(location: Location) {
        repository.updateLocation(location)
    }
}

// 3. Activity наблюдает за LiveData
class MapActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        LocationRepository.getInstance().lastLocation
            .observe(this) { location ->
                // Получаем последнюю локацию
                updateMap(location)
            }
    }
}
```

---

## **Ключевые отличия в деталях:**

### **Метод получения:**
```java
// sendBroadcast() - обычный
context.sendBroadcast(intent);

// sendStickyBroadcast() - устарел
context.sendStickyBroadcast(intent); // Требует BROADCAST_STICKY permission

// sendOrderedBroadcast() - с порядком доставки
context.sendOrderedBroadcast(intent, null);
```

### **Получение последнего sticky:**
```java
// Sticky можно получить через registerReceiver с null receiver
Intent lastSticky = context.registerReceiver(
    null, // null receiver!
    new IntentFilter("MY_STICKY_ACTION")
);
// lastSticky будет содержать последний sticky intent

// Обычный broadcast так получить нельзя
Intent lastNormal = context.registerReceiver(
    null,
    new IntentFilter("MY_NORMAL_ACTION")
); // Вернет null, если нет текущего broadcast'а
```

### **Удаление sticky:**
```java
// Sticky можно удалить только заменой или
context.removeStickyBroadcast(intent);

// Обычный broadcast удаляется автоматически после доставки
```

---

## **Итог для Junior:**

1. **`sendBroadcast()`** — обычная отправка, получают только активные receiver'ы
2. **`sendStickyBroadcast()`** — **УСТАРЕЛ**, сохранял intent для поздних receiver'ов
3. **Не используйте sticky** в новых проектах (deprecated с Android 5.0)
4. **Используйте альтернативы:** SharedPreferences, LiveData, Room, Flow
5. **Sticky был полезен** для системных состояний (заряд батареи), но теперь есть официальные API

**Правило:** Если видите `sendStickyBroadcast()` в коде — это легаси, требующее рефакторинга!

</details>

<details>
  <summary> Расскажи жизненный цикл Broadcast Receiver'а   </summary>

**Жизненный цикл Broadcast Receiver'а очень короткий и простой.**

---

## **Ключевой метод жизненного цикла:**

### **`onReceive(Context context, Intent intent)`**
- **Единственный** метод, который определяет жизненный цикл
- Вызывается при получении broadcast'а
- **Работает максимум 10 секунд**, после чего система может убить процесс
- Выполняется в **главном (UI) потоке**

---

## **Схема жизненного цикла:**

```
         Регистрация
              │
              ▼
      [Receiver создан]
              │
              ▼
    Ожидание broadcast'а
              │
              ▼
    Пришел broadcast intent
              │
              ▼
    ┌─────────────────────┐
    │  ВЫЗЫВАЕТСЯ         │
    │  onReceive()        │ ←─ Короткое выполнение (макс 10 сек)
    └─────────────────────┘
              │
              ▼
    [Receiver активен]    ←─ Можно запустить Service для длительной работы
              │
              ▼
      Завершение работы
              │
              ▼
      [Receiver уничтожен]
```

---

## **Типы Receiver'ов и их особенности:**

### **1. Динамический Receiver (Context-registered)**

```kotlin
class MainActivity : AppCompatActivity() {
    private lateinit var receiver: BroadcastReceiver
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate()
        
        // 1. СОЗДАНИЕ Receiver'а
        receiver = object : BroadcastReceiver() {
            override fun onReceive(context: Context, intent: Intent) {
                // 3. АКТИВНАЯ ФАЗА
                Log.d("Receiver", "Получен broadcast")
                
                // Важно: не делайте длительные операции здесь!
                // Максимум 10 секунд на выполнение
            }
        }
        
        // 2. РЕГИСТРАЦИЯ
        val filter = IntentFilter().apply {
            addAction(Intent.ACTION_AIRPLANE_MODE_CHANGED)
        }
        registerReceiver(receiver, filter)
    }
    
    override fun onDestroy() {
        super.onDestroy()
        // 4. УНИЧТОЖЕНИЕ (обязательно!)
        unregisterReceiver(receiver)
    }
}
```

**Жизненный цикл динамического Receiver'а:**
```
onCreate() → registerReceiver() → [ожидание] → onReceive() → [работа] → onDestroy() → unregisterReceiver()
```

---

### **2. Статический Receiver (Manifest-declared)**

```xml
<!-- AndroidManifest.xml -->
<receiver 
    android:name=".MyReceiver"
    android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.BOOT_COMPLETED"/>
    </intent-filter>
</receiver>
```

```kotlin
class MyReceiver : BroadcastReceiver() {
    override fun onReceive(context: Context, intent: Intent) {
        // Receiver создается системой при получении broadcast'а
        
        // Короткая работа (макс 10 секунд)
        Log.d("Receiver", "Устройство загружено!")
        
        // После завершения onReceive() Receiver уничтожается
    }
}
```

**Жизненный цикл статического Receiver'а:**
```
Система получает broadcast → Создает экземпляр Receiver'а → onReceive() → Уничтожает Receiver
```

---

## **Важные особенности жизненного цикла:**

### **1. Короткое время жизни**
```kotlin
override fun onReceive(context: Context, intent: Intent) {
    // ❌ НЕПРАВИЛЬНО - долгая операция
    Thread.sleep(15000) // ANR после 10 секунд!
    
    // ✅ ПРАВИЛЬНО - запуск Service для длительной работы
    val serviceIntent = Intent(context, MyService::class.java)
    context.startService(serviceIntent)
    
    // ✅ Или использование goAsync() (Android 3.0+)
    val pendingResult = goAsync()
    Thread {
        // Долгая операция в фоне
        Thread.sleep(20000)
        pendingResult.finish()
    }.start()
}
```

### **2. Автоматическое уничтожение**
- После `onReceive()` система может убить процесс
- Не храните состояние в полях Receiver'а
- Используйте `goAsync()` для асинхронной работы

### **3. Ограничения начиная с Android 8.0 (API 26)**
```kotlin
// Статические receiver'ы НЕ получают неявные broadcast'ы
// Исключения:
// - BOOT_COMPLETED
// - LOCALE_CHANGED  
// - другие явные исключения

// Решение для Android 8.0+:
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        // Динамическая регистрация
        val receiver = MyReceiver()
        val filter = IntentFilter("MY_CUSTOM_ACTION")
        
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            // Context-registered receivers still work
            registerReceiver(receiver, filter)
        }
    }
}
```

---

## **Паттерны работы с жизненным циклом:**

### **Паттерн 1: Запуск Service для длительной работы**
```kotlin
class MyReceiver : BroadcastReceiver() {
    override fun onReceive(context: Context, intent: Intent) {
        // 1. Быстрая проверка
        if (!isValidIntent(intent)) return
        
        // 2. Запуск Service (работает дольше 10 секунд)
        val serviceIntent = Intent(context, DownloadService::class.java)
        serviceIntent.putExtra("url", intent.getStringExtra("url"))
        
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            context.startForegroundService(serviceIntent)
        } else {
            context.startService(serviceIntent)
        }
        
        // 3. onReceive завершается быстро
    }
}
```

### **Паттерн 2: Использование goAsync()**
```kotlin
class AsyncReceiver : BroadcastReceiver() {
    override fun onReceive(context: Context, intent: Intent) {
        // Получаем PendingResult для асинхронной работы
        val pendingResult = goAsync()
        
        CoroutineScope(Dispatchers.IO).launch {
            try {
                // Длительная операция
                processData(intent)
                
                // Сохраняем результат
                saveToDatabase()
                
                // Завершаем работу receiver'а
                pendingResult.finish()
            } catch (e: Exception) {
                pendingResult.finish()
            }
        }
        
        // onReceive завершается сразу, но receiver жив до pendingResult.finish()
    }
}
```

### **Паттерн 3: Отложенная работа через Handler**
```kotlin
class DelayedReceiver : BroadcastReceiver() {
    override fun onReceive(context: Context, intent: Intent) {
        // Используем Handler для отложенной работы в UI потоке
        Handler(context.mainLooper).postDelayed({
            // Эта работа выполнится через 5 секунд
            showNotification(context, "Task completed")
        }, 5000)
        
        // Важно: receiver будет жить пока сообщения в Handler'е не обработаны
    }
}
```

---

## **Ошибки в жизненном цикле:**

### **❌ ОШИБКА: Долгая операция в onReceive()**
```kotlin
override fun onReceive(context: Context, intent: Intent) {
    // Будет ANR (Application Not Responding)
    downloadLargeFile() // 30 секунд
    processData()       // Еще 20 секунд
}
```

### **✅ РЕШЕНИЕ: Правильный паттерн**
```kotlin
override fun onReceive(context: Context, intent: Intent) {
    when (intent.action) {
        Intent.ACTION_POWER_CONNECTED -> {
            // Быстрая операция
            showToast("Charging started")
            
            // Долгая операция в Service
            startSyncService(context)
        }
    }
}
```

---

## **Сравнение с жизненным циклом Activity:**

| Параметр | Broadcast Receiver | Activity |
|----------|-------------------|----------|
| **Основной метод** | `onReceive()` | `onCreate()`, `onStart()`, `onResume()` и т.д. |
| **Время жизни** | Мгновенное (до 10 сек) | Длительное (пока пользователь взаимодействует) |
| **Уничтожение** | Автоматически после `onReceive()` | По команде системы или пользователя |
| **Сохранение состояния** | Не поддерживается | `onSaveInstanceState()` |
| **Восстановление** | Невозможно | `onRestoreInstanceState()` |

---

## **Практический пример: Мониторинг сети**

```kotlin
class NetworkReceiver : BroadcastReceiver() {
    // Жизненный цикл:
    // 1. Создание при регистрации
    // 2. Ожидание события изменения сети
    // 3. onReceive() при изменении сети
    // 4. Уничтожение после onReceive()
    
    override fun onReceive(context: Context, intent: Intent) {
        // ФАЗА АКТИВНОЙ РАБОТЫ (макс 10 секунд)
        
        val connectivityManager = context.getSystemService(
            Context.CONNECTIVITY_SERVICE
        ) as ConnectivityManager
        
        val networkInfo = connectivityManager.activeNetworkInfo
        
        if (networkInfo != null && networkInfo.isConnected) {
            // Сеть есть - запускаем синхронизацию
            val syncIntent = Intent(context, SyncService::class.java)
            context.startService(syncIntent)
        } else {
            // Нет сети - показываем уведомление
            showNoInternetNotification(context)
        }
        
        // Receiver будет уничтожен системой после выхода из onReceive()
    }
    
    private fun showNoInternetNotification(context: Context) {
        // Быстрая операция - создание уведомления
        val notification = NotificationCompat.Builder(context, "network_channel")
            .setContentTitle("Нет интернета")
            .setSmallIcon(android.R.drawable.ic_dialog_alert)
            .build()
        
        NotificationManagerCompat.from(context)
            .notify(1, notification)
    }
}
```

---

## **Итог для Junior:**

1. **Жизненный цикл Broadcast Receiver'а состоит из одного метода — `onReceive()`**
2. **Receiver живет очень коротко** — максимум 10 секунд на выполнение
3. **Динамические receiver'ы** должны быть зарегистрированы и отрегистрированы явно
4. **Статические receiver'ы** создаются и уничтожаются системой автоматически
5. **Не делайте долгих операций** в `onReceive()` — используйте Service или `goAsync()`
6. **Receiver не сохраняет состояние** между вызовами

**Помните:** Broadcast Receiver — это "одноразовый" компонент для быстрой реакции на события, а не для длительной работы!

</details>

<details>
  <summary> Какие изменения были внесены Android 8.0 и выше (уровень API 26) касательно Broadcast Receiver. (белый список транcляций)   </summary>

  **Начиная с Android 8.0 (API 26) были введены серьезные ограничения на работу Broadcast Receiver'ов.**

---

## **Основное изменение: Implicit Broadcast Ban**

### **Что такое Implicit Broadcast?**
```kotlin
// Неявный (implicit) broadcast - без указания конкретного получателя
val intent = Intent("MY_ACTION") // Только action
sendBroadcast(intent)

// Явный (explicit) broadcast - с указанием конкретного компонента
val intent = Intent(this, MyReceiver::class.java) // Указан конкретный класс
intent.action = "MY_ACTION"
sendBroadcast(intent)
```

---

## **Что ЗАПРЕЩЕНО в Android 8.0+:**

### **❌ Нельзя регистрировать в манифесте для неявных broadcast'ов:**
```xml
<!-- AndroidManifest.xml -->
<receiver android:name=".MyReceiver">
    <intent-filter>
        <!-- ЗАПРЕЩЕНО для Android 8.0+ -->
        <action android:name="android.intent.action.AIRPLANE_MODE"/>
        <action android:name="android.net.conn.CONNECTIVITY_CHANGE"/>
        <action android:name="android.intent.action.BATTERY_LOW"/>
    </intent-filter>
</receiver>
```

---

## **Белый список (Exceptions) — что РАБОТАЕТ:**

### **Разрешены в манифесте следующие системные broadcast'ы:**

| Broadcast Action | Описание |
|------------------|----------|
| `ACTION_LOCKED_BOOT_COMPLETED` | Завершение загрузки (устройство заблокировано) |
| `ACTION_BOOT_COMPLETED` | Завершение загрузки системы |
| `ACTION_TIMEZONE_CHANGED` | Изменение часового пояса |
| `ACTION_LOCALE_CHANGED` | Изменение языка/локали |
| `ACTION_USB_ACCESSORY_ATTACHED` | Подключение USB аксессуара |
| `ACTION_USB_ACCESSORY_DETACHED` | Отключение USB аксессуара |
| `ACTION_USB_DEVICE_ATTACHED` | Подключение USB устройства |
| `ACTION_USB_DEVICE_DETACHED` | Отключение USB устройства |
| `ACTION_HEADSET_PLUG` | Подключение/отключение наушников |
| `ACTION_CONNECTION_STATE_CHANGED` | Изменение состояния подключения Bluetooth |
| `ACTION_ACL_CONNECTED` | Подключение Bluetooth устройства |
| `ACTION_ACL_DISCONNECTED` | Отключение Bluetooth устройства |

**Полный список:** [Android Developers — Implicit Broadcast Exceptions](https://developer.android.com/guide/components/broadcast-exceptions)

---

## **Примеры РАБОТАЮЩИХ broadcast'ов в манифесте:**

```xml
<!-- ЭТО РАБОТАЕТ в Android 8.0+ -->
<receiver 
    android:name=".BootReceiver"
    android:exported="true">
    <intent-filter>
        <!-- В белом списке -->
        <action android:name="android.intent.action.BOOT_COMPLETED"/>
        <action android:name="android.intent.action.LOCALE_CHANGED"/>
    </intent-filter>
</receiver>
```

---

## **Примеры НЕРАБОТАЮЩИХ broadcast'ов в манифесте:**

```xml
<!-- ЭТО НЕ РАБОТАЕТ в Android 8.0+ (нужна динамическая регистрация) -->
<receiver android:name=".NetworkReceiver">
    <intent-filter>
        <!-- НЕ в белом списке -->
        <action android:name="android.net.conn.CONNECTIVITY_CHANGE"/>
        <action android:name="android.intent.action.AIRPLANE_MODE"/>
        <action android:name="android.intent.action.BATTERY_CHANGED"/>
    </intent-filter>
</receiver>
```

---

## **Решения для Android 8.0+:**

### **1. Динамическая регистрация (рекомендуется)**
```kotlin
class MainActivity : AppCompatActivity() {
    private lateinit var networkReceiver: BroadcastReceiver
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        // Создаем receiver
        networkReceiver = object : BroadcastReceiver() {
            override fun onReceive(context: Context, intent: Intent) {
                when (intent.action) {
                    ConnectivityManager.CONNECTIVITY_ACTION -> {
                        handleNetworkChange()
                    }
                    Intent.ACTION_AIRPLANE_MODE_CHANGED -> {
                        handleAirplaneMode()
                    }
                }
            }
        }
        
        // Регистрируем динамически
        val filter = IntentFilter().apply {
            addAction(ConnectivityManager.CONNECTIVITY_ACTION)
            addAction(Intent.ACTION_AIRPLANE_MODE_CHANGED)
        }
        registerReceiver(networkReceiver, filter)
    }
    
    override fun onDestroy() {
        super.onDestroy()
        // Не забываем отрегистрировать!
        unregisterReceiver(networkReceiver)
    }
}
```

### **2. JobScheduler / WorkManager (для фоновой работы)**
```kotlin
// Вместо CONNECTIVITY_CHANGE в манифесте
class NetworkWorker(context: Context, params: WorkerParameters) 
    : Worker(context, params) {
    
    override fun doWork(): Result {
        // Проверяем сеть и выполняем работу
        if (isNetworkAvailable()) {
            syncData()
            return Result.success()
        }
        return Result.retry()
    }
}

// Запускаем с условием сети
val constraints = Constraints.Builder()
    .setRequiredNetworkType(NetworkType.CONNECTED)
    .build()

val workRequest = OneTimeWorkRequestBuilder<NetworkWorker>()
    .setConstraints(constraints)
    .build()

WorkManager.getInstance(context).enqueue(workRequest)
```

### **3. Использование новых API**
```kotlin
// Вместо BATTERY_CHANGED broadcast
val batteryManager = getSystemService(BATTERY_SERVICE) as BatteryManager

// Получаем уровень заряда напрямую
val batteryLevel = batteryManager.getIntProperty(
    BatteryManager.BATTERY_PROPERTY_CAPACITY
)

// Или через Broadcast Receiver с динамической регистрацией
val batteryReceiver = object : BroadcastReceiver() {
    override fun onReceive(context: Context, intent: Intent) {
        val level = intent.getIntExtra(BatteryManager.EXTRA_LEVEL, -1)
        val scale = intent.getIntExtra(BatteryManager.EXTRA_SCALE, -1)
        val batteryPercent = level * 100 / scale.toFloat()
    }
}

registerReceiver(batteryReceiver, 
    IntentFilter(Intent.ACTION_BATTERY_CHANGED))
```

---

## **Практический пример миграции:**

### **Было (до Android 8.0):**
```xml
<!-- AndroidManifest.xml -->
<receiver android:name=".MyReceiver">
    <intent-filter>
        <action android:name="android.net.conn.CONNECTIVITY_CHANGE"/>
        <action android:name="android.intent.action.BATTERY_LOW"/>
        <action android:name="android.intent.action.ACTION_POWER_CONNECTED"/>
    </intent-filter>
</receiver>
```

### **Стало (Android 8.0+):**
```kotlin
// 1. Для сетевых событий - динамическая регистрация
class MainActivity : AppCompatActivity() {
    private val connectivityReceiver = object : BroadcastReceiver() {
        override fun onReceive(context: Context, intent: Intent) {
            updateNetworkStatus()
        }
    }
    
    override fun onStart() {
        super.onStart()
        registerReceiver(connectivityReceiver, 
            IntentFilter(ConnectivityManager.CONNECTIVITY_ACTION))
    }
    
    override fun onStop() {
        unregisterReceiver(connectivityReceiver)
        super.onStop()
    }
}

// 2. Для событий батареи - прямые API вызовы или динамическая регистрация
class BatteryMonitor {
    fun getBatteryLevel(context: Context): Float {
        val batteryIntent = context.registerReceiver(
            null,
            IntentFilter(Intent.ACTION_BATTERY_CHANGED)
        ) ?: return 0f
        
        val level = batteryIntent.getIntExtra(BatteryManager.EXTRA_LEVEL, -1)
        val scale = batteryIntent.getIntExtra(BatteryManager.EXTRA_SCALE, -1)
        return level * 100 / scale.toFloat()
    }
}

// 3. Для BOOT_COMPLETED - все еще можно в манифесте (в белом списке)
```
```xml
<!-- Только broadcast'ы из белого списка -->
<receiver 
    android:name=".BootReceiver"
    android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.BOOT_COMPLETED"/>
    </intent-filter>
</receiver>
```

---

## **Кастомные (пользовательские) broadcast'ы:**

### **✅ Явные broadcast'ы РАБОТАЮТ:**
```kotlin
// Явный broadcast (с указанием класса получателя)
val intent = Intent(this, MyReceiver::class.java)
intent.action = "MY_CUSTOM_ACTION"
sendBroadcast(intent)

// В манифесте:
<receiver 
    android:name=".MyReceiver"
    android:exported="true"/> // exported может быть false для внутриприложенных
```

### **❌ Неявные кастомные broadcast'ы НЕ РАБОТАЮТ в манифесте:**
```xml
<!-- НЕ РАБОТАЕТ для кастомных действий -->
<receiver android:name=".MyReceiver">
    <intent-filter>
        <action android:name="com.example.MY_CUSTOM_ACTION"/>
    </intent-filter>
</receiver>
```

---

## **LocalBroadcastManager (также затронут):**

**LocalBroadcastManager устарел**, но если используете:

```kotlin
// Local broadcast'ы работают, но рекомендуется переходить на:
// - LiveData
// - Flow
// - EventBus
LocalBroadcastManager.getInstance(this)
    .sendBroadcast(intent) // Все еще работает
```

---

## **Проверка версии Android в коде:**

```kotlin
fun registerReceivers() {
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
        // Android 8.0+ - только динамическая регистрация
        registerDynamicReceivers()
    } else {
        // Старые версии - можно использовать манифест
        // Но лучше тоже переходить на динамическую
        registerDynamicReceivers()
    }
}

private fun registerDynamicReceivers() {
    // Регистрируем все необходимые receiver'ы динамически
    val filter = IntentFilter().apply {
        addAction(ConnectivityManager.CONNECTIVITY_ACTION)
        addAction(Intent.ACTION_BATTERY_CHANGED)
        addAction(Intent.ACTION_AIRPLANE_MODE_CHANGED)
    }
    
    val receiver = object : BroadcastReceiver() {
        override fun onReceive(context: Context, intent: Intent) {
            // Обработка событий
        }
    }
    
    registerReceiver(receiver, filter)
}
```

---

## **Итог для Junior:**

1. **Android 8.0+ запрещает** регистрацию неявных broadcast'ов в манифесте
2. **Белый список** — около 100 системных broadcast'ов, которые все еще работают
3. **Динамическая регистрация** — основное решение для Android 8.0+
4. **JobScheduler/WorkManager** — лучше для фоновой работы
5. **Явные broadcast'ы** (с указанием класса) все еще работают
6. **Всегда проверяйте** `Build.VERSION.SDK_INT` для совместимости

**Правило:** Если приложение target API 26+ и receiver не в белом списке — регистрируйте его динамически в коде!


</details>

</details>

<details> 
  <summary> <h2> 🌿 Middle </h2> </summary>

  <details>
  <summary> В чем разница между Broadcast Receiver и Content Provider? </summary>

  **Broadcast Receiver и Content Provider — это два совершенно разных компонента Android с разными целями.**

---

## **Краткое сравнение:**

| Параметр | **Broadcast Receiver** | **Content Provider** |
|----------|---------------------|---------------------|
| **Основная цель** | Реакция на события | Управление данными |
| **Работа с данными** | Получает интенты с данными | CRUD операции над данными |
| **Источник данных** | Система, другие приложения | БД, файлы, сеть |
| **Потребитель** | Само приложение | Другие приложения |
| **Безопасность** | Permissions на получение | URI permissions, read/write permissions |

---

## **Broadcast Receiver — "Слушатель событий"**

### **Что делает:**
```kotlin
// Получает broadcast'ы (события)
class MyReceiver : BroadcastReceiver() {
    override fun onReceive(context: Context, intent: Intent) {
        // Реагирует на событие
        when (intent.action) {
            "android.intent.action.BOOT_COMPLETED" -> startApp()
            "android.net.conn.CONNECTIVITY_CHANGE" -> checkNetwork()
        }
    }
}
```

### **Ключевые особенности:**
- **Пассивный** — ждет события
- **Короткоживущий** — работает до 10 секунд
- **Однонаправленный** — только получает данные
- **Примеры использования:**
  - Реакция на изменение сети
  - Запуск при загрузке системы
  - Получение SMS

---

## **Content Provider — "Поставщик данных"**

### **Что делает:**
```kotlin
// Предоставляет доступ к данным
class MyProvider : ContentProvider() {
    override fun query(
        uri: Uri,
        projection: Array<String>?,
        selection: String?,
        selectionArgs: Array<String>?,
        sortOrder: String?
    ): Cursor {
        // Возвращает данные из БД
        val db = databaseHelper.readableDatabase
        return db.query("users", projection, selection, 
                       selectionArgs, null, null, sortOrder)
    }
    
    override fun insert(uri: Uri, values: ContentValues?): Uri {
        // Добавляет данные
        val id = db.insert("users", null, values)
        return ContentUris.withAppendedId(uri, id)
    }
}
```

### **Ключевые особенности:**
- **Активный** — предоставляет API для работы с данными
- **Долгоживущий** — работает пока нужно
- **Двунаправленный** — и отдает, и принимает данные
- **Примеры использования:**
  - Контакты, календарь, медиа
  - Общие данные между приложениями
  - Структурированное хранение

---

## **Архитектурные различия:**

### **Broadcast Receiver: Event-Driven Architecture**
```
[Система/Приложение] --(broadcast)--> [Receiver] --(действие)--> [Логика приложения]
      ↑                                     ↓
  (событие)                           (реакция на событие)
```

### **Content Provider: Client-Server Architecture**
```
[Приложение-клиент] --(запрос)--> [Content Provider] --(данные)--> [Приложение-клиент]
         ↓                               ↓                              ↑
   (нужны данные)                (обработка запроса)             (получает данные)
```

---

## **Примеры использования:**

### **Broadcast Receiver (получение SMS):**
```kotlin
class SmsReceiver : BroadcastReceiver() {
    override fun onReceive(context: Context, intent: Intent) {
        if (intent.action == "android.provider.Telephony.SMS_RECEIVED") {
            val bundle = intent.extras
            val messages = Telephony.Sms.Intents.getMessagesFromIntent(intent)
            // Обработка SMS
            processSms(messages)
        }
    }
}
```

### **Content Provider (контакты телефона):**
```kotlin
// Чтение контактов через Content Provider
val cursor = contentResolver.query(
    ContactsContract.Contacts.CONTENT_URI,
    arrayOf(ContactsContract.Contacts.DISPLAY_NAME),
    null, null, null
)

cursor?.use {
    while (it.moveToNext()) {
        val name = it.getString(0)
        // Используем контакт
    }
}
```

---

## **Детальное сравнение:**

### **1. Жизненный цикл**
**Broadcast Receiver:**
```kotlin
// Мгновенное создание и уничтожение
override fun onReceive(context: Context, intent: Intent) {
    // Живет только пока выполняется этот метод
    // Максимум 10 секунд!
}
```

**Content Provider:**
```kotlin
class MyProvider : ContentProvider() {
    override fun onCreate(): Boolean {
        // Инициализация при создании
        // Живет долго (пока используется)
        return true
    }
    
    override fun onLowMemory() {
        // Очистка при нехватке памяти
    }
}
```

### **2. Безопасность**
**Broadcast Receiver:**
```xml
<!-- Permissions в манифесте -->
<uses-permission android:name="android.permission.RECEIVE_SMS"/>
<uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED"/>

<!-- Защита receiver'а -->
<receiver 
    android:name=".MyReceiver"
    android:permission="com.example.CUSTOM_PERMISSION"
    android:exported="true|false">
```

**Content Provider:**
```xml
<provider
    android:name=".MyProvider"
    android:authorities="com.example.provider"
    android:exported="true|false"
    android:readPermission="com.example.READ_DATA"
    android:writePermission="com.example.WRITE_DATA"
    android:grantUriPermissions="true">
    
    <!-- Path-specific permissions -->
    <path-permission
        android:path="/secret"
        android:permission="com.example.SECRET_ACCESS"/>
</provider>
```

### **3. Взаимодействие с другими приложениями**
**Broadcast Receiver:**
```kotlin
// Отправка broadcast'а другим приложениям
val intent = Intent("com.example.MY_ACTION")
intent.setPackage("com.other.app") // Явное указание пакета
sendBroadcast(intent)

// Получение broadcast'а от системы
registerReceiver(receiver, IntentFilter(Intent.ACTION_BATTERY_LOW))
```

**Content Provider:**
```kotlin
// Предоставление данных другим приложениям
class SharedProvider : ContentProvider() {
    override fun query(uri: Uri, ...): Cursor {
        // Проверяем permission
        context.enforceCallingOrSelfPermission(
            "com.example.READ_DATA", "Need permission")
        
        // Возвращаем данные
        return database.query(...)
    }
}

// Клиентское приложение запрашивает данные
val cursor = contentResolver.query(
    Uri.parse("content://com.example.provider/users"),
    null, null, null, null
)
```

### **4. Работа с данными**
**Broadcast Receiver** (только получение):
```kotlin
// Получает простые данные в Intent
val data = intent.getStringExtra("key")
val number = intent.getIntExtra("count", 0)
```

**Content Provider** (полный CRUD):
```kotlin
// CRUD операции через ContentResolver
class DataManager(val resolver: ContentResolver) {
    
    // CREATE
    fun insertUser(name: String): Uri {
        val values = ContentValues().apply {
            put("name", name)
        }
        return resolver.insert(UsersContract.CONTENT_URI, values)
    }
    
    // READ
    fun getUsers(): List<User> {
        val cursor = resolver.query(
            UsersContract.CONTENT_URI,
            arrayOf("id", "name"),
            null, null, "name ASC"
        )
        // Конвертация cursor в List<User>
    }
    
    // UPDATE
    fun updateUser(id: Long, newName: String): Int {
        val values = ContentValues().apply {
            put("name", newName)
        }
        return resolver.update(
            ContentUris.withAppendedId(UsersContract.CONTENT_URI, id),
            values, null, null
        )
    }
    
    // DELETE
    fun deleteUser(id: Long): Int {
        return resolver.delete(
            ContentUris.withAppendedId(UsersContract.CONTENT_URI, id),
            null, null
        )
    }
}
```

---

## **Практические сценарии:**

### **Сценарий 1: Синхронизация данных при изменении сети**
```kotlin
// Broadcast Receiver для события сети
class NetworkReceiver : BroadcastReceiver() {
    override fun onReceive(context: Context, intent: Intent) {
        // Сеть доступна → запускаем синхронизацию
        val syncIntent = Intent(context, SyncService::class.java)
        context.startService(syncIntent)
    }
}

// Content Provider для доступа к синхронизированным данным
class SyncProvider : ContentProvider() {
    override fun query(uri: Uri, ...): Cursor {
        // Возвращаем синхронизированные данные
        return syncDatabase.query(...)
    }
}
```

### **Сценарий 2: Обновление UI при изменении данных**
```kotlin
// Content Provider уведомляет об изменениях
override fun insert(uri: Uri, values: ContentValues?): Uri {
    val id = db.insert(TABLE_NAME, null, values)
    val newUri = ContentUris.withAppendedId(uri, id)
    
    // Уведомляем всех наблюдателей
    context.contentResolver.notifyChange(newUri, null)
    
    // Можно отправить broadcast для legacy кода
    val intent = Intent("DATA_CHANGED")
    intent.putExtra("table", TABLE_NAME)
    context.sendBroadcast(intent)
    
    return newUri
}
```

---

## **Совместное использование:**

### **Паттерн: Event-Driven Data Updates**
```kotlin
// 1. Content Provider управляет данными
class AppProvider : ContentProvider() {
    fun onDataChanged() {
        // 2. При изменении данных отправляем broadcast
        val intent = Intent("com.example.DATA_UPDATED")
        LocalBroadcastManager.getInstance(context)
            .sendBroadcast(intent)
    }
}

// 3. Broadcast Receiver в Activity
class MainActivity : AppCompatActivity() {
    private val dataReceiver = object : BroadcastReceiver() {
        override fun onReceive(context: Context, intent: Intent) {
            // 4. Обновляем UI при изменении данных
            updateUI()
        }
    }
    
    override fun onStart() {
        LocalBroadcastManager.getInstance(this)
            .registerReceiver(dataReceiver, 
                IntentFilter("com.example.DATA_UPDATED"))
        super.onStart()
    }
}
```

---

## **Modern Alternatives:**

### **Вместо Broadcast Receiver:**
- **LocalBroadcastManager** (устарел)
- **LiveData/Flow** + ViewModel (для UI)
- **WorkManager** для фоновых задач
- **EventBus** (GreenRobot, LiveEventBus)

### **Вместо Content Provider:**
- **Room** + DAO для локальной БД
- **Retrofit** для сетевых данных
- **SharedPreferences**/DataStore для настроек
- **FileProvider** для общего доступа к файлам

---

## **Когда что использовать:**

### **✅ Используйте Broadcast Receiver когда:**
- Нужно реагировать на системные события
- Приложение должно запускаться по событию (BOOT_COMPLETED)
- Получение push-уведомлений через FCM
- Простая коммуникация между компонентами

### **✅ Используйте Content Provider когда:**
- Нужно предоставить данные другим приложениям
- Работа со структурированными данными (БД)
- Интеграция с системными приложениями (контакты, календарь)
- Сложные запросы с сортировкой, фильтрацией
- Нужен ContentObserver для автоматических обновлений

---

## **Итог для Middle разработчика:**

1. **Broadcast Receiver** — для **событий** (event-driven)
2. **Content Provider** — для **данных** (data management)
3. **Receiver** — пассивный слушатель, живет мгновения
4. **Provider** — активный сервер данных, живет долго
5. В современных приложениях оба используются реже (есть альтернативы)
6. Но понимание необходимо для работы с легаси кодом и системными компонентами

**Ключевая аналогия:**  
**Broadcast Receiver** — как радиоприемник (ловит сигналы)  
**Content Provider** — как библиотека (хранит и выдает книги)

  </details>


   <details>
  <summary> Работает ли Broadcast Receiver в бэкграунде? Пример  </summary>

**Короткий ответ:** Да, Broadcast Receiver может работать в бэкграунде, но с серьезными ограничениями начиная с Android 8.0 (API 26).

---

## **Как работает в бэкграунде:**

### **1. Статический Receiver (в манифесте)**
```xml
<!-- AndroidManifest.xml -->
<receiver 
    android:name=".BootReceiver"
    android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.BOOT_COMPLETED"/>
    </intent-filter>
</receiver>
```

```kotlin
class BootReceiver : BroadcastReceiver() {
    override fun onReceive(context: Context, intent: Intent) {
        // Выполнится при загрузке устройства, даже если приложение закрыто
        if (intent.action == Intent.ACTION_BOOT_COMPLETED) {
            startMyService(context)
        }
    }
    
    private fun startMyService(context: Context) {
        val serviceIntent = Intent(context, BackgroundService::class.java)
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            context.startForegroundService(serviceIntent)
        } else {
            context.startService(serviceIntent)
        }
    }
}
```

### **2. Динамический Receiver (в бэкграундном Service)**
```kotlin
class BackgroundService : Service() {
    private lateinit var receiver: BroadcastReceiver
    
    override fun onCreate() {
        super.onCreate()
        
        // Регистрируем receiver в сервисе
        receiver = object : BroadcastReceiver() {
            override fun onReceive(context: Context, intent: Intent) {
                when (intent.action) {
                    Intent.ACTION_SCREEN_ON -> {
                        Log.d("Receiver", "Screen turned ON")
                        // Делаем что-то в бэкграунде
                        performBackgroundTask()
                    }
                    Intent.ACTION_SCREEN_OFF -> {
                        Log.d("Receiver", "Screen turned OFF")
                    }
                }
            }
        }
        
        val filter = IntentFilter().apply {
            addAction(Intent.ACTION_SCREEN_ON)
            addAction(Intent.ACTION_SCREEN_OFF)
        }
        
        registerReceiver(receiver, filter)
    }
    
    override fun onDestroy() {
        super.onDestroy()
        unregisterReceiver(receiver)
    }
    
    override fun onBind(intent: Intent?): IBinder? = null
}
```

---

## **Практический пример: Мониторинг сети в бэкграунде**

### **Полная реализация:**

```kotlin
// 1. Сервис, который будет работать в бэкграунде
class NetworkMonitorService : Service() {
    
    companion object {
        fun start(context: Context) {
            val intent = Intent(context, NetworkMonitorService::class.java)
            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
                context.startForegroundService(intent)
            } else {
                context.startService(intent)
            }
        }
    }
    
    private lateinit var networkReceiver: BroadcastReceiver
    private lateinit var notificationManager: NotificationManager
    private val notificationId = 1001
    
    override fun onCreate() {
        super.onCreate()
        
        // Создаем канал уведомлений (для Foreground Service)
        createNotificationChannel()
        
        // Запускаем как Foreground Service (обязательно для Android 8+)
        startForeground(notificationId, createNotification("Мониторинг сети активен"))
        
        // Инициализируем receiver для мониторинга сети
        initNetworkReceiver()
    }
    
    private fun initNetworkReceiver() {
        networkReceiver = object : BroadcastReceiver() {
            override fun onReceive(context: Context, intent: Intent) {
                val connectivityManager = getSystemService(CONNECTIVITY_SERVICE) 
                        as ConnectivityManager
                
                if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
                    val network = connectivityManager.activeNetwork
                    val capabilities = connectivityManager
                        .getNetworkCapabilities(network)
                    
                    val hasInternet = capabilities?.let {
                        it.hasTransport(NetworkCapabilities.TRANSPORT_WIFI) ||
                        it.hasTransport(NetworkCapabilities.TRANSPORT_CELLULAR)
                    } ?: false
                    
                    if (hasInternet) {
                        onNetworkAvailable()
                    } else {
                        onNetworkLost()
                    }
                } else {
                    @Suppress("DEPRECATION")
                    val networkInfo = connectivityManager.activeNetworkInfo
                    if (networkInfo?.isConnected == true) {
                        onNetworkAvailable()
                    } else {
                        onNetworkLost()
                    }
                }
            }
        }
        
        // Регистрируем receiver
        val filter = IntentFilter(ConnectivityManager.CONNECTIVITY_ACTION)
        registerReceiver(networkReceiver, filter)
        
        Log.d("NetworkMonitor", "Receiver зарегистрирован в бэкграунде")
    }
    
    private fun onNetworkAvailable() {
        Log.d("NetworkMonitor", "Сеть доступна")
        
        // Показываем уведомление
        showNotification("Интернет подключен", "Сеть доступна для использования")
        
        // Можем запустить синхронизацию данных
        startDataSync()
    }
    
    private fun onNetworkLost() {
        Log.d("NetworkMonitor", "Сеть потеряна")
        showNotification("Интернет отключен", "Нет подключения к сети")
        
        // Сохраняем состояние или останавливаем операции
        pauseDataSync()
    }
    
    private fun startDataSync() {
        // Запускаем синхронизацию в фоне
        CoroutineScope(Dispatchers.IO).launch {
            try {
                // Имитация синхронизации
                delay(5000)
                Log.d("NetworkMonitor", "Данные синхронизированы")
                showNotification("Синхронизация", "Данные успешно обновлены")
            } catch (e: Exception) {
                Log.e("NetworkMonitor", "Ошибка синхронизации", e)
            }
        }
    }
    
    private fun pauseDataSync() {
        // Приостанавливаем синхронизацию
        Log.d("NetworkMonitor", "Синхронизация приостановлена")
    }
    
    private fun createNotificationChannel() {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            val channel = NotificationChannel(
                "network_monitor",
                "Мониторинг сети",
                NotificationManager.IMPORTANCE_LOW
            ).apply {
                description = "Уведомления о состоянии сети"
            }
            
            notificationManager = getSystemService(NotificationManager::class.java)
            notificationManager.createNotificationChannel(channel)
        }
    }
    
    private fun createNotification(text: String): Notification {
        return NotificationCompat.Builder(this, "network_monitor")
            .setContentTitle("Монитор сети")
            .setContentText(text)
            .setSmallIcon(android.R.drawable.ic_dialog_info)
            .setOngoing(true)
            .build()
    }
    
    private fun showNotification(title: String, text: String) {
        val notification = NotificationCompat.Builder(this, "network_monitor")
            .setContentTitle(title)
            .setContentText(text)
            .setSmallIcon(android.R.drawable.ic_dialog_info)
            .setAutoCancel(true)
            .build()
        
        notificationManager.notify(notificationId + 1, notification)
    }
    
    override fun onDestroy() {
        // Важно: отменяем регистрацию receiver'а
        unregisterReceiver(networkReceiver)
        Log.d("NetworkMonitor", "Сервис остановлен, receiver отрегистрирован")
        super.onDestroy()
    }
    
    override fun onBind(intent: Intent?): IBinder? = null
}
```

### **2. Activity для управления сервисом:**

```kotlin
class MainActivity : AppCompatActivity() {
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        
        // Проверяем разрешение на уведомления (Android 13+)
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU) {
            if (checkSelfPermission(Manifest.permission.POST_NOTIFICATIONS) 
                != PackageManager.PERMISSION_GRANTED) {
                requestPermissions(
                    arrayOf(Manifest.permission.POST_NOTIFICATIONS),
                    100
                )
            }
        }
        
        findViewById<Button>(R.id.startBtn).setOnClickListener {
            // Запускаем сервис мониторинга
            NetworkMonitorService.start(this)
            Toast.makeText(this, "Мониторинг запущен", Toast.LENGTH_SHORT).show()
        }
        
        findViewById<Button>(R.id.stopBtn).setOnClickListener {
            // Останавливаем сервис
            stopService(Intent(this, NetworkMonitorService::class.java))
            Toast.makeText(this, "Мониторинг остановлен", Toast.LENGTH_SHORT).show()
        }
    }
}
```

### **3. Манифест с разрешениями:**

```xml
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="android.permission.FOREGROUND_SERVICE" />
<uses-permission android:name="android.permission.POST_NOTIFICATIONS" />

<application>
    <service
        android:name=".NetworkMonitorService"
        android:exported="false" />
    
    <!-- Для Android 8.0+ нельзя регистрировать CONNECTIVITY_ACTION в манифесте -->
    <!-- Используем динамическую регистрацию в сервисе -->
    
    <!-- Но BOOT_COMPLETED все еще можно в манифесте -->
    <receiver
        android:name=".BootReceiver"
        android:exported="true"
        android:enabled="true">
        <intent-filter>
            <action android:name="android.intent.action.BOOT_COMPLETED" />
        </intent-filter>
    </receiver>
</application>
```

---

## **Важные ограничения Android 8.0+:**

### **1. Background Execution Limits**
```kotlin
// ❌ НЕ РАБОТАЕТ в Android 8.0+:
class BadReceiver : BroadcastReceiver() {
    override fun onReceive(context: Context, intent: Intent) {
        // Этот receiver не получит CONNECTIVITY_ACTION, 
        // если зарегистрирован в манифесте
        // и приложение в бэкграунде
    }
}
```

### **2. Правильный подход для Android 8.0+:**
```kotlin
// ✅ РАБОТАЕТ:
// 1. Запускаем Foreground Service
// 2. В сервисе регистрируем receiver динамически
// 3. Receiver будет работать пока жив сервис

class BackgroundReceiverService : Service() {
    override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
        // Делаем сервис foreground
        startForeground(NOTIFICATION_ID, createNotification())
        
        // Регистрируем receiver
        val receiver = MyReceiver()
        val filter = IntentFilter(ConnectivityManager.CONNECTIVITY_ACTION)
        registerReceiver(receiver, filter)
        
        return START_STICKY
    }
}
```

---

## **Пример: Мониторинг батареи в бэкграунде**

```kotlin
class BatteryMonitorService : Service() {
    
    private lateinit var batteryReceiver: BroadcastReceiver
    
    override fun onCreate() {
        super.onCreate()
        startForegroundServiceWithNotification()
        setupBatteryReceiver()
    }
    
    private fun setupBatteryReceiver() {
        batteryReceiver = object : BroadcastReceiver() {
            override fun onReceive(context: Context, intent: Intent) {
                when (intent.action) {
                    Intent.ACTION_BATTERY_CHANGED -> {
                        val level = intent.getIntExtra(
                            BatteryManager.EXTRA_LEVEL, -1
                        )
                        val scale = intent.getIntExtra(
                            BatteryManager.EXTRA_SCALE, -1
                        )
                        val batteryPercent = level * 100 / scale.toFloat()
                        
                        if (batteryPercent < 20) {
                            showLowBatteryWarning(batteryPercent)
                        }
                    }
                    
                    Intent.ACTION_POWER_CONNECTED -> {
                        Log.d("Battery", "Зарядка подключена")
                        // Можно остановить энергозатратные операции
                    }
                    
                    Intent.ACTION_POWER_DISCONNECTED -> {
                        Log.d("Battery", "Зарядка отключена")
                    }
                }
            }
        }
        
        val filter = IntentFilter().apply {
            addAction(Intent.ACTION_BATTERY_CHANGED)
            addAction(Intent.ACTION_POWER_CONNECTED)
            addAction(Intent.ACTION_POWER_DISCONNECTED)
        }
        
        registerReceiver(batteryReceiver, filter)
    }
    
    private fun showLowBatteryWarning(percent: Float) {
        // Показываем уведомление при низком заряде
        val notification = NotificationCompat.Builder(this, "battery_channel")
            .setContentTitle("Низкий заряд батареи")
            .setContentText("Заряд: ${"%.1f".format(percent)}%")
            .setSmallIcon(android.R.drawable.ic_dialog_alert)
            .setPriority(NotificationCompat.PRIORITY_HIGH)
            .setAutoCancel(true)
            .build()
        
        NotificationManagerCompat.from(this).notify(2001, notification)
    }
}
```

---

## **Лучшие практики для бэкграунд работы:**

### **1. Используйте WorkManager вместо long-running receivers:**
```kotlin
// Вместо постоянного мониторинга в receiver
class NetworkCheckWorker(
    context: Context,
    params: WorkerParameters
) : CoroutineWorker(context, params) {
    
    override suspend fun doWork(): Result {
        // Периодически проверяем сеть
        val hasNetwork = checkNetworkAvailability()
        
        if (hasNetwork) {
            // Выполняем работу
            syncData()
            return Result.success()
        }
        
        return Result.retry()
    }
}

// Запускаем периодическую работу
val constraints = Constraints.Builder()
    .setRequiredNetworkType(NetworkType.CONNECTED)
    .build()

val workRequest = PeriodicWorkRequestBuilder<NetworkCheckWorker>(
    15, TimeUnit.MINUTES
).setConstraints(constraints).build()

WorkManager.getInstance(context).enqueue(workRequest)
```

### **2. Ограничивайте время работы в onReceive():**
```kotlin
class EfficientReceiver : BroadcastReceiver() {
    override fun onReceive(context: Context, intent: Intent) {
        // Используем goAsync() для длительных операций
        val pendingResult = goAsync()
        
        CoroutineScope(Dispatchers.IO).launch {
            try {
                // Длительная операция
                processInBackground(intent)
            } finally {
                pendingResult.finish()
            }
        }
    }
}
```

### **3. Учитывайте Doze mode и App Standby:**
```kotlin
// Проверяем, не в Doze ли режиме
val powerManager = getSystemService(POWER_SERVICE) as PowerManager

if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
    if (powerManager.isDeviceIdleMode) {
        // Устройство в Doze режиме
        // Некоторые broadcast'ы не будут доставлены
        Log.d("Receiver", "Device in Doze mode")
    }
}
```

---

## **Итог для Middle разработчика:**

1. **Broadcast Receiver МОЖЕТ работать в бэкграунде**, но:
   - Через **Foreground Service** (Android 8.0+)
   - С **динамической регистрацией** в сервисе
   - Для **системных событий из белого списка** (BOOT_COMPLETED)

2. **Статические receivers** в манифесте ограничены Android 8.0+

3. **Лучшие практики:**
   - Используйте `WorkManager` для периодических задач
   - Для постоянного мониторинга — Foreground Service + динамический receiver
   - Всегда отменяйте регистрацию receivers
   - Не делайте длительных операций в `onReceive()`

4. **Примеры рабочих сценариев:**
   - Мониторинг сети/батареи в Foreground Service
   - Автозапуск при загрузке через BOOT_COMPLETED
   - Реакция на системные события в активном сервисе

**Ключевое правило:** На Android 8.0+ бэкграунд работа требует Foreground Service с уведомлением пользователя.

  </details>

  
</details>


<details> 
  <summary> <h2> 🌳 Senior </h2> </summary>

  <details>
  <summary> Как перезапустить Broadcast Receiver после перезагрузки девайса? </summary>

   **Senior-ответ: Есть несколько стратегий с разными trade-offs. Вот полное руководство:**

---

## **1. Основной подход: Использование BOOT_COMPLETED**

### **Проблема:** После перезагрузки все динамически зарегистрированные receivers сбрасываются.

### **Решение:** Регистрировать receiver заново при загрузке системы.

```kotlin
// 1. Receiver для перехвата события загрузки
class BootReceiver : BroadcastReceiver() {
    override fun onReceive(context: Context, intent: Intent) {
        if (intent.action == Intent.ACTION_BOOT_COMPLETED || 
            intent.action == Intent.ACTION_LOCKED_BOOT_COMPLETED) {
            
            Log.d("BootReceiver", "Device booted, restarting receivers")
            
            // Перезапускаем наши receivers
            restartMyReceivers(context)
            
            // Или запускаем сервис, который зарегистрирует receivers
            startMonitoringService(context)
        }
    }
    
    private fun restartMyReceivers(context: Context) {
        // Используем WorkManager для отложенного запуска
        val workRequest = OneTimeWorkRequestBuilder<ReceiverRestartWorker>()
            .setInitialDelay(30, TimeUnit.SECONDS) // Даем системе время на инициализацию
            .setConstraints(
                Constraints.Builder()
                    .setRequiresCharging(false)
                    .setRequiresBatteryNotLow(false)
                    .build()
            )
            .build()
        
        WorkManager.getInstance(context).enqueue(workRequest)
    }
    
    private fun startMonitoringService(context: Context) {
        val intent = Intent(context, BackgroundMonitorService::class.java)
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            context.startForegroundService(intent)
        } else {
            context.startService(intent)
        }
    }
}
```

### **Worker для перезапуска:**
```kotlin
class ReceiverRestartWorker(
    context: Context,
    params: WorkerParameters
) : CoroutineWorker(context, params) {
    
    override suspend fun doWork(): Result {
        return try {
            // Регистрируем receivers заново
            ReceiversManager.registerAllReceivers(applicationContext)
            Result.success()
        } catch (e: Exception) {
            Result.retry()
        }
    }
}
```

### **Манифест для Boot Receiver:**
```xml
<!-- Разрешения -->
<uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED" />

<!-- Для Android 10+ нужно объявить BOOT_COMPLETED как предварительно объявленный -->
<queries>
    <intent>
        <action android:name="android.intent.action.BOOT_COMPLETED" />
    </intent>
</queries>

<receiver
    android:name=".BootReceiver"
    android:enabled="true"
    android:exported="true"
    android:directBootAware="true" <!-- Для Android 7.0+ и Direct Boot -->
    android:permission="android.permission.RECEIVE_BOOT_COMPLETED">
    
    <intent-filter android:priority="1000">
        <!-- Основное событие загрузки -->
        <action android:name="android.intent.action.BOOT_COMPLETED" />
        
        <!-- Для Android 10+ (API 29) и выше -->
        <action android:name="android.intent.action.LOCKED_BOOT_COMPLETED" />
        
        <!-- Для быстрой загрузки (некоторые производители) -->
        <action android:name="android.intent.action.QUICKBOOT_POWERON" />
        
        <!-- Для некоторых кастомных прошивок -->
        <action android:name="com.htc.intent.action.QUICKBOOT_POWERON" />
    </intent-filter>
</receiver>
```

---

## **2. Стратегия: Direct Boot Aware (Android 7.0+)**

### **Для работы до разблокировки устройства:**
```kotlin
// Receiver, который работает даже до ввода PIN/пароля
class DirectBootReceiver : BroadcastReceiver() {
    
    companion object {
        const val PREFS_DIRECT_BOOT = "direct_boot_prefs"
    }
    
    override fun onReceive(context: Context, intent: Intent) {
        // Используем контекст для Direct Boot
        val deviceProtectedContext = context.createDeviceProtectedStorageContext()
        
        // Сохраняем факт перезагрузки
        val prefs = deviceProtectedContext.getSharedPreferences(
            PREFS_DIRECT_BOOT, 
            Context.MODE_PRIVATE
        )
        prefs.edit().putBoolean("device_rebooted", true).apply()
        
        // Планируем перезапуск receivers после разблокировки
        scheduleReceiverRestart(deviceProtectedContext)
    }
    
    private fun scheduleReceiverRestart(context: Context) {
        // Используем AlarmManager для точного времени
        val alarmManager = context.getSystemService(ALARM_SERVICE) as AlarmManager
        val restartIntent = Intent(context, ReceiverRestartReceiver::class.java)
        val pendingIntent = PendingIntent.getBroadcast(
            context,
            0,
            restartIntent,
            PendingIntent.FLAG_UPDATE_CURRENT or 
            PendingIntent.FLAG_IMMUTABLE
        )
        
        // Запускаем через 1 минуту после загрузки
        val triggerTime = System.currentTimeMillis() + 60000
        
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
            alarmManager.setExactAndAllowWhileIdle(
                AlarmManager.RTC_WAKEUP,
                triggerTime,
                pendingIntent
            )
        } else {
            alarmManager.setExact(
                AlarmManager.RTC_WAKEUP,
                triggerTime,
                pendingIntent
            )
        }
    }
}
```

### **Манифест для Direct Boot:**
```xml
<receiver
    android:name=".DirectBootReceiver"
    android:directBootAware="true"
    android:exported="true">
    
    <intent-filter>
        <action android:name="android.intent.action.LOCKED_BOOT_COMPLETED" />
    </intent-filter>
</receiver>

<!-- Дополнительный receiver для работы после разблокировки -->
<receiver
    android:name=".UserUnlockReceiver"
    android:exported="false">
    
    <intent-filter>
        <action android:name="android.intent.action.USER_UNLOCKED" />
    </intent-filter>
</receiver>
```

---

## **3. Продвинутая стратегия: Комбинированный подход**

### **Architecture: Централизованный менеджер receivers**
```kotlin
object ReceiversManager {
    
    private val registeredReceivers = mutableMapOf<String, BroadcastReceiver>()
    private var isInitialized = false
    
    // Инициализация при запуске приложения
    fun initialize(context: Context) {
        if (isInitialized) return
        
        // Проверяем, была ли перезагрузка
        checkAndHandleReboot(context)
        
        // Регистрируем системные receivers
        registerSystemReceivers(context)
        
        // Регистрируем кастомные receivers
        registerCustomReceivers(context)
        
        isInitialized = true
        saveInitializationState(context, true)
    }
    
    // Проверка перезагрузки
    private fun checkAndHandleReboot(context: Context) {
        val prefs = getBootPreferences(context)
        val lastBootTime = prefs.getLong("last_boot_time", 0)
        val currentBootTime = System.currentTimeMillis()
        
        // Если разница больше 30 секунд - считаем что была перезагрузка
        if (currentBootTime - lastBootTime > 30000) {
            onDeviceRebooted(context)
        }
        
        // Обновляем время загрузки
        prefs.edit().putLong("last_boot_time", currentBootTime).apply()
    }
    
    private fun onDeviceRebooted(context: Context) {
        Log.i("ReceiversManager", "Device reboot detected")
        
        // 1. Восстанавливаем состояние
        restoreReceiverState(context)
        
        // 2. Перезапускаем необходимые receivers
        restartCriticalReceivers(context)
        
        // 3. Уведомляем другие компоненты
        notifyRebootEvent(context)
    }
    
    // Регистрация всех receivers
    fun registerAllReceivers(context: Context) {
        // Сетевые события
        registerNetworkReceiver(context)
        
        // События батареи
        registerBatteryReceiver(context)
        
        // Кастомные события
        registerCustomEventReceivers(context)
        
        // Screen on/off
        registerScreenReceiver(context)
    }
    
    private fun registerNetworkReceiver(context: Context) {
        val receiver = object : BroadcastReceiver() {
            override fun onReceive(context: Context, intent: Intent) {
                // Обработка сетевых событий
                NetworkMonitor.handleNetworkChange(context, intent)
            }
        }
        
        val filter = IntentFilter().apply {
            addAction(ConnectivityManager.CONNECTIVITY_ACTION)
            addAction(ConnectivityManager.CONNECTIVITY_CHANGE)
        }
        
        context.registerReceiver(receiver, filter)
        registeredReceivers["network"] = receiver
        
        Log.d("ReceiversManager", "Network receiver registered")
    }
    
    // Сохранение состояния
    fun saveReceiverState(context: Context) {
        val state = JSONObject().apply {
            put("network_monitoring", true)
            put("battery_monitoring", true)
            put("last_update", System.currentTimeMillis())
        }
        
        val prefs = context.getSharedPreferences("receivers_state", Context.MODE_PRIVATE)
        prefs.edit().putString("state", state.toString()).apply()
    }
    
    // Восстановление состояния после перезагрузки
    private fun restoreReceiverState(context: Context) {
        val prefs = context.getSharedPreferences("receivers_state", Context.MODE_PRIVATE)
        val stateJson = prefs.getString("state", null)
        
        stateJson?.let {
            try {
                val state = JSONObject(it)
                val shouldMonitorNetwork = state.optBoolean("network_monitoring", false)
                
                if (shouldMonitorNetwork) {
                    registerNetworkReceiver(context)
                }
                
                // Восстанавливаем другие receivers
                // ...
                
            } catch (e: Exception) {
                Log.e("ReceiversManager", "Error restoring receiver state", e)
            }
        }
    }
}
```

---

## **4. Использование WorkManager для гарантированного перезапуска**

### **Периодическая проверка и перерегистрация:**
```kotlin
class ReceiverHealthCheckWorker(
    context: Context,
    params: WorkerParameters
) : CoroutineWorker(context, params) {
    
    override suspend fun doWork(): Result {
        return try {
            // 1. Проверяем, активны ли receivers
            val receiversActive = checkReceiversActive()
            
            if (!receiversActive) {
                // 2. Если нет - перезапускаем
                restartAllReceivers()
                
                // 3. Логируем событие
                Analytics.logEvent("receivers_restarted", mapOf(
                    "reason" to "health_check_failed",
                    "timestamp" to System.currentTimeMillis()
                ))
            }
            
            // 4. Сохраняем состояние
            saveHealthStatus(true)
            
            Result.success()
        } catch (e: Exception) {
            Log.e("ReceiverHealth", "Health check failed", e)
            Result.retry()
        }
    }
    
    private suspend fun checkReceiversActive(): Boolean {
        // Проверяем различные индикаторы активности
        
        // 1. Проверяем через ActivityManager
        val activityManager = applicationContext.getSystemService(
            Context.ACTIVITY_SERVICE
        ) as ActivityManager
        
        val runningServices = activityManager.getRunningServices(100)
        val ourServiceRunning = runningServices.any { 
            it.service.className.contains("BackgroundMonitorService")
        }
        
        // 2. Проверяем через BroadcastManager (кастомная логика)
        val prefs = applicationContext.getSharedPreferences(
            "receiver_status",
            Context.MODE_PRIVATE
        )
        val lastEventTime = prefs.getLong("last_event_time", 0)
        val timeSinceLastEvent = System.currentTimeMillis() - lastEventTime
        
        // Если не было событий более 5 минут - считаем неактивным
        return ourServiceRunning && timeSinceLastEvent < 5 * 60 * 1000
    }
}
```

### **Настройка периодической работы:**
```kotlin
object ReceiverScheduler {
    
    fun scheduleHealthChecks(context: Context) {
        val constraints = Constraints.Builder()
            .setRequiredNetworkType(NetworkType.CONNECTED)
            .setRequiresBatteryNotLow(false)
            .setRequiresCharging(false)
            .build()
        
        // Ежедневная проверка
        val dailyCheck = PeriodicWorkRequestBuilder<ReceiverHealthCheckWorker>(
            1, TimeUnit.DAYS,
            15, TimeUnit.MINUTES
        ).setConstraints(constraints)
         .setBackoffCriteria(
             BackoffPolicy.EXPONENTIAL,
             30, TimeUnit.MINUTES
         ).build()
        
        WorkManager.getInstance(context).enqueueUniquePeriodicWork(
            "receiver_health_check",
            ExistingPeriodicWorkPolicy.KEEP,
            dailyCheck
        )
        
        // Немедленная проверка после загрузки
        val immediateCheck = OneTimeWorkRequestBuilder<ReceiverHealthCheckWorker>()
            .setInitialDelay(2, TimeUnit.MINUTES) // Даем время на инициализацию
            .setConstraints(
                Constraints.Builder()
                    .setRequiresDeviceIdle(false)
                    .build()
            ).build()
        
        WorkManager.getInstance(context).enqueue(immediateCheck)
    }
}
```

---

## **5. Обработка edge cases и производители**

### **Для разных производителей Android:**
```kotlin
object ManufacturerSpecificHandler {
    
    fun handleBootForManufacturer(context: Context, manufacturer: String) {
        when (manufacturer.lowercase()) {
            "samsung" -> handleSamsungBoot(context)
            "xiaomi", "redmi", "poco" -> handleXiaomiBoot(context)
            "huawei", "honor" -> handleHuaweiBoot(context)
            "oneplus" -> handleOnePlusBoot(context)
            else -> handleGenericBoot(context)
        }
    }
    
    private fun handleXiaomiBoot(context: Context) {
        // Xiaomi имеет агрессивную оптимизацию батареи
        // Нужно добавлять приложение в автозапуск
        
        val intent = Intent()
        intent.component = ComponentName(
            "com.miui.securitycenter",
            "com.miui.permcenter.autostart.AutoStartManagementActivity"
        )
        
        try {
            context.startActivity(intent)
        } catch (e: Exception) {
            // Альтернативный путь
            showAutoStartInstructions(context)
        }
        
        // Также регистрируем receivers
        ReceiversManager.registerAllReceivers(context)
    }
    
    private fun handleHuaweiBoot(context: Context) {
        // Huawei Protected Apps
        val intent = Intent()
        intent.component = ComponentName(
            "com.huawei.systemmanager",
            "com.huawei.systemmanager.optimize.process.ProtectActivity"
        )
        
        try {
            context.startActivity(intent)
        } catch (e: Exception) {
            // Запасной вариант
        }
    }
}
```

---

## **6. Мониторинг и диагностика**

### **Система логирования состояния receivers:**
```kotlin
object ReceiverDiagnostics {
    
    fun logReceiverStatus(context: Context) {
        val status = buildReceiverStatus(context)
        val logEntry = createLogEntry(status)
        
        // Сохраняем в локальную БД
        saveToDatabase(logEntry)
        
        // Отправляем на сервер (если нужно)
        if (isNetworkAvailable(context)) {
            sendToAnalytics(logEntry)
        }
    }
    
    private fun buildReceiverStatus(context: Context): Map<String, Any> {
        return mapOf(
            "timestamp" to System.currentTimeMillis(),
            "boot_count" to getBootCount(context),
            "last_boot_time" to getLastBootTime(context),
            "registered_receivers" to getRegisteredReceiversCount(),
            "battery_optimization" to isBatteryOptimized(context),
            "manufacturer" to Build.MANUFACTURER,
            "android_version" to Build.VERSION.SDK_INT,
            "app_version" to getAppVersion(context)
        )
    }
    
    private fun getBootCount(context: Context): Int {
        val prefs = context.getSharedPreferences("boot_stats", Context.MODE_PRIVATE)
        return prefs.getInt("boot_count", 0)
    }
    
    fun incrementBootCount(context: Context) {
        val prefs = context.getSharedPreferences("boot_stats", Context.MODE_PRIVATE)
        val current = prefs.getInt("boot_count", 0)
        prefs.edit().putInt("boot_count", current + 1).apply()
    }
}
```

---

## **7. Полная реализация с учетом всех сценариев**

### **Main Application Class:**
```kotlin
class MyApplication : Application() {
    
    override fun onCreate() {
        super.onCreate()
        
        // Инициализация при запуске
        ReceiversManager.initialize(this)
        
        // Планируем периодические проверки
        ReceiverScheduler.scheduleHealthChecks(this)
        
        // Регистрируем Activity Lifecycle Callback для отслеживания
        registerActivityLifecycleCallbacks(AppLifecycleTracker())
        
        // Проверяем, была ли перезагрузка
        checkForReboot()
    }
    
    private fun checkForReboot() {
        val prefs = getSharedPreferences("system_state", MODE_PRIVATE)
        val lastAppStart = prefs.getLong("last_app_start", 0)
        val currentTime = System.currentTimeMillis()
        
        // Если прошло больше 10 минут с последнего старта - вероятно была перезагрузка
        if (currentTime - lastAppStart > 10 * 60 * 1000) {
            onPossibleReboot()
        }
        
        prefs.edit().putLong("last_app_start", currentTime).apply()
    }
    
    private fun onPossibleReboot() {
        // Перезапускаем critical receivers
        ReceiversManager.restartCriticalReceivers(this)
        
        // Логируем событие
        ReceiverDiagnostics.logReceiverStatus(this)
        ReceiverDiagnostics.incrementBootCount(this)
    }
}

// Трекер жизненного цикла
class AppLifecycleTracker : Application.ActivityLifecycleCallbacks {
    
    override fun onActivityResumed(activity: Activity) {
        // При возвращении в приложение проверяем состояние receivers
        if (activity is MainActivity) {
            CoroutineScope(Dispatchers.IO).launch {
                delay(5000) // Даем время на инициализацию
                ReceiversManager.verifyReceiversActive(activity)
            }
        }
    }
}
```

### **Backup Agent для сохранения состояния (опционально):**
```kotlin
class ReceiverBackupAgent : BackupAgentHelper() {
    
    override fun onCreate() {
        super.onCreate()
        
        // Сохраняем состояние receivers в backup
        val helper = SharedPreferencesBackupHelper(this, "receivers_state")
        addHelper("receivers_backup", helper)
    }
    
    override fun onRestore(
        data: BackupDataInput?, 
        appVersionCode: Int, 
        newState: ParcelFileDescriptor?
    ) {
        super.onRestore(data, appVersionCode, newState)
        
        // После восстановления из backup перезапускаем receivers
        ReceiversManager.restoreFromBackup(this)
    }
}
```

---

## **Итоговая стратегия для Senior:**

1. **Основной триггер:** `BOOT_COMPLETED` + `LOCKED_BOOT_COMPLETED`
2. **Резервный механизм:** `WorkManager` с периодической проверкой
3. **State management:** Сохранение/восстановление состояния receivers
4. **Manufacturer handling:** Особые случаи для Xiaomi, Huawei и др.
5. **Диагностика:** Логирование и мониторинг состояния
6. **Direct Boot:** Поддержка Android 7.0+ для работы до разблокировки
7. **Graceful degradation:** Постепенное восстановление при ошибках

### **Критические моменты:**
- Всегда используйте `WorkManager` для отложенного перезапуска (дайте системе время)
- Сохраняйте состояние receivers в SharedPreferences или Room
- Обрабатывайте случаи, когда BOOT_COMPLETED не срабатывает (некоторые кастомные ROM)
- Учитывайте Battery Optimization на разных производителях
- Тестируйте на реальных устройствах разных брендов

### **Production-ready код должен включать:**
- Exponential backoff при повторных попытках
- Analytics для отслеживания успешности перезапуска
- Fallback механизмы
- Конфигурацию через remote config (какие receivers перезапускать)
- A/B тестирование разных стратегий

  </details>

  <details>
  <summary> Чем будет отличаться поведение Broadcast Receiver'а при регистрации через Активити и Манифест?  </summary>

  **Senior-ответ: Различия фундаментальны и затрагивают жизненный цикл, производительность, безопасность и архитектурные решения.**

---

## **Архитектурное сравнение**

| Аспект | **Регистрация в Activity** | **Регистрация в Manifest** |
|--------|----------------------------|----------------------------|
| **Жизненный цикл** | Привязан к жизненному циклу Activity | Независим от UI компонентов |
| **Время жизни** | Короткое (пока Activity alive) | Долгое (пока приложение установлено) |
| **Производительность** | Высокая (нет системного overhead) | Низкая (система должна инстанциировать) |
| **Безопасность** | Локальный scope, безопаснее | Глобальный scope, требует защиты |
| **Use Case** | UI-реактивные события | Системные/критические события |

---

## **1. Жизненный цикл и управление памятью**

### **Activity-registered (динамический):**
```kotlin
class MainActivity : AppCompatActivity() {
    private lateinit var networkReceiver: BroadcastReceiver
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate()
        
        networkReceiver = object : BroadcastReceiver() {
            override fun onReceive(context: Context, intent: Intent) {
                // ОБРАТИТЕ ВНИМАНИЕ:
                // 1. Этот receiver живет только пока Activity alive
                // 2. Имеет доступ к UI-элементам Activity
                updateNetworkStatusUI()
            }
        }
        
        // Регистрация при создании Activity
        registerReceiver(networkReceiver, 
            IntentFilter(ConnectivityManager.CONNECTIVITY_ACTION))
    }
    
    override fun onDestroy() {
        // ОБЯЗАТЕЛЬНО отменить регистрацию!
        unregisterReceiver(networkReceiver)
        super.onDestroy()
    }
}
```

**Проблема:** Утечка памяти при неправильной отмене регистрации:
```kotlin
// ❌ ОПАСНО: Receiver удерживает ссылку на Activity
val receiver = object : BroadcastReceiver() {
    override fun onReceive(context: Context, intent: Intent) {
        // Неявно держит ссылку на внешний класс (Activity)
        this@MainActivity.someMethod() 
    }
}

// После destroy Activity receiver продолжает существовать в системе
// → Memory leak + возможные crashes при обращении к destroyed Activity
```

### **Manifest-registered (статический):**
```xml
<receiver 
    android:name=".BootCompletedReceiver"
    android:exported="true"
    android:enabled="true">
    <intent-filter>
        <action android:name="android.intent.action.BOOT_COMPLETED"/>
    </intent-filter>
</receiver>
```

```kotlin
class BootCompletedReceiver : BroadcastReceiver() {
    override fun onReceive(context: Context, intent: Intent) {
        // Ключевые отличия:
        // 1. Receiver создается системой при каждом событии
        // 2. Живет только во время выполнения onReceive()
        // 3. НЕТ доступа к UI компонентам приложения
        // 4. Контекст - это ApplicationContext, не Activity
        
        // ✅ Нет утечек памяти - система уничтожает после onReceive()
        // ❌ Нет доступа к текущему состоянию UI
    }
}
```

---

## **2. Производительность и системные overheads**

### **Activity-registered:**
```kotlin
// Benchmark: Регистрация/отмена 1000 receivers
fun benchmarkActivityRegistration() {
    val times = mutableListOf<Long>()
    
    repeat(1000) {
        val start = System.nanoTime()
        
        val receiver = BroadcastReceiver { _, _ -> }
        val filter = IntentFilter("TEST_ACTION")
        
        // Быстрая регистрация в памяти процесса
        registerReceiver(receiver, filter)
        
        // Быстрая отмена
        unregisterReceiver(receiver)
        
        times.add(System.nanoTime() - start)
    }
    
    val avg = times.average() / 1_000_000.0 // ~0.2-0.5ms
    Log.d("Benchmark", "Activity registration avg: ${avg}ms")
}
```

**Преимущества:**
- Нет IPC вызовов (внутрипроцессная коммуникация)
- Нет сериализации/десериализации компонентов
- Кэширование фильтров в памяти приложения

### **Manifest-registered:**
```kotlin
// Системный процесс при получении broadcast'а:
// 1. System Server получает broadcast
// 2. Проверяет манифесты всех приложений (PackageManager)
// 3. Для каждого matching receiver:
//    - Создает новый процесс приложения (если не запущен)
//    - Инстанциирует класс receiver'а
//    - Вызывает onReceive() через Binder IPC
//    - Уничтожает receiver

// Overhead: ~5-50ms в зависимости от системы
```

**Системные затраты:**
```kotlin
// Пример IPC вызова
IBinder receiverBinder = ServiceManager.getService("activity");
IActivityManager am = IActivityManager.Stub.asInterface(receiverBinder);

// Для каждого manifest receiver:
// 1. Binder transaction с системным процессом
// 2. Загрузка класса через ClassLoader
// 3. Security checks и permission validation
// 4. Process lifecycle management
```

---

## **3. Безопасность и permission модель**

### **Activity-registered:**
```kotlin
// Локальная регистрация - безопаснее
class SecureActivity : AppCompatActivity() {
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate()
        
        // Receiver виден только внутри процесса
        val receiver = object : BroadcastReceiver() {
            override fun onReceive(context: Context, intent: Intent) {
                // Внутренний event - не требует защиты
                handleInternalEvent(intent)
            }
        }
        
        // Можно регистрировать без permissions
        registerReceiver(receiver, IntentFilter("INTERNAL_EVENT"))
        
        // Для системных events нужны permissions
        if (checkSelfPermission(Manifest.permission.READ_PHONE_STATE) 
            == PackageManager.PERMISSION_GRANTED) {
            registerReceiver(phoneReceiver, 
                IntentFilter(TelephonyManager.ACTION_PHONE_STATE_CHANGED))
        }
    }
}
```

**Security Model:**
- Receiver существует только в контексте Activity
- Невидим для других приложений
- Умирает вместе с процессом приложения
- Меньше attack surface

### **Manifest-registered:**
```xml
<!-- Глобальная видимость - требует защиты -->
<receiver 
    android:name=".IncomingCallReceiver"
    android:exported="true"
    android:permission="android.permission.READ_PHONE_STATE">
    
    <intent-filter>
        <action android:name="android.intent.action.PHONE_STATE"/>
    </intent-filter>
</receiver>
```

**Уязвимости:**
```kotlin
// Атака: Broadcast Injection
class MaliciousApp {
    fun attack() {
        // Отправляем поддельный broadcast
        val intent = Intent("android.intent.action.PHONE_STATE").apply {
            putExtra("state", "RINGING")
            putExtra("incoming_number", "+1234567890")
        }
        
        // Если receiver не защищен permission...
        context.sendBroadcast(intent)
    }
}

// Защита:
// 1. android:exported="false" для internal receivers
// 2. Проверка caller identity в onReceive()
// 3. Signature-level permissions
```

---

## **4. Современные ограничения (Android 8.0+)**

### **Activity-registered (все еще работает):**
```kotlin
class ModernActivity : AppCompatActivity() {
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate()
        
        // ✅ Динамическая регистрация работает для всех broadcast'ов
        // Даже тех, что запрещены в манифесте на Android 8.0+
        
        val receiver = object : BroadcastReceiver() {
            override fun onReceive(context: Context, intent: Intent) {
                // Обрабатываем системные события
            }
        }
        
        // Эти действия НЕЛЬЗЯ регистрировать в манифесте на API 26+,
        // но МОЖНО динамически:
        val filter = IntentFilter().apply {
            addAction(ConnectivityManager.CONNECTIVITY_ACTION)
            addAction(Intent.ACTION_BATTERY_CHANGED)
            addAction(Intent.ACTION_AIRPLANE_MODE_CHANGED)
        }
        
        registerReceiver(receiver, filter)
    }
}
```

### **Manifest-registered (ограничения):**
```xml
<!-- Android 8.0+ Implicit Broadcast Restrictions -->
<receiver android:name=".NetworkReceiver">
    <intent-filter>
        <!-- ❌ НЕ РАБОТАЕТ на API 26+ -->
        <action android:name="android.net.conn.CONNECTIVITY_CHANGE"/>
        <action android:name="android.intent.action.BATTERY_CHANGED"/>
        
        <!-- ✅ РАБОТАЕТ (белый список) -->
        <action android:name="android.intent.action.BOOT_COMPLETED"/>
        <action android:name="android.intent.action.LOCALE_CHANGED"/>
        <action android:name="android.intent.action.TIMEZONE_CHANGED"/>
    </intent-filter>
</receiver>
```

**Исключения (белый список):**
```kotlin
// Полный список ~100 исключений
val allowedImplicitBroadcasts = listOf(
    Intent.ACTION_BOOT_COMPLETED,
    Intent.ACTION_LOCKED_BOOT_COMPLETED,
    Intent.ACTION_LOCALE_CHANGED,
    Intent.ACTION_TIMEZONE_CHANGED,
    Intent.ACTION_TIME_SET,
    Intent.ACTION_DATE_CHANGED,
    Intent.ACTION_USER_INITIALIZE,
    Intent.ACTION_USER_UNLOCKED,
    Intent.ACTION_PACKAGE_DATA_CLEARED,
    Intent.ACTION_PACKAGE_ADDED,
    Intent.ACTION_PACKAGE_REMOVED,
    Intent.ACTION_PACKAGE_REPLACED,
    Intent.ACTION_PACKAGE_FULLY_REMOVED,
    // ... и другие
)
```

---

## **5. Архитектурные паттерны и best practices**

### **Паттерн 1: ViewModel + LiveData вместо Activity receivers**
```kotlin
// Вместо регистрации receiver в Activity:
class NetworkViewModel : ViewModel() {
    
    private val _networkStatus = MutableLiveData<NetworkStatus>()
    val networkStatus: LiveData<NetworkStatus> = _networkStatus
    
    private val receiver = object : BroadcastReceiver() {
        override fun onReceive(context: Context, intent: Intent) {
            val status = getNetworkStatus(context)
            _networkStatus.postValue(status)
        }
    }
    
    fun register(context: Context) {
        val filter = IntentFilter(ConnectivityManager.CONNECTIVITY_ACTION)
        context.registerReceiver(receiver, filter)
    }
    
    fun unregister(context: Context) {
        context.unregisterReceiver(receiver)
    }
}

// Activity только наблюдает за LiveData
class MainActivity : AppCompatActivity() {
    
    private val viewModel: NetworkViewModel by viewModels()
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate()
        
        viewModel.networkStatus.observe(this) { status ->
            updateUI(status)
        }
    }
    
    override fun onStart() {
        super.onStart()
        viewModel.register(this)
    }
    
    override fun onStop() {
        viewModel.unregister(this)
        super.onStop()
    }
}
```

### **Паттерн 2: Service-based receivers для долгоживущих событий**
```kotlin
// Для событий, которые нужны вне зависимости от UI
class BackgroundEventService : Service() {
    
    private val screenReceiver = object : BroadcastReceiver() {
        override fun onReceive(context: Context, intent: Intent) {
            when (intent.action) {
                Intent.ACTION_SCREEN_ON -> onScreenOn()
                Intent.ACTION_SCREEN_OFF -> onScreenOff()
                Intent.ACTION_USER_PRESENT -> onUserUnlocked()
            }
        }
    }
    
    override fun onCreate() {
        super.onCreate()
        
        // Foreground service для Android 8.0+
        startForeground(NOTIFICATION_ID, createNotification())
        
        // Регистрируем долгоживущие receivers
        val filter = IntentFilter().apply {
            addAction(Intent.ACTION_SCREEN_ON)
            addAction(Intent.ACTION_SCREEN_OFF)
            addAction(Intent.ACTION_USER_PRESENT)
        }
        
        registerReceiver(screenReceiver, filter)
    }
    
    override fun onDestroy() {
        unregisterReceiver(screenReceiver)
        super.onDestroy()
    }
}
```

### **Паттерн 3: Комбинированный подход для critical events**
```kotlin
object CriticalEventManager {
    
    // Manifest receiver для гарантированного получения
    class CriticalManifestReceiver : BroadcastReceiver() {
        override fun onReceive(context: Context, intent: Intent) {
            // Гарантированно получит событие даже если приложение не запущено
            when (intent.action) {
                Intent.ACTION_BOOT_COMPLETED -> {
                    // Запускаем сервис
                    startMonitoringService(context)
                    
                    // Сохраняем событие для UI
                    EventCache.saveBootEvent()
                }
            }
        }
    }
    
    // Activity receiver для UI updates
    class CriticalActivityReceiver : BroadcastReceiver() {
        override fun onReceive(context: Context, intent: Intent) {
            // Обновляем UI если Activity активна
            (context as? MainActivity)?.updateCriticalStatus()
        }
    }
}
```

---

## **6. Производительность в production**

### **Benchmark результаты:**
```kotlin
data class ReceiverBenchmark(
    val registrationType: String,
    val registrationTimeMs: Double,
    val eventDeliveryTimeMs: Double,
    val memoryFootprintKb: Double,
    val batteryImpact: Int // 1-10
)

fun runBenchmarks(): List<ReceiverBenchmark> {
    return listOf(
        ReceiverBenchmark(
            "Activity-registered",
            0.3,  // Быстрая регистрация
            0.1,  // Быстрая доставка (внутри процесса)
            50.0, // Маленький footprint
            2     // Низкое влияние на батарею
        ),
        ReceiverBenchmark(
            "Manifest-registered (экспортированный)",
            5.0,  // Медленная регистрация (системный overhead)
            15.0, // Медленная доставка (IPC + процесс creation)
            500.0,// Большой footprint
            8     // Высокое влияние на батарею
        ),
        ReceiverBenchmark(
            "Manifest-registered (неэкспортированный)",
            3.0,  // Средняя регистрация
            8.0,  // Средняя доставка
            300.0,// Средний footprint
            5     // Среднее влияние
        )
    )
}
```

### **Оптимизации для manifest receivers:**
```xml
<!-- AndroidManifest.xml оптимизации -->
<receiver
    android:name=".OptimizedReceiver"
    android:exported="false"  <!-- Если не нужно другим приложениям -->
    android:enabled="@bool/is_receiver_enabled" <!-- Динамическое управление -->
    android:directBootAware="false" <!-- Только если нужно до unlock -->
    android:permission="custom_permission" <!-- Минимизируйте использование -->
    
    <!-- Минимизируйте intent-filter -->
    <intent-filter>
        <action android:name="android.intent.action.BOOT_COMPLETED"/>
        <!-- НЕ добавляйте лишние actions -->
    </intent-filter>
</receiver>
```

---

## **7. Отладка и мониторинг**

### **Для Activity-registered:**
```kotlin
// Debug инструменты
object ReceiverDebugger {
    
    fun debugActivityReceiver(activity: Activity, receiver: BroadcastReceiver) {
        // Проверка утечек памяти
        val weakRef = WeakReference(activity)
        val leakDetector = object : BroadcastReceiver() {
            override fun onReceive(context: Context, intent: Intent) {
                val activity = weakRef.get()
                if (activity == null || activity.isDestroyed) {
                    Log.e("LeakDetector", "Receiver удерживает destroyed Activity!")
                }
            }
        }
    }
    
    fun monitorPerformance() {
        // Мониторинг времени доставки
        val startTime = System.nanoTime()
        
        val receiver = object : BroadcastReceiver() {
            override fun onReceive(context: Context, intent: Intent) {
                val deliveryTime = (System.nanoTime() - startTime) / 1_000_000.0
                Analytics.logDeliveryTime("activity_receiver", deliveryTime)
            }
        }
    }
}
```

### **Для Manifest-registered:**
```kotlin
// System trace для manifest receivers
class TracedReceiver : BroadcastReceiver() {
    
    override fun onReceive(context: Context, intent: Intent) {
        // Начало trace
        Trace.beginSection("ManifestReceiver.onReceive")
        
        try {
            // Ваша логика
            processIntent(intent)
        } finally {
            Trace.endSection()
            
            // Логирование производительности
            val processName = getProcessName(context)
            val component = ComponentName(context, this::class.java)
            
            Log.d("ReceiverPerf", 
                "Manifest receiver ${component.shortClassName} " +
                "executed in process: $processName")
        }
    }
    
    private fun getProcessName(context: Context): String {
        val pid = android.os.Process.myPid()
        val manager = context.getSystemService(ACTIVITY_SERVICE) as ActivityManager
        return manager.runningAppProcesses
            ?.find { it.pid == pid }
            ?.processName ?: "unknown"
    }
}
```

---

## **8. Migration Guide (старый код → современный)**

### **Legacy code (до Android 8.0):**
```xml
<!-- Старый манифест -->
<receiver android:name=".NetworkReceiver">
    <intent-filter>
        <action android:name="android.net.conn.CONNECTIVITY_CHANGE"/>
        <action android:name="android.intent.action.BATTERY_LOW"/>
        <action android:name="android.intent.action.USER_PRESENT"/>
    </intent-filter>
</receiver>
```

### **Modern migration:**
```kotlin
// Шаг 1: Критические события (BOOT_COMPLETED) оставляем в манифесте
// Шаг 2: Остальные переносим в Activity/Service

class ModernNetworkHandler {
    
    companion object {
        // Shared preferences для состояния
        private const val PREFS_NAME = "network_state"
    }
    
    // Activity-based для UI updates
    class ActivityNetworkReceiver(
        private val callback: (NetworkStatus) -> Unit
    ) : BroadcastReceiver() {
        override fun onReceive(context: Context, intent: Intent) {
            val status = NetworkMonitor.getCurrentStatus(context)
            callback(status)
        }
    }
    
    // Service-based для фоновой работы
    class BackgroundNetworkReceiver : BroadcastReceiver() {
        override fun onReceive(context: Context, intent: Intent) {
            // Сохраняем состояние для работы вне UI
            val status = NetworkMonitor.getCurrentStatus(context)
            saveNetworkState(context, status)
            
            // Запускаем фоновые задачи если нужно
            if (status.isConnected) {
                scheduleBackgroundSync(context)
            }
        }
    }
    
    // Manifest-based для гарантированных событий
    class BootReceiver : BroadcastReceiver() {
        override fun onReceive(context: Context, intent: Intent) {
            // Только BOOT_COMPLETED
            if (intent.action == Intent.ACTION_BOOT_COMPLETED) {
                // Запускаем service который зарегистрирует остальные receivers
                NetworkMonitorService.start(context)
            }
        }
    }
}
```

---

## **Итог для Senior разработчика:**

### **Когда использовать Activity-registered:**
1. **UI-реактивные события** (изменение сети для обновления интерфейса)
2. **Временные подписки** (только когда экран активен)
3. **Высокочастотные события** (где производительность критична)
4. **Внутренние события приложения** (не нужно экспортировать)

### **Когда использовать Manifest-registered:**
1. **Критические системные события** (BOOT_COMPLETED, LOCALE_CHANGED)
2. **События когда приложение не запущено** (входящие вызовы, SMS)
3. **Межпроцессное взаимодействие** (когда нужно получать от других apps)
4. **Гарантированная доставка** (даже если процесс убит)

### **Критические trade-offs:**
1. **Performance vs Reliability**: Activity быстрее, Manifest надежнее
2. **Memory vs Battery**: Activity легче, Manifest может разбудить процесс
3. **Security vs Flexibility**: Activity безопаснее, Manifest более гибкий
4. **Complexity vs Simplicity**: Activity проще управлять, Manifest проще объявить

### **Рекомендации для production:**
1. **Минимизируйте manifest receivers** - только для критических событий
2. **Используйте ViewModel для UI событий** - избегайте регистрации в Activity
3. **Для фоновых событий используйте WorkManager** вместо long-running receivers
4. **Всегда защищайте exported receivers** permissions и проверками
5. **Мониторьте производительность** обоих подходов в реальных условиях

**Архитектурное правило:** Manifest receivers для "что", Activity receivers для "когда и как".

  </details>

  </details>
  
</details>

-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
**Ранее**

- []()
- 
**Далее**
- []()

