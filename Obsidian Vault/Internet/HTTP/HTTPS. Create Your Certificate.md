## Выпускаем собственные сертификаты

Теперь, когда мы разобрали теорию, самое время приступить к практике! Нам понадобятся OpenSSL и keytool (входит в поставку JDK). Для начала создадим сертификат корневого CA, которым будем подписывать запросы на подпись сертификата клиента и сервера. Сгенерируем приватный ключ RSA зашифрованный AES 256 с паролем "password" длиной 4096 бит (меньше 1024 считается ненадежным) в файл CA-private-key.key:

```
openssl genrsa -aes256 -passout pass:password -out CA-private-key.key 4096
```

Нет какого-то принятого стандарта расширений для файлов, связанных с сертификатами. Мы будем использовать:

-   .key - для приватного ключа
    
-   .csr - для запроса на подпись сертификата
    
-   .pem - для сертификата в Privacy Enchanced Mail формате. Записывается в base64 между -----BEGIN CERTIFICATE----- и -----END CERTIFICATE-----. Также существует Distinguished Encoding Rules (DER) формат, где информация хранится как binary.
    
-   p12 - для хранилища ключей с сертификатами (keystore) и хранилища доверенных сертификатов (truststore) в формате Public-Key Cryptography Standards 12.
    

Далее создадим новый запрос на подпись сертификата CA-certificate-signing-request.csr, передавая информацию о субъекте "CN=Certificate authority" (если не указывать ключ -subj вас попросят указать: Сountry (C), Locality (L), Organisation (O), Organisation Unit (OU), Common Name (CN), Email, Challenge password - все поля, кроме CN опциональны), приватный ключ и пароль от него:

```
openssl req -new -key CA-private-key.key -passin pass:password -subj "/CN=Certificate authority/" -out CA-certificate-signing-request.csr t $3
```

Так как подписать сертификат другим сертификатом пока нельзя, подпишем запрос его же приватным ключом. Получившейся сертификат CA-self-signed-certificate.pem будет самоподписанным со сроком действия 1 день.

```
openssl x509 -req -in CA-certificate-signing-request.csr -signkey CA-private-key.key -passin pass:password -days 1 -out CA-self-signed-certificate.pempemE
```

Теперь у нас есть сертификат, которому в будущем будут доверять наши клиент и сервер. Похожим образом сделаем приватные ключи и запросы на подпись сертификата для них:

```
openssl genrsa -aes256 -passout pass:password -out Server-private-key.key 4096
openssl req -new -key Server-private-key.key -passin pass:password -subj "/CN=localhost/" -out Server-certificate-signing-request.csrt $3

openssl genrsa -aes256 -passout pass:password -out Client-private-key.key 4096
openssl req -new -key Client-private-key.key -passin pass:password -subj "/CN=Client/" -out Client-certificate-signing-request.csr
```

Подпишем запросы нашим сертификатом CA. Ключ CAcreateserial отвечает за создание файла (в данном случае CA-self-signed-certificate.srl) , в котором будет храниться серийный номер для следующего подписываемого этим сертификатом запроса. Серийный номер для текущего же сертификата сгенерируется случайно.

```
openssl x509 -req -in Server-certificate-signing-request.csr -CA CA-self-signed-certificate.pem -CAkey CA-private-key.key -passin pass:password -CAcreateserial -days 1 -out Server-certificate.pemt $4
openssl x509 -req -in Client-certificate-signing-request.csr -CA CA-self-signed-certificate.pem -CAkey CA-private-key.key -passin pass:password -days 1 -out Client-certificate.pem
```

После этого необходимо создать хранилище ключей с сертификатами (keystore) Server-keystore.p12 для использования в нашем приложении. Положим туда сертификат сервера, приватный ключ сервера и защитим хранилище паролем "password":

```
openssl pkcs12 -export -in Server-certificate.pem -inkey Server-private-key.key -passin pass:password -passout pass:password -out Server-keystore.p12      
```

Осталось только создать хранилище доверенных сертификатов (truststore): сервер будет доверять всем клиентам, в цепочке подписания которых есть сертификат из truststore. К сожалению, для Java сертификаты в truststore [должны содержать специальный object identifier](https://github.com/kaikramer/keystore-explorer/issues/35#issuecomment-223837809), а [OpenSSL пока не поддерживает их добавление](https://github.com/openssl/openssl/issues/6684). Поэтому здесь мы прибегнем к поставляемому вместе с JDK keytool:

```
keytool -import -file CA-self-signed-certificate.pem -keystore Server-truststore.p12 -storetype PKCS12 -storepass password -noprompt    
```

Для удобства, все описанные выше действия упакованы в [bash script](https://gist.github.com/Xini1/bec10de6b85b1f77c534f8dd590ac8dc).

## Настройка TLS в Spring Boot приложении

Основой для нашего проекта послужит шаблон с [https://start.spring.io/](https://start.spring.io/) с одной лишь зависимостью Spring Web. Для включения TLS указываем в **application.properties**:

```
server.port=443
server.ssl.enabled=true
server.ssl.protocol=TLS
server.ssl.enabled-protocols=TLSv1.2
```

После этого указываем Spring тип keystore, путь к нему и пароль:

```
server.ssl.key-store-type=PKCS12
server.ssl.key-store=Server-keystore.p12
server.ssl.key-store-password=password
```

Для проверки доступа создадим минимальный **контроллер**:

```
@RestController
public class TlsController {

    @GetMapping
    public String helloWorld() {
        return "Hello, world!";
    }
}
```

Запускаем проект. Попробуем сделать запрос с помощью **curl**:

```
curl https://localhost/

curl: (60) SSL certificate problem: unable to get local issuer certificate
More details here: https://curl.haxx.se/docs/sslcerts.html

curl failed to verify the legitimacy of the server and therefore could not
establish a secure connection to it. To learn more about this situation and
how to fix it, please visit the web page mentioned above.
```

Как видим, curl не доверяет сертификату сервера. Сделаем еще один запрос, указав наш **CA сертификат** в ключе --cacert:

```
curl --cacert CA-self-signed-certificate.pem https://localhost/

Hello, world!
```

На этот раз все сработало, TLS в Spring Boot работает! Мы на этом не остановимся, добавим в приложение **аутентификацию клиента** (указываем truststore):

```
server.ssl.client-auth=need
server.ssl.trust-store-type=PKCS12
server.ssl.trust-store=Server-truststore.p12
server.ssl.trust-store-password=password
```

Запускаем и снова пытаемся выполнить запрос:

```
curl --cacert CA-self-signed-certificate.pem https://localhost/

curl: (35) error:14094412:SSL routines:ssl3_read_bytes:sslv3 alert bad certificate     
```

Очевидно, что сервер закрыл соединение, так как curl не предоставил никакого сертификата. Дополним запрос **клиентским сертификатом и его закрытым ключом**:

```
curl --cacert CA-self-signed-certificate.pem --cert Client-certificate.pem:password --key Client-private-key.key https://localhost/     

Hello, world!
```