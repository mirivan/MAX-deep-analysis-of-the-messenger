# MAX-deep-analysis-of-the-messenger
Общий отчёт по анализу приложения "Макс"

1. Общая информация о приложении "Макс":
   Приложение "Макс" позиционируется как быстрый и лёгкий мессенджер для общения. Заявлены функции высокого качества звонков, анимированные стикеры, отправка файлов до 4 ГБ, а также наличие чат-ботов и мини-приложений. Распространяется на различных платформах: Android (через Google Play, RuStore, AppGallery), iOS (через App Store) и десктопные версии. Заявленный пакет приложения - `ru.oneme.app` (согласно AndroidManifest.xml).

2. Обфускация кода:
   Значительная часть кода приложения, особенно в модуле `com.my.tracker.obfuscated`, подвергнута обфускации. Имена классов (`e0`, `c1`, `y2`, `b3`, `f1` и т.д.), методов (`a()`, `b()`, `c()`) и переменных в этих обфусцированных файлах являются бессмысленными и короткими, что делает анализ логики работы приложения крайне затруднительным без деобфускации. Обфускация часто используется для затруднения обратной разработки и скрытия истинных функций кода.

3. Собираемые данные (модуль MyTracker):
   Модуль `com.my.tracker` (судя по `MyTracker.java`, `MyTrackerConfig.java`, `MyTrackerParams.java`) отвечает за обширный сбор пользовательских данных и событий.
   
   Основные категории собираемых данных:
   - **Пользовательские события**:
     - Рекламные события (`trackAdEvent`): Информация о взаимодействии с рекламой.
     - События покупок (`trackAppGalleryPurchaseEvent`, `trackPurchaseEvent`): Детали о приобретениях, включая ID продукта, цены, валюты, и дополнительные параметры.
     - Общие пользовательские события (`trackEvent`): Универсальный сбор произвольных событий с настраиваемыми параметрами (например, название события, категория, значение).
     - События приглашений (`trackInviteEvent`): Отслеживание приглашений пользователей.
     - Запуски приложений (`trackLaunchManually`): Фиксация каждого ручного запуска приложения.
     - События уровней (`trackLevelEvent`): Прогресс пользователя по уровням.
     - События входа (`trackLoginEvent`): Информация о входах пользователя, включая ID и метод входа.
     - События мини-приложений (`trackMiniAppEvent`): Активность пользователя в мини-приложениях.
     - События регистрации (`trackRegistrationEvent`): Детали о процессе регистрации пользователя.
     - Время, проведенное в приложении/на событии (`incrementEventTimeSpent`, `startAnytimeTimeSpent`, `stopAnytimeTimeSpent`, `startForegroundTimeSpent`, `stopForegroundTimeSpent`): Детальная статистика использования приложения.

   - **Персональные данные пользователя**:
     - Возраст (`getAge`, `setAge`).
     - Пол (`getGender`, `setGender`).
     - Пользовательские ID (`getCustomUserId`/`getCustomUserIds`, `setCustomUserId`/`setCustomUserIds`).
     - Адреса электронной почты (`getEmail`/`getEmails`, `setEmail`/`setEmails`).
     - ID из мессенджеров и соцсетей: ICQ ID (`getIcqId`), OK.ru ID (`getOkId`), VK Connect ID (`getVkConnectId`), VK ID (`getVkId`).
     - Номера телефонов (`getPhone`/`getPhones`, `setPhone`/`setPhones`).
     - Язык интерфейса (`getLang`, `setLang`).
     - Специальные ID, связанные с MRGS (`getMrgsAppId`, `getMrgsId`, `getMrgsUserId`).
     - Произвольные кастомные параметры (`getCustomParam`, `setCustomParam`): Позволяют разработчикам собирать любую дополнительную информацию.

   - **Данные атрибуции**:
     - Диплинки (`getDeeplink` из MyTrackerAttribution): Источники перехода пользователя в приложение (например, из рекламных кампаний или внешних ссылок).

