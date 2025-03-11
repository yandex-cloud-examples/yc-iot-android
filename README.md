# Пример работы с Yandex Iot Core на платформе Android

Пример сделан с использованием Android Studio и библиотеки [Paho для Andoid](https://github.com/eclipse/paho.mqtt.android). Для работы нужно иметь созданный [реестр](https://cloud.yandex.ru/docs/iot-core/quickstart#create-registry) и [устройство](https://cloud.yandex.ru/docs/iot-core/quickstart#create-device). [Сертификат удостоверяющего центра](https://storage.yandexcloud.net/mqtt/rootCA.crt) включен в проект в качестве ресурса.

Пример показывает возможность авторизации как с помощью сертификатов, так и с помощью логина и пароля, а также демонстрирует отправку и прием данных.

В интерфейсе пользователя приложения:
* Кнопка **Connect** производит подключение к Yandex IoT Core.
* Кнопка **Publish** отправляет сообщение в топик *publishTopic*.
* Поле **Received message** показывает сообщение, принятое из *subscribeTopic*.

Сообщения принимаются и отправляются в кодировке UTF-8.


Чтобы подключиться к Yandex IoT Core с устройства Android с помощью библиотеки Paho и языка программирования Java, воспользуйтесь практическим руководством [Работа с Yandex IoT Core с устройства Android на языке Java](https://yandex.cloud/ru/docs/tutorials/iot/android-java).


## Сертификаты

Чтобы использовать авторизацию с помощью сертификатов, нужно сделать следующее:

1. Сгенерировать сертификат в формате PKCS12 из [pem сертификатов](https://cloud.yandex.ru/docs/iot-core/quickstart#create-ca) с помощью команды:

    ```bash
    openssl pkcs12 -export -in cert.pem -inkey key.pem -out keystore.p12
    ```

1. Заменить в ресурсах файл `res/raw/your_cert.p12` на сгенерированный в п.1 сертификат.
1. В файле `MainActivity.java` указать актуальные [топики](https://cloud.yandex.ru/docs/iot-core/concepts/topic) публикации и подписки:

    ```java
    private static String publishTopic = "$devices/<device id>/events";
    private static String subscribeTopic = "$devices/<device id>/commands";
    ```

По умолчанию, пример использует авторизацию с помощью сертификатов.

За настройки клиента для авторизации с помощью сертификатов отвечает следующий код в `MainActivity.java`:

```java
// Use this to connect using client certificate
try {  
    sslSocketFactory = getSocketFactory(
    getApplicationContext().getResources().openRawResource(R.raw.root_ca),
    getApplicationContext().getResources().openRawResource(R.raw.your_cert), "");
} catch (Exception e) {  
 e.printStackTrace();
}
```


## Логин и пароль

Чтобы использовать авторизацию с помощью логина и пароля:

1. Сгенерировать пароли для [реестра](https://cloud.yandex.ru/docs/iot-core/operations/password/registry-password) или для [устройства](https://cloud.yandex.ru/docs/iot-core/operations/password/device-password).
1. В файле `MainActivity.java` указать актуальные логин или пароль:

    ```java
    private static String mqttUserName = "<client username>";
    private static String mqttPassword = "<client password>";
    ```

1. В файле `MainActivity.java` указать актуальные [топики](https://cloud.yandex.ru/docs/iot-core/concepts/topic) публикации и подписки:

    ```java
    private static String publishTopic = "$devices/<device id>/events";
    private static String subscribeTopic = "$devices/<device id>/commands";
    ```

1. Закомменировать код, отвечающий авторизацию с помощью сертификатов:

    ```java
    // Use this to connect using client certificate  
    //   try {  
    //       sslSocketFactory = getSocketFactory(  
    //       getApplicationContext().getResources().openRawResource(R.raw.root_ca),  
    //       getApplicationContext().getResources().openRawResource(R.raw.your_cert), "");  
    //   } catch (Exception e) {  
    //       e.printStackTrace();  
    //   }  
    ```

1. Раскомментировать код, отвечающий за авторизацию с помощью логина и пароля:

    ```java
    // Use this to connect using username and password  
    options.setUserName(mqttUserName); 
    options.setPassword(mqttPassword.toCharArray());  
    try { sslSocketFactory = getSocketFactory(
         getApplicationContext().getResources().openRawResource(R.raw.root_ca), null, ""); 
    } catch (Exception e) { e.printStackTrace(); }  
    ```


### Особенности примера

При переносе кода данного примера в другие проекты, следует обратить внимание на следующие особенности:

1. Приложению нужно в `AndroidManifest.xml` дать несколько разрешений:

    ```xml
    <uses-permission android:name="android.permission.WAKE_LOCK"/>
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
    <uses-permission android:name="android.permission.INTERNET"/>
    ```

1. Pacho Android клиент запускает сервис. Это нужно указать в `AndroidManifest.xml`:

   ```xml
   <service android:name="org.eclipse.paho.android.service.MqttService"/>
   ```

1. В примере класс `AdditionslKeyStoresSSLSocketFactory` используется для работы с самоподписными сертификатами.