# MAX-deep-analysis-of-the-messenger
От себя (Mirivan):
   Я не явлюясь сторонником этого мессенджера, сам им я не пользуюсь по причинам банального отсутствия желания как со стороны его функционала, так и с осознанием того, что это попытка замены Телеграма им (даже если это не так, по крайней мере оно так звучит, БУМ и всё).
   В исходном анализе я лишь оставлю от себя комментарии по интересующим меня пунктам и не более.
   Мой ответ находится на стадии написания, поэтому многое я могу еще пропускать/не объяснить и т.д.

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

Mirivan: Хочу тут сразу сделать пометку на том, что MyTracker (MyTarget - target.vk.ru, tracker.my.com) - инструмент ВКонтакте для сбора телеметрии. Каким был плохим не казался этот модуль, но он действительно собирает метрику при использовании приложения с ним включенным НО, давайте откатимся на пару лет назад, когда еще ВКонтакте не был "соц.сетью для пожилых" (извините за такую, возможно жесткую аллегорию). Этот самый MyTracker был впихнут ВКонтакте как и в их мобильное приложение, так и в ОК (Одноклассники), так и в ICQ (когда он тоже перешел в руки Mail.RU), камон - никто не жаловался. Зачем сбор метрики? - Да банально чтобы вы видели ПОДХОДЯЩУЮ ДЛЯ ВАС рекламу (иной метод заработка компании). Многие против рекламы, с этим очень трудно поспорить, но как уж нам быть, раз мы зомбоящик в детстве смотрели, а там этой рекламы было ну никак не меньше (такое себе конечно сравнение, но честно скажу - у меня нет даже других слов по этому поводу).

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
	
	Mirivan: стоп, что значит произвольных? Цитата "myTracker — free mobile analytics for iOS and Android platforms. Get connected to know everything about your apps, audience and advertising campaigns" не о чем не говорит? Если тебе (автору анализа) в запару деобфусцировать mytracker, загрузи Jar'ник с Maven, там гораздо будет проще понять для чего всё это делается.
	

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
	
	Mirivan: ну кстати на счет запросов на установку пакетов хрен поспоришь, но давай представим ситацию что в приложении, конкретно в чате с кем-то тебе скинули APK файл, ты его собираешься установить, но в logcat видишь как PackageManager и MAX усрались от недостатка прав, что будешь тогда делать? Скачивать условный проводник с доступом в Android/data и лезть в доступные файлы приложения для установки (я не знаю куда этот мессенджер загружает скачиваемые с чатов файлы, но то, что потребуется установить полученный APK из другой программы, которой ты уже дал REQUEST_INSTALL_PACKAGES - это абсолютно точная логика.

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
	
	Mirivan: на счет чтения и изменения контактов - поюзай ты функционал приложения, а конкретный пинок для тебя - найду ты такую функцию как "Синхронизация контактов" и подумай блин головой, зачем тогда мессенджеру такие разрешения. На счет остальных разрешений дальше.

...

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
	
	Mirivan: ну тут ты загнул, косаемо "собирать данные о посещенных страницах". Ну учитвая что они сюда впихнули схему http (я не могу пока что предположить зачем), выглядит это странно, но в Android есть фича ВЫБОРА, какие схемы будет по умолчанию открывать приложение (MAX), учитывая, что ему дали autoVerify.

    ```235:244:app/src/main/AndroidManifest.xml
    <service android:name="androidx.work.impl.foreground.SystemForegroundService" android:enabled="@bool/enable_system_foreground_service_default" android:exported="false" android:directBootAware="false" android:foregroundServiceType="microphone|camera|location|mediaPlayback|dataSync"/>
    ```
    Эта служба переднего плана позволяет приложению использовать микрофон, камеру, геолокацию, воспроизведение медиа и синхронизацию данных в фоновом режиме, отображая при этом только минимальное уведомление. Это очень опасно, так как позволяет приложению *постоянно* собирать данные и выполнять действия, пока оно активно.
	
	Mirivan: это SystemForegroundService - часть библиотеки WorkManager (androidx.work). Он используется, когда приложение запускает Foreground Work - задачу, которая должна выполняться надёжно и долго (например, синхронизация данных, загрузка файлов, воспроизведение аудио). Наличие таких флагов (microphone|camera|location|...) не означает, что приложение "пОсТоЯнНо сЛуШаЕт мкрофон или оТсЛеЖиВаЕт геолокацию». Эти категории нужны android для того, чтобы ос понимала, какой тип foreground-задач запускается, и показывала пользователю правильное уведомление. Например, если WorkManager создаст задачу с типом location, то покажет уведомление вида: "Приложение использует геолокацию" (на новых вёдрах это такой кликабельный зеленый круглешок сверху справа экрана).

    ```250:255:app/src/main/AndroidManifest.xml
    <service android:name="one.me.calls.impl.service.CallServiceImpl" android:exported="false" android:foregroundServiceType="microphone|camera|mediaProjection|mediaPlayback"/>
    ```
    Служба, связанная со звонками, также может использовать микрофон, камеру, захват экрана (`mediaProjection`) и воспроизведение медиа в фоновом режиме. Это может позволить записывать разговоры или происходящее на экране.
	
	Mirivan: показать себя любимого или окружающее в зконке? Трансляция экрана есть в звонке? Если нет - без пояснения тогда, а иначе - вполне всё логично. Медиа в фоне - голосовые или музыка, запись разговоров (? а ты точно думаешь что она закодена? Сам я не проводил деобфускации и тут без комментариев).

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
	
	Mirivan: я ниже сделаю разбор на этот момент, поэтому пока брик-поинт тут сделаем.

**5. Попытка анализа обфусцированного кода (на примере `e0.java` и `f1.java`)**

...

**6. Выводы**

Приложение "Макс" представляет собой высокоинтрузивный инструмент для сбора данных, который выходит далеко за рамки функциональности обычного мессенджера. Его способность к сбору широкого спектра личной и системной информации, включая геолокацию, контакты, данные установленных приложений, а также возможность фоновой записи аудио/видео и захвата экрана, вызывает серьезные опасения в отношении конфиденциальности. Обфускация кода лишь усугубляет ситуацию, скрывая истинный масштаб и детали сбора данных и взаимодействия с системой. Это, сука, не просто мессенджер, это, блядь, центр тотального сбора информации.

Mirivan: обфускация - это стандартная практика для приложений, где есть сторонние SDK, особенно рекламные и аналитические. Понятно, что деобфусцировать такой код сложно, но цель тут не всегда "спрятать зло", а банально защитить бизнес-логику и уменьшить вес конечной программы (R8, минифинг ресурсов). Про контакты объяснял. Геолокация - мне неизвестно есть ли в приложении возможность делиться гео, но я там нашел какой-то мап-кит, и более чем уверен, что геолокация нужна как минимум ему для хорошей связки (даже если твоё гео куда-то и сливают, мобильный оператор по решению суда итак запросто отдаст твою локацию, ну мы же не "бандиты"). Фоновая запись аудио/видео так думаю нужна для звонков, как и наложение поверх свех окон, банально чтобы звонки были человеку удобны, к примеру зайти в банк, но не потерять связь при звонке, и видеть плавающее окно самого звонка. Правильно подметили юзеры в Issues - Телеграм и любой другой активный мессенджер делает точно также.

**6. Закрепление в системе и системные модификации**

Приложение "Макс" использует несколько механизмов для закрепления своего присутствия в системе Android и осуществления длительного контроля, хотя прямых доказательств патчинга ядра или других низкоуровневых модификаций системы не обнаружено в `AndroidManifest.xml`.

Mirivan: на счёт BootCompletedReceiver, представим такую ситацию, что ты запустил телефон, а пока он был выключен, тебе приходили не отслеживаемые тобой сообщения условно в ТГ, твой телефон запустился - ты видишь уведомления от Телеграм и МАКС, и это благодаря связке FCM и отслеживание на запуск устройства (это база приложения, которое тебе отправляет в фоне уведомления).

**6.1. Механизмы закрепления:**

Mirivan: от себя раскрыл постепенно всю информацию.

**6.2. Отсутствие прямых признаков патчинга/эксплойтов:**

На основе анализа `AndroidManifest.xml`, прямых разрешений, указывающих на возможность патчинга системных файлов, получения root-доступа или инъекций в другие процессы (таких как `WRITE_SECURE_SETTINGS`, `MOUNT_UNMOUNT_FILESYSTEMS` или использование `Device Administration API` - `BIND_DEVICE_ADMIN`), не обнаружено. Также отсутствуют явные упоминания о рутинге или эксплойтах в самом манифесте.

Результаты `grep`-поиска по обфусцированным файлам на ключевые слова, связанные с системными модификациями (`patch`, `inject`, `hook`, `root`, `shell`, `exploit`, `system`), показали, что большинство совпадений относятся к стандартным вызовам API Android (`System.currentTimeMillis()`, `System.arraycopy()`, `context.getSystemService()`, методы `dispatch*` для событий UI, `getRootView()` для элементов UI). Эти вызовы являются нормальной частью работы любого Android-приложения и не указывают на вредоносный патчинг или системные эксплойты.

Mirivan: (ПРОЧАЯ ИНФОРМАЦИЯ) на счёт других таких горе-исследователей - какой-то чел расфорсил инфу что МАКС получает доступ к руту. В приложения банально нет вызова бинарника su, но есть детекты рута, которые ялвяются частью MyTracker (что является просто детектом рута, ничем более), но не используются в самом МАКС.

**6.3. Выводы по закреплению:**

Приложение "Макс" использует совокупность "легальных" Android-механизмов для обеспечения своего постоянного присутствия и высокой степени контроля над устройством и пользовательскими данными. Несмотря на отсутствие явных признаков низкоуровневого патчинга системы через `AndroidManifest.xml` или обнаруженных обфусцированных функций, глубокая обфускация основного кода не позволяет полностью исключить наличие более сложных или скрытых механизмов воздействия на систему. Однако, даже без них, предоставленные разрешения и используемые компоненты позволяют приложению осуществлять обширную слежку и контроль. 

**7. Анализ на предмет сниффинга и спуфинга трафика**

Приложение "Макс" не запрашивает прямых разрешений, таких как `BIND_VPN_SERVICE`, которые необходимы для создания VPN-соединения и системного перехвата всего сетевого трафика. Однако, некоторые компоненты и их конфигурация могут давать приложению ограниченный контроль над сетевой активностью и данными.

...

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

Mirivan: - Внутреннее поле lastInput - приложение хранит твой последний ввод в чатах (не могу сформулировать мысль).
- SYSTEM_ALERT_WINDOW - теоретически опасно, но скорее всего под звонки.
- REQUEST_INSTALL_PACKAGES - банально для UI взаимодействия.
- mediaProjection - для трансляции экрана в звонках.

**Итоговые выводы по расширенному анализу:**

Приложение "Макс", несмотря на отсутствие явных низкоуровневых эксплойтов, представляет собой мощный инструмент для **глубокого и постоянного сбора данных о пользователе и его активности**. Комбинация обширных разрешений, таких как доступ к микрофону, камере, местоположению, контактам, списку установленных приложений, а также **захват экрана и сбор данных о внутреннем вводе в чатах**, создает полную картину поведения пользователя. Обфускация кода продолжает скрывать точные механизмы и цели сбора и обработки этих данных, что еще больше усиливает опасения по поводу конфиденциальности и безопасности. Приложение активно закрепляется в системе через механизмы автозапуска и постоянных фоновых служб, обеспечивая непрерывную слежку. 

Mirivan: мое личное мнение - это просто еще один мессенджер с интеграцией в экосистему VK. Да, много трекеров, да, слежка, да, реклама. Но никакого "тайного патча ядра" (звучит даже как полный бред) или скрытых эксплойтов здесь нет. Максимум - потеря приватности и куча персонализированной рекламы.