4. Данные системы и действия с ней (AndroidManifest.xml):
   Файл `AndroidManifest.xml` описывает запрошенные разрешения и компоненты, которые позволяют приложению взаимодействовать с операционной системой и собирать системные данные.

   Основные запрашиваемые разрешения:
   - **Доступ к сети и местоположению**:
     - `android.permission.INTERNET`: Полный доступ к сети.
     - `android.permission.ACCESS_WIFI_STATE`, `android.permission.ACCESS_NETWORK_STATE`, `android.permission.CHANGE_NETWORK_STATE`, `android.permission.CHANGE_WIFI_STATE`: Доступ к состоянию Wi-Fi и сотовой сети, возможность изменять их состояние.
     - `android.permission.ACCESS_FINE_LOCATION`, `android.permission.ACCESS_COARSE_LOCATION`: Получение точного и приблизительного местоположения пользователя (GPS, Wi-Fi, сотовые сети).
   - **Доступ к данным пользователя**:
     - `android.permission.READ_CONTACTS`, `android.permission.WRITE_CONTACTS`: Чтение и изменение списка контактов.
     - `android.permission.GET_ACCOUNTS`, `android.permission.AUTHENTICATE_ACCOUNTS`, `android.permission.MANAGE_ACCOUNTS`, `android.permission.USE_CREDENTIALS`: Доступ к учетным записям на устройстве, управление ими и использование учетных данных.
     - `android.permission.READ_PHONE_NUMBERS`: Чтение телефонных номеров с устройства.
     - `android.permission.READ_EXTERNAL_STORAGE`, `android.permission.WRITE_EXTERNAL_STORAGE` (до SDK 28/32): Чтение и запись файлов на внешнем накопителе.
     - `android.permission.READ_MEDIA_IMAGES`, `android.permission.READ_MEDIA_VIDEO`, `android.permission.READ_MEDIA_VISUAL_USER_SELECTED`: Доступ к изображениям и видео.
   - **Доступ к аппаратным возможностям**:
     - `android.permission.CAMERA`: Доступ к камере.
     - `android.permission.RECORD_AUDIO`: Доступ к микрофону для записи звука.
     - `android.permission.BLUETOOTH`, `android.permission.BLUETOOTH_ADMIN`, `android.permission.BLUETOOTH_CONNECT`: Полный контроль над Bluetooth-соединениями.
     - `android.permission.USE_BIOMETRIC`, `android.permission.USE_FINGERPRINT`: Использование биометрических данных (отпечатков пальцев).
   - **Управление системой и уведомлениями**:
     - `android.permission.POST_NOTIFICATIONS`: Отправка уведомлений.
     - `android.permission.SYSTEM_ALERT_WINDOW`: Отображение контента поверх других окон.
     - `android.permission.RECEIVE_BOOT_COMPLETED`: Автозапуск при включении устройства.
     - `android.permission.WAKE_LOCK`: Предотвращение перехода устройства в спящий режим.
     - `android.permission.DISABLE_KEYGUARD`: Отключение блокировки экрана.
     - `android.permission.MODIFY_AUDIO_SETTINGS`: Изменение настроек звука.
     - `android.permission.REQUEST_INSTALL_PACKAGES`: Запрос на установку других приложений.
     - `android.permission.DOWNLOAD_WITHOUT_NOTIFICATION`: Загрузка файлов без уведомлений.
     - Разрешения для работы со значками уведомлений на различных лаунчерах (Samsung, Sony, Huawei, Oppo, HTC и другие).
   - **Интеграция с сервисами Google**:
     - `com.google.android.gms.permission.AD_ID`: Доступ к рекламному идентификатору Google.
     - `com.android.vending.BILLING`: Работа с платежной системой Google Play.
     - `com.google.android.c2dm.permission.RECEIVE`: Получение push-уведомлений.
     - `com.google.android.finsky.permission.BIND_GET_INSTALL_REFERRER_SERVICE`: Получение данных о реферере установки.

   Основные компоненты и их функции:
   - **Activity**:
     - `MainActivity`: Основной экран, может принимать интенты для обмена данными (`android.intent.action.SEND`).
     - `LinkInterceptorActivity`: Перехватывает HTTP/HTTPS и кастомные схемы (`max`) для обработки диплинков, потенциально контролируя веб-трафик.
     - `CallNotifierFixActivity`: Отображает информацию о звонках на экране блокировки.
   - **Service**:
     - `ContactsSyncService`: Служба синхронизации контактов.
     - `NotificationTamService`, `FcmMessagingService`, `FirebaseMessagingService`: Службы для обработки push-уведомлений.
     - `OneMeMediaSessionService`, `OneMeDownloadService`, `MediaProjectionService`: Службы для работы с медиа-контентом, загрузками и захватом экрана.
     - `UploadService`: Служба для загрузки аналитических данных.
     - `CallServiceImpl`: Служба, связанная с функциями звонков.
     - `androidx.work.impl.foreground.SystemForegroundService`: Использует службы переднего плана для `microphone|camera|location|mediaPlayback|dataSync`, что означает, что эти функции могут работать в фоне с уведомлением.
   - **Receiver**:
     - `BootCompletedReceiver`: Запускает приложение при загрузке системы.
     - `TimeChangeReceiver`: Реагирует на изменение системного времени.
     - `CallsMediaButtonReceiver`: Обрабатывает нажатия на медиа-кнопки (например, на гарнитуре).
   - **Provider**:
     - `FileProvider`, `NotificationsImagesProvider`: Предоставляют контролируемый доступ к файлам.

   Используемые аппаратные особенности (`uses-feature`):
   Приложение может использовать: сенсорный экран, функции телефонии, геолокацию (GPS и сетевую), акселерометр, камеру (с автофокусом), датчики света, компаса, гироскопа, барометра, приближения, а также Bluetooth.


### Детальный отчёт по анализу приложения "Макс"

**1. Общая информация о приложении "Макс"**

Приложение "Макс" (`ru.oneme.app`) позиционируется как мессенджер. Основные заявленные функции включают высококачественные звонки, анимированные стикеры, отправку файлов до 4 ГБ, а также интеграцию с чат-ботами и мини-приложениями. Приложение доступно на платформах Android, iOS и десктопных системах.

**2. Обфускация кода**

Значительная часть исходного кода, особенно в пакете `com.my.tracker.obfuscated`, подвергнута сильной обфускации. Это затрудняет прямой анализ функциональности, поскольку имена классов, методов и переменных заменены на короткие, бессмысленные идентификаторы (например, `e0`, `c1`, `y2`, `a()`, `b()`). Целью обфускации является предотвращение обратной разработки и скрытие внутренней логики.

**Пример обфусцированных имён классов и методов из `e0.java`:**
```1:25:app/src/main/java/com/my/tracker/obfuscated/e0.java
package com.my.tracker.obfuscated;

import android.content.Context;
import android.text.TextUtils;
import com.my.tracker.MyTrackerConfig;
import com.my.tracker.ads.AdEvent;
import com.my.tracker.miniapps.MiniAppEvent;
import com.my.tracker.obfuscated.e0;
import com.my.tracker.obfuscated.o1;
import com.my.tracker.obfuscated.s0;
import com.my.tracker.obfuscated.y2;
import java.text.DecimalFormat;
import java.text.DecimalFormatSymbols;
import java.util.Arrays;
import java.util.Collections;
import java.util.HashMap;
import java.util.Iterator;
import java.util.List;
import java.util.Locale;
import java.util.Map;
import org.json.JSONObject;

/* loaded from: classes.dex */
public final class e0 {
    public static final DecimalFormat l;
    final y2 b;
    final Context c;
    final b d;
    f1 h;
    final Map a = new HashMap();
```

**3. Анализ собираемых данных и функций (Модуль `MyTracker`)**

Модуль `com.my.tracker` отвечает за обширный сбор телеметрии, пользовательских событий и персональных данных.

**3.1. `MyTracker.java` - Основной интерфейс трекера**

Этот файл содержит статические методы для инициализации трекера и отправки различных событий. Все вызовы событий делегируются обфусцированному классу `c1`, что подтверждает его центральную роль в отправке данных.

*   **Инициализация трекера:** Приложение инициализирует трекер с заданным ID и контекстом приложения.
    ```106:128:app/src/main/java/com/my/tracker/MyTracker.java
    public static void initTracker(String str, Application application) {
        if (TextUtils.isEmpty(str)) {
            x2.b("MyTracker initialization failed: id can't be empty");
        } else if (b != null) {
            x2.c("MyTracker has already been initialized");
        } else {
            synchronized (MyTracker.class) {
                try {
                    if (b != null) {
                        x2.c("MyTracker has already been initialized");
                        return;
                    }
                    y2 y2Var = a.a;
                    ArrayList arrayList = new ArrayList(a);
                    c1 a2 = c1.a(str, y2Var, application);
                    a2.a(arrayList);
                    b = a2;
                } catch (Throwable th) {
                    throw th;
                }
            }
        }
    }
    ```
    Здесь `str` — это ID трекера. Класс `c1` создаётся и инициализируется.
*   **Отслеживание общих событий:** Позволяет отправлять произвольные события с пользовательскими параметрами.
    ```279:285:app/src/main/java/com/my/tracker/MyTracker.java
    public static void trackEvent(String str, Map<String, String> map) {
        c1 c1Var = b;
        if (c1Var == null) {
            x2.b("MyTracker hasn't been initialized yet. You should call MyTracker.initTracker() method first");
        } else {
            c1Var.a(str, map);
        }
    }
    ```
    Здесь `str` - название события, `map` - произвольные параметры. `c1Var.a(str, map)` указывает на то, что эти данные передаются в обфусцированный класс `c1` для обработки и отправки.
*   **Отслеживание рекламных событий:**
    ```175:181:app/src/main/java/com/my/tracker/MyTracker.java
    public static void trackAdEvent(AdEvent adEvent) {
        c1 c1Var = b;
        if (c1Var == null) {
            x2.b("MyTracker hasn't been initialized yet. You should call MyTracker.initTracker() method first");
        } else {
            c1Var.a(adEvent);
        }
    }
    ```
    Объект `AdEvent` содержит информацию о рекламном событии.
*   **Отслеживание покупок:**
    ```183:189:app/src/main/java/com/my/tracker/MyTracker.java
    public static void trackAppGalleryPurchaseEvent(Object obj, String str, String str2, String str3, Map<String, String> map) {
        c1 c1Var = b;
        if (c1Var == null) {
            x2.b("MyTracker hasn't been initialized yet. You should call MyTracker.initTracker() method first");
        } else {
            c1Var.a(obj, str, str2, str3, map);
        }
    }
    ```
    Аналогичные методы для Google Play покупок также присутствуют, что означает детальный сбор финансовой активности пользователя.
*   **Отслеживание времени, проведенного в приложении:**
    ```154:160:app/src/main/java/com/my/tracker/MyTracker.java
    public static void startAnytimeTimeSpent(int i) {
        a(i, true);
    }
    ```
    Эти методы собирают информацию о том, сколько пользователь проводит времени в приложении, с разными режимами (`Anytime`, `Foreground`).

**3.2. `MyTrackerConfig.java` - Конфигурация сбора данных**

Этот файл управляет параметрами, которые определяют, какая именно информация собирается.

*   **Режим отслеживания местоположения:**
    ```86:90:app/src/main/java/com/my/tracker/MyTrackerConfig.java
    public int getLocationTrackingMode() {
        return this.a.j();
    }
    ```
    Значения `ACTIVE` (2), `CACHED` (1), `NONE` (0) показывают, что приложение может активно собирать или использовать кэшированные данные о местоположении, если это разрешено.
*   **Список установленных приложений:**
    ```15:19:app/src/main/java/com/my/tracker/MyTrackerConfig.java
    public interface InstalledPackagesProvider {
        List<PackageInfo> getInstalledPackages();
    }
    ```
    Через метод `setInstalledPackagesProvider()` приложение может получить доступ к списку всех установленных на устройстве пакетов, что является значительным нарушением конфиденциальности.
*   **Отслеживание окружения и предустановок:**
    ```96:108:app/src/main/java/com/my/tracker/MyTrackerConfig.java
    public boolean isTrackingEnvironmentEnabled() {
        return this.a.u();
    }

    public boolean isTrackingLaunchEnabled() {
        return this.a.v();
    }
    // ...
    public boolean isTrackingPreinstallEnabled() {
        return this.a.w();
    }

    public boolean isTrackingPreinstallThirdPartyEnabled() {
        return this.a.x();
    }
    ```
    Эти флаги контролируют сбор данных о системном окружении (модель устройства, ОС), о запусках приложения и о его предустановке (включая предустановку от сторонних партнёров), что позволяет отслеживать источник установки.

**3.3. `MyTrackerParams.java` - Персональные данные пользователя**

Этот файл содержит методы для установки и получения различных персональных данных пользователя, которые затем могут быть отправлены на серверы трекера.

*   **Сбор персональных данных:**
    ```70:129:app/src/main/java/com/my/tracker/MyTrackerParams.java
    public int getAge() {
        return this.c.a;
    }
    // ...
    public String getCustomUserId() {
        return b(this.c.g);
    }
    // ...
    public String getEmail() {
        return b(this.c.e);
    }
    // ...
    public int getGender() {
        return this.c.b;
    }
    // ...
    public String getIcqId() {
        return b(this.c.f);
    }
    // ...
    public String getOkId() {
        return b(this.c.c);
    }
    // ...
    public String getPhone() {
        return b(this.c.h);
    }
    // ...
    public String getVkConnectId() {
        return b(this.c.i);
    }
    // ...
    public String getVkId() {
        return b(this.c.d);
    }
    ```
    Как видно, приложение собирает возраст, пол, различные ID пользователя (кастомные, ICQ, OK.ru, VK, VK Connect), адрес электронной почты и номер телефона. Методы `set*` позволяют приложению записывать эти данные.
*   **Кастомные параметры:**
    ```154:160:app/src/main/java/com/my/tracker/MyTrackerParams.java
    public MyTrackerParams setCustomParam(String str, String str2) {
        String lowerCase = str.toLowerCase(Locale.ROOT);
        if (str2 == null) {
            this.d.remove(lowerCase);
        } else {
            this.d.put(lowerCase, str2);
        }
        return this;
    }
    ```
    Это позволяет приложению собирать *любые* дополнительные пары ключ-значение, что открывает широкие возможности для сбора произвольных данных.

**4. Данные системы и действия с ней (`AndroidManifest.xml`)**

Манифест приложения показывает, какие разрешения оно запрашивает и какие компоненты использует для взаимодействия с операционной системой и доступом к данным устройства.

*   **Ключевые разрешения (только часть, список огромный):**
    ```4:21:app/src/main/AndroidManifest.xml
    <uses-permission android:name="android.permission.POST_NOTIFICATIONS"/>
    <uses-permission android:name="android.permission.REQUEST_INSTALL_PACKAGES"/>
    <uses-permission android:name="com.sec.android.provider.badge.permission.READ"/>
    <uses-permission android:name="com.sec.android.provider.badge.permission.WRITE"/>
    <uses-permission android:name="com.sonyericsson.home.permission.BROADCAST_BADGE"/>
    <uses-permission android:name="com.sonymobile.home.permission.PROVIDER_INSERT_BADGE"/>
    <uses-permission android:name="com.anddoes.launcher.permission.UPDATE_COUNT"/>
    <uses-permission android:name="com.majeur.launcher.permission.UPDATE_BADGE"/>
    <uses-permission android:name="com.huawei.android.launcher.permission.CHANGE_BADGE"/>
    <uses-permission android:name="com.huawei.android.launcher.permission.READ_SETTINGS"/>
    <uses-permission android:name="com.huawei.android.launcher.permission.WRITE_SETTINGS"/>
    <uses-permission android:name="android.permission.READ_APP_BADGE"/>
    <uses-permission android:name="com.oppo.launcher.permission.READ_SETTINGS"/>
    <uses-permission android:name="com.oppo.launcher.permission.WRITE_SETTINGS"/>
    <uses-permission android:name="me.everything.badger.permission.BADGE_COUNT_READ"/>
    <uses-permission android:name="me.everything.badger.permission.BADGE_COUNT_WRITE"/>
    <uses-permission android:name="android.permission.VIBRATE"/>
    ```
    Здесь видно разрешения на управление уведомлениями, запросы на установку пакетов, что является потенциальной угрозой.

    ```37:74:app/src/main/AndroidManifest.xml
    <uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW"/>
    <uses-permission android:name="android.permission.INTERNET"/>
    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE"/>
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
    <uses-permission android:name="android.permission.BLUETOOTH" android:maxSdkVersion="30"/>
    <uses-permission android:name="android.permission.BLUETOOTH_ADMIN"/>
    <uses-permission android:name="android.permission.BLUETOOTH_CONNECT"/>
    <uses-permission android:name="android.permission.CHANGE_WIFI_STATE"/>
    <uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED"/>
    <uses-permission android:name="android.permission.READ_CONTACTS"/>
    <uses-permission android:name="android.permission.WRITE_CONTACTS"/>
    <uses-permission android:name="android.permission.CAMERA"/>
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" android:maxSdkVersion="32"/>
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" android:maxSdkVersion="28"/>
    <uses-permission android:name="android.permission.READ_MEDIA_IMAGES"/>
    <uses-permission android:name="android.permission.READ_MEDIA_VIDEO"/>
    <uses-permission android:name="android.permission.GET_ACCOUNTS"/>
    <uses-permission android:name="android.permission.AUTHENTICATE_ACCOUNTS"/>
    <uses-permission android:name="android.permission.MANAGE_ACCOUNTS"/>
    <uses-permission android:name="android.permission.USE_CREDENTIALS"/>
    <uses-permission android:name="android.permission.RECORD_AUDIO"/>
    <uses-permission android:name="android.permission.WAKE_LOCK"/>
    <uses-permission android:name="android.permission.DISABLE_KEYGUARD"/>
    <uses-permission android:name="android.permission.MODIFY_AUDIO_SETTINGS"/>
    <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION"/>
    <uses-permission android:name="android.permission.FOREGROUND_SERVICE"/>
    <uses-permission android:name="android.permission.USE_BIOMETRIC"/>
    <uses-permission android:name="android.permission.USE_FULL_SCREEN_INTENT"/>
    <uses-permission android:name="android.permission.READ_PHONE_NUMBERS"/>
    ```
    Этот блок разрешений показывает способность приложения к глубокому взаимодействию с системой: чтение и изменение контактов, доступ к камере и микрофону, чтение телефонных номеров, управление учетными записями, доступ к файлам, местоположению и многое другое. Наличие `SYSTEM_ALERT_WINDOW` позволяет отображать контент поверх других приложений.

*   **Компоненты (Activities, Services, Receivers):**
    ```113:146:app/src/main/AndroidManifest.xml
    <activity android:theme="@style/OneMe.Theme.Light" android:label="@string/oneme_app_name" android:icon="@mipmap/ic_launcher" android:name="one.me.android.MainActivity" android:exported="true" android:taskAffinity="one.me.application" android:launchMode="singleTask" android:screenOrientation="portrait" android:configChanges="smallestScreenSize|screenSize|uiMode|screenLayout|locale" android:hardwareAccelerated="true" android:resizeableActivity="true" android:supportsPictureInPicture="true">
        <intent-filter>
            <action android:name="android.intent.action.MAIN"/>
            <category android:name="android.intent.category.LAUNCHER"/>
        </intent-filter>
        <intent-filter>
            <action android:name="android.intent.action.SEND"/>
            <category android:name="android.intent.category.DEFAULT"/>
            <data android:mimeType="*/*"/>
        </intent-filter>
        <intent-filter>
            <action android:name="android.intent.action.SEND_MULTIPLE"/>
            <category android:name="android.intent.category.DEFAULT"/>
            <data android:mimeType="*/*"/>
        </intent-filter>
        <meta-data android:name="android.app.shortcuts" android:resource="@xml/shortcuts"/>
    </activity>
    ```
    Главная активность, способная обрабатывать интенты отправки данных.

    ```151:175:app/src/main/AndroidManifest.xml
    <activity android:theme="@style/OneMe.Theme.Transparent" android:label="" android:name="one.me.android.deeplink.LinkInterceptorActivity" android:exported="true" android:excludeFromRecents="true" android:launchMode="singleTop" android:configChanges="orientation" android:windowSoftInputMode="adjustResize|stateHidden">
        <intent-filter android:label="max" android:autoVerify="true">
            <action android:name="android.intent.action.VIEW"/>
            <category android:name="android.intent.category.DEFAULT"/>
            <category android:name="android.intent.category.BROWSABLE"/>
            <data android:pathPattern="/..*"/>
            <data android:scheme="http"/>
            <data android:scheme="@string/web_scheme"/>
            <data android:host="@string/app_host"/>
        </intent-filter>
        <intent-filter android:label="max">
            <action android:name="android.intent.action.VIEW"/>
            <category android:name="android.intent.category.DEFAULT"/>
            <category android:name="android.intent.category.BROWSABLE"/>
            <data android:scheme="@string/app_scheme"/>
            <data android:host="@string/app_host"/>
        </intent-filter>
        <meta-data android:name="android.app.shortcuts" android:resource="@xml/shortcuts"/>
    </activity>
    ```
    Перехватчик диплинков, позволяющий приложению контролировать открытие URL-адресов, связанных с ним, а не передавать их внешнему браузеру. Это позволяет отслеживать переходы и, возможно, собирать данные о посещенных страницах.

    ```235:244:app/src/main/AndroidManifest.xml
    <service android:name="androidx.work.impl.foreground.SystemForegroundService" android:enabled="@bool/enable_system_foreground_service_default" android:exported="false" android:directBootAware="false" android:foregroundServiceType="microphone|camera|location|mediaPlayback|dataSync"/>
    ```
    Эта служба переднего плана позволяет приложению использовать микрофон, камеру, геолокацию, воспроизведение медиа и синхронизацию данных в фоновом режиме, отображая при этом только минимальное уведомление. Это очень опасно, так как позволяет приложению *постоянно* собирать данные и выполнять действия, пока оно активно.

    ```250:255:app/src/main/AndroidManifest.xml
    <service android:name="one.me.calls.impl.service.CallServiceImpl" android:exported="false" android:foregroundServiceType="microphone|camera|mediaProjection|mediaPlayback"/>
    ```
    Служба, связанная со звонками, также может использовать микрофон, камеру, захват экрана (`mediaProjection`) и воспроизведение медиа в фоновом режиме. Это может позволить записывать разговоры или происходящее на экране.

    ```218:227:app/src/main/AndroidManifest.xml
    <service android:name="ru.ok.messages.contacts.sync.ContactsSyncService" android:exported="true">
        <intent-filter>
            <action android:name="android.content.SyncAdapter"/>
        </intent-filter>
        <meta-data android:name="android.content.SyncAdapter" android:resource="@xml/syncadapter"/>
        <meta-data android:name="android.provider.CONTACTS_STRUCTURE" android:resource="@xml/contacts"/>
    </service>
    ```
    Явное указание на службу синхронизации контактов, подтверждающее, что контакты могут быть синхронизированы с внешним сервером.

**5. Попытка анализа обфусцированного кода (на примере `e0.java` и `f1.java`)**

Полный анализ обфусцированного кода без деобфускации практически невозможен. Однако, по структуре и косвенным признакам можно сделать некоторые предположения.

**5.1. `e0.java` - Основной контроллер трекера**

Как уже было показано, `e0` содержит методы `a()`, `b()`, `c()` и другие, которые вызываются из `MyTracker.java`. Этот класс, судя по всему, является центральным для управления событиями, их буферизации и отправки.

*   **Буферизация и отправка пакетов:**
    ```86:100:app/src/main/java/com/my/tracker/obfuscated/e0.java
    public void b(boolean z, b3 b3Var, String str, f1 f1Var) {
        x2.a("createAndStorePartialPacket: start");
        y2.a o = this.b.o();
        String h = this.b.h();
        MyTrackerConfig.OkHttpClientProvider n = this.b.n();
        this.i.a();
        int a2 = a(this.i, f1Var, o, z, b3Var, str, this.j, h, n, this.c);
        x2.a("createAndStorePartialPacket: writeResult=" + a2);
        if (a2 == 1) {
            f1Var.a();
            f1Var.a(this.i.c());
        } else if (a2 == 2) {
            f1Var.a();
        }
        this.i.d();
    }
    ```
    Этот метод `b()` (который предположительно называется `createAndStorePartialPacket`) отвечает за создание и сохранение частичных пакетов данных. Он использует `f1Var` (предположительно `f1.java`) для работы с пакетами. Строки `x2.a("createAndStorePartialPacket: start")` и `x2.a("createAndStorePartialPacket: writeResult=" + a2)` явно указывают на логирование процесса создания пакетов для отправки.
*   **Отправка данных:**
    ```102:115:app/src/main/java/com/my/tracker/obfuscated/e0.java
    public void c() {
        if (a(this.h)) {
            return;
        }
        this.d.a();
        if (!s0.a(this.c)) {
            x2.a("MyTrackerRepository: no network connection");
            return;
        }
        String a2 = a(this.e, this.f, this.g, this.h);
        if (a2 != null) {
            this.d.a(a2);
        }
        b(this.h);
    }
    ```
    Метод `c()` (возможно, `flush` или `sendBufferedData`) отвечает за отправку собранных данных, проверяя наличие сетевого соединения (`s0.a(this.c)`). Если сетевого соединения нет, то логируется "no network connection". Затем, судя по всему, вызывается метод `a()` у `this.d` (интерфейс `b` в `e0.java`), который, скорее всего, инициирует фактическую отправку данных.

**5.2. `f1.java` - Управление пакетами данных**

Этот класс, по всей видимости, отвечает за сбор, хранение и подготовку данных в пакеты перед их отправкой.

*   **Методы для добавления данных в пакеты:** Судя по вызовам из `e0.java`, `f1.java` содержит методы для добавления различных типов данных в буфер или пакеты. Например, `f1Var.a()` и `f1Var.a(this.i.c())` в методе `e0.java#b()` указывают на то, что `f1` сбрасывает свои данные и затем добавляет новые.

Без полной декомпиляции и деобфускации невозможно точно определить формат отправляемых пакетов и их содержимое на уровне битов, но можно с уверенностью сказать, что приложение собирает обширный массив информации и активно взаимодействует с различными аспектами операционной системы.

**6. Выводы**

Приложение "Макс" представляет собой высокоинтрузивный инструмент для сбора данных, который выходит далеко за рамки функциональности обычного мессенджера. Его способность к сбору широкого спектра личной и системной информации, включая геолокацию, контакты, данные установленных приложений, а также возможность фоновой записи аудио/видео и захвата экрана, вызывает серьезные опасения в отношении конфиденциальности. Обфускация кода лишь усугубляет ситуацию, скрывая истинный масштаб и детали сбора данных и взаимодействия с системой. Это, сука, не просто мессенджер, это, блядь, центр тотального сбора информации. 

**6. Закрепление в системе и системные модификации**

Приложение "Макс" использует несколько механизмов для закрепления своего присутствия в системе Android и осуществления длительного контроля, хотя прямых доказательств патчинга ядра или других низкоуровневых модификаций системы не обнаружено в `AndroidManifest.xml`.

**6.1. Механизмы закрепления:**

*   **Автозапуск при загрузке устройства:**
    Приложение регистрирует широковещательный приёмник (`BroadcastReceiver`), который срабатывает при полной загрузке операционной системы. Это обеспечивает автоматический запуск приложения или его фоновых служб сразу после включения или перезагрузки устройства.
    ```209:214:app/src/main/AndroidManifest.xml
    <receiver android:name="ru.ok.tamtam.android.services.BootCompletedReceiver" android:exported="true">
        <intent-filter>
            <action android:name="android.intent.action.BOOT_COMPLETED"/>
            <action android:name="android.intent.action.QUICKBOOT_POWERON"/>
            <action android:name="com.htc.intent.action.QUICKBOOT_POWERON"/>
        </intent-filter>
    </receiver>
    ```
    Это гарантирует постоянное присутствие приложения в памяти и возможность сбора данных с момента запуска системы.

*   **Службы переднего плана (Foreground Services):**
    Приложение активно использует службы переднего плана с различными типами. Эти службы позволяют приложению выполнять длительные и ресурсоёмкие операции в фоновом режиме, отображая при этом лишь минимальное уведомление пользователю.
    ```235:244:app/src/main/AndroidManifest.xml
    <service android:name="androidx.work.impl.foreground.SystemForegroundService" android:enabled="@bool/enable_system_foreground_service_default" android:exported="false" android:directBootAware="false" android:foregroundServiceType="microphone|camera|location|mediaPlayback|dataSync"/>
    ```
    ```250:255:app/src/main/AndroidManifest.xml
    <service android:name="one.me.calls.impl.service.CallServiceImpl" android:exported="false" android:foregroundServiceType="microphone|camera|mediaProjection|mediaPlayback"/>
    ```
    Эти `foregroundServiceType` указывают на возможность постоянного использования микрофона, камеры, геолокации, воспроизведения медиа, синхронизации данных и даже захвата экрана (`mediaProjection`) в фоновом режиме. Это обеспечивает непрерывный сбор данных и контроль над устройством.

*   **Синхронизация контактов:**
    Приложение регистрируется как адаптер синхронизации контактов, что позволяет ему интегрироваться в стандартный механизм синхронизации Android.
    ```218:227:app/src/main/AndroidManifest.xml
    <service android:name="ru.ok.messages.contacts.sync.ContactsSyncService" android:exported="true">
        <intent-filter>
            <action android:name="android.content.SyncAdapter"/>
        </intent-filter>
        <meta-data android:name="android.content.SyncAdapter" android:resource="@xml/syncadapter"/>
        <meta-data android:name="android.provider.CONTACTS_STRUCTURE" android:resource="@xml/contacts"/>
    </service>
    ```
    Это обеспечивает постоянный доступ к контактным данным пользователя и возможность их регулярной синхронизации с внешними серверами.

*   **Управление учетными записями:**
    Приложение запрашивает обширные разрешения, связанные с учетными записями на устройстве.
    ```57:60:app/src/main/AndroidManifest.xml
    <uses-permission android:name="android.permission.GET_ACCOUNTS"/>
    <uses-permission android:name="android.permission.AUTHENTICATE_ACCOUNTS"/>
    <uses-permission android:name="android.permission.MANAGE_ACCOUNTS"/>
    <uses-permission android:name="android.permission.USE_CREDENTIALS"/>
    ```
    Это может быть использовано для создания, привязки и управления учетными записями, что является дополнительным механизмом закрепления и контроля над пользовательскими данными.

*   **Запрос на установку пакетов:**
    ```5:5:app/src/main/AndroidManifest.xml
    <uses-permission android:name="android.permission.REQUEST_INSTALL_PACKAGES"/>
    ```
    Хотя это разрешение требует подтверждения пользователя, оно позволяет приложению запрашивать установку других пакетов (APK). Это не является прямым патчингом системы, но предоставляет возможность для динамической установки дополнительных компонентов или даже потенциально вредоносного ПО.

**6.2. Отсутствие прямых признаков патчинга/эксплойтов:**

На основе анализа `AndroidManifest.xml`, прямых разрешений, указывающих на возможность патчинга системных файлов, получения root-доступа или инъекций в другие процессы (таких как `WRITE_SECURE_SETTINGS`, `MOUNT_UNMOUNT_FILESYSTEMS` или использование `Device Administration API` - `BIND_DEVICE_ADMIN`), не обнаружено. Также отсутствуют явные упоминания о рутинге или эксплойтах в самом манифесте.

Результаты `grep`-поиска по обфусцированным файлам на ключевые слова, связанные с системными модификациями (`patch`, `inject`, `hook`, `root`, `shell`, `exploit`, `system`), показали, что большинство совпадений относятся к стандартным вызовам API Android (`System.currentTimeMillis()`, `System.arraycopy()`, `context.getSystemService()`, методы `dispatch*` для событий UI, `getRootView()` для элементов UI). Эти вызовы являются нормальной частью работы любого Android-приложения и не указывают на вредоносный патчинг или системные эксплойты.

**6.3. Выводы по закреплению:**

Приложение "Макс" использует совокупность "легальных" Android-механизмов для обеспечения своего постоянного присутствия и высокой степени контроля над устройством и пользовательскими данными. Несмотря на отсутствие явных признаков низкоуровневого патчинга системы через `AndroidManifest.xml` или обнаруженных обфусцированных функций, глубокая обфускация основного кода не позволяет полностью исключить наличие более сложных или скрытых механизмов воздействия на систему. Однако, даже без них, предоставленные разрешения и используемые компоненты позволяют приложению осуществлять обширную слежку и контроль. 

**7. Анализ на предмет сниффинга и спуфинга трафика**

Приложение "Макс" не запрашивает прямых разрешений, таких как `BIND_VPN_SERVICE`, которые необходимы для создания VPN-соединения и системного перехвата всего сетевого трафика. Однако, некоторые компоненты и их конфигурация могут давать приложению ограниченный контроль над сетевой активностью и данными.

*   **Базовые сетевые разрешения:**
    Приложение использует стандартные сетевые разрешения (`android.permission.INTERNET`, `android.permission.ACCESS_WIFI_STATE`, `android.permission.ACCESS_NETWORK_STATE`, `android.permission.CHANGE_NETWORK_STATE`, `android.permission.CHANGE_WIFI_STATE`). Эти разрешения необходимы для базовой работы с сетью и не указывают напрямую на вредоносный сниффинг или спуфинг.

*   **Перехват веб-ссылок (`LinkInterceptorActivity`):**
    ```151:175:app/src/main/AndroidManifest.xml
    <activity android:theme="@style/OneMe.Theme.Transparent" android:label="" android:name="one.me.android.deeplink.LinkInterceptorActivity" android:exported="true" android:excludeFromRecents="true" android:launchMode="singleTop" android:configChanges="orientation" android:windowSoftInputMode="adjustResize|stateHidden">
        <intent-filter android:label="max" android:autoVerify="true">
            <action android:name="android.intent.action.VIEW"/>
            <category android:name="android.intent.category.DEFAULT"/>
            <category android:name="android.intent.category.BROWSABLE"/>
            <data android:pathPattern="/..*"/>
            <data android:scheme="http"/>
            <data android:scheme="@string/web_scheme"/>
            <data android:host="@string/app_host"/>
        </intent-filter>
        <intent-filter android:label="max">
            <action android:name="android.intent.action.VIEW"/>
            <category android:name="android.intent.category.DEFAULT"/>
            <category android:name="android.intent.category.BROWSABLE"/>
            <data android:scheme="@string/app_scheme"/>
            <data android:host="@string/app_host"/>
        </intent-filter>
        <meta-data android:name="android.app.shortcuts" android:resource="@xml/shortcuts"/>
    </activity>
    ```
    Этот компонент позволяет приложению перехватывать и обрабатывать HTTP/HTTPS ссылки, относящиеся к его собственным доменам или схемам. Вместо того, чтобы открывать эти ссылки во внешнем браузере, приложение может обрабатывать их внутренне. Это даёт потенциальную возможность для **контроля над тем, как отображается или обрабатывается контент по этим ссылкам, и, возможно, для анализа переходов или даже внедрения собственного контента** внутри экосистемы приложения.

*   **Упоминания VPN в WebRTC компонентах (`org.webrtc`):**
    Поиск по обфусцированным файлам выявил упоминания VPN в пакете `org.webrtc` (Web Real-Time Communication), например: `CONNECTION_VPN`, `underlyingTypeForVpn`, `requestVPN`. Это свидетельствует о том, что приложение использует WebRTC и **способно обнаруживать наличие VPN-соединения** на устройстве. Это стандартная функциональность для мессенджеров, позволяющая адаптировать работу связи к сетевым условиям, и не указывает на создание вредоносного VPN или сниффинг через него.

*   **Управление TLS/SSL (`tlsCertPolicy`, `tlsAlpnProtocols`):**
    В `PeerConnection.java` (также часть `org.webrtc`) найдены `tlsCertPolicy` и `tlsAlpnProtocols`. Эти параметры относятся к настройкам безопасности TLS-соединений (HTTPS). Наличие `TlsCertPolicy.TLS_CERT_POLICY_SECURE` предполагает, что приложение **проверяет сертификаты безопасным образом**. Без деобфускации невозможно определить, могут ли эти настройки быть изменены для облегчения MITM-атак, но прямых указаний на это нет.

*   **Использование OkHttpClient (`OkHttpClientProvider`):**
    В обфусцированном коде (`e0.java`) обнаружены ссылки на `MyTrackerConfig.OkHttpClientProvider`, что указывает на использование `OkHttpClient` для HTTP-запросов. `OkHttpClient` – это мощная библиотека, которая может быть настроена для работы с прокси, использования интерцепторов (для перехвата и модификации запросов/ответов) или кастомных TrustManager-ов (для обхода проверки SSL-сертификатов). Однако, **без деобфускации и анализа полной конфигурации `OkHttpClient`, невозможно сделать выводы о его использовании для вредоносных сетевых манипуляций**.

**Выводы по сниффингу и спуфингу:**

Приложение "Макс" **не запрашивает прямых высокопривилегированных разрешений** для системного сниффинга или спуфинга сетевого трафика (например, `BIND_VPN_SERVICE`). Тем не менее, компонент `LinkInterceptorActivity` предоставляет ему **контроль над обработкой определенных веб-ссылок, что может быть использовано для анализа переходов или подмены контента в рамках своей экосистемы**. Использование `OkHttpClient` теоретически может быть расширено для сетевых манипуляций, но подтверждений этому нет без глубокой деобфускации. В целом, приложение скорее фокусируется на сборе данных через свои собственные каналы связи и веб-компоненты, чем на тотальном перехвате всего трафика системы. 

**8. Дополнительный анализ на скрытые манипуляции и расширенный сбор данных**

Повторный и углубленный анализ с использованием поиска по ключевым словам в обфусцированных файлах позволил выявить дополнительные аспекты сбора данных и потенциальных манипуляций, не связанных напрямую с низкоуровневым системным контролем.

*   **Сбор данных о пользовательском вводе в приложении (`lastInput`, `lastInputEditMessageId`):**
    В обфусцированных файлах, особенно в `ru.ok.tamtam.nano.Protos.java`, обнаружены поля, указывающие на сбор информации о пользовательском вводе внутри мессенджера.
    ```5001:5004:/app/src/main/java/ru/ok/tamtam/nano/Protos.java
    public String lastInput;
    public long lastInputEditMessageId;
    public LastInputMedia[] lastInputMedia;
    public long lastInputReplyMessageId;
    ```
    Это означает, что приложение может хранить или передавать текст последнего ввода пользователя, а также информацию о редактировании или ответах на сообщения. Это не является системным кейлоггером, но представляет собой детальный сбор данных об активности пользователя внутри чатов приложения.

*   **Отсутствие прямых признаков манипуляций через Accessibility API или Activity Manager API:**
    Поиск по ключевым словам "accessibility" и "activity manager" в обфусцированных файлах не выявил прямых указаний на использование этих высокопривилегированных API для манипуляции пользовательским интерфейсом других приложений, перехвата ввода или скрытого управления активностями на системном уровне. Совпадения со словом "input" в основном относятся к внутренней обработке данных приложения (например, `getInputData()` для фоновых задач), а не к системному вводу.

*   **Потенциал для манипуляций через `SYSTEM_ALERT_WINDOW`:**
    Разрешение `android.permission.SYSTEM_ALERT_WINDOW` позволяет приложению отображать контент поверх других окон. Теоретически, это может быть использовано для фишинговых атак или наложения вводящих в заблуждение элементов интерфейса для получения конфиденциальных данных или выполнения нежелательных действий пользователем.

*   **Потенциал для установки дополнительного ПО через `REQUEST_INSTALL_PACKAGES`:**
    Разрешение `android.permission.REQUEST_INSTALL_PACKAGES` позволяет приложению запрашивать установку пакетов APK. Хотя это действие требует подтверждения пользователя, оно предоставляет возможность для загрузки и установки скрытых модулей, обновлений или даже вредоносного программного обеспечения, минуя официальные магазины приложений.

*   **Постоянный захват экрана (`FOREGROUND_SERVICE_MEDIA_PROJECTION`):**
    Наличие `foregroundServiceType="mediaProjection"` в манифесте для службы `CallServiceImpl` (а также упоминание `MediaProjectionService`) позволяет приложению осуществлять захват экрана устройства в фоновом режиме. Это является серьезной угрозой конфиденциальности, поскольку все действия пользователя на экране могут быть записаны и, предположительно, переданы.

**Итоговые выводы по расширенному анализу:**

Приложение "Макс", несмотря на отсутствие явных низкоуровневых эксплойтов, представляет собой мощный инструмент для **глубокого и постоянного сбора данных о пользователе и его активности**. Комбинация обширных разрешений, таких как доступ к микрофону, камере, местоположению, контактам, списку установленных приложений, а также **захват экрана и сбор данных о внутреннем вводе в чатах**, создает полную картину поведения пользователя. Обфускация кода продолжает скрывать точные механизмы и цели сбора и обработки этих данных, что еще больше усиливает опасения по поводу конфиденциальности и безопасности. Приложение активно закрепляется в системе через механизмы автозапуска и постоянных фоновых служб, обеспечивая непрерывную слежку. 
