## Введение
Let’s Encrypt представляет собой центр сертификации (Certificate Authority, CA), позволяющий получать и устанавливать бесплатные сертификаты TLS/SSL, тем самым позволяя использовать шифрованный HTTPS на веб-серверах. Процесс получения сертификатов упрощается за счёт наличия клиента Certbot, который пытается автоматизировать большую часть (если не все) необходимых операций. Более подробно о том, как работает LetsEncrypt можно узнать по [этой ссылке](https://letsencrypt.org/ru/how-it-works/).    

При использовании сертификатов LetsEncrypt следует принимать во внимание следующее:
- Let's Encrypt выпускают только `DV-сертификаты (Domain Validation)` - это сертификаты, которые  подтверждают подлинность доменного имени. Для получения `OV SSL (Organization validation)` и `EV SSL (Extended validation)` следует воспользоваться коммерческими сертификатами.
- Бесплатный сертификат Let's Encrypt выпускается на 90 дней. Можно настроить автоматическое продление, но требуется дополнительно проверять правильность открытия сайта через HTTPS после перевыпуска сертификата. Коммерческий сертификат можно выпустить на срок от 1 года до 2х лет.
- Let's Encrypt не дает никаких гарантий и не несет обязательств перед пользователем. Если вдруг произойдет взлом бесплатного сертификата, то компенсации предоставлены не будут.

В этом руководстве мы используем Certbot для получения бесплатного SSL сертификата для Apache на Ubuntu 18.04, а также настроим автоматическое продление этого сертификата.  

## Перед установкой
Перед тем, как начать следовать описанным в этой статье шагам, убедитесь, что у вас есть:  
- Сервер с Ubuntu 18.04, включающий настройку не-рутового (non-root) пользователя с привилегиями `sudo` и настройку файрвола.
- Зарегистрированное доменное имя. В этом руководстве мы будем использовать **example.com**.
- Для вашего сервера настроены обе записи DNS, указанные ниже:
    - Запись `A` для `example.com`, указывающая на публичный IP адрес вашего сервера.
    - Запись `A` для `www.example.com`, указывающая на публичный IP адрес вашего сервера.
- Установленный и настроенный веб-сервер Apache. Убедитесь, что у вас есть настроенный файл виртуального хоста для вашего домена с правильной директивой `ServerName`. 
- В файрволе включено разрешение для трафика HTTPS

## Шаг 1 - Установка Certbot
Перед началом использования Let’s Encrypt для получения SSL сертификаты установим Certbot на ваш сервер.  

Сначала добавим репозиторий:
```
$ sudo add-apt-repository ppa:certbot/certbot
```
Далее нажмите `ENTER`.

Установим пакет Certbot для Apache с помощью `apt`:
```
$ sudo apt install python-certbot-apache
```
Теперь Certbot готов к использованию.

## Шаг 2 - Получение SSL сертификата
Certbot предоставляет несколько способов получения сертификатов SSL с использованием плагинов. Плагин для Apache берёт на себя настройку Apache и перезагрузку конфигурации, когда это необходимо. Для использования плагина выполним команду:
```
$ sudo certbot --apache -d example.com -d www.example.com
```
Эта команда запускает `certbot` с плагином `--apache`, ключи `-d` определяют имена доменов, для которых должен быть выпущен сертификат.

Если это первый раз, когда вы запускаете `certbot`, вам будет предложено ввести адрес электронной почты и согласиться с условиями использования сервиса. После этого `certbot` свяжется с сервером Let’s Encrypt, а затем проверит, что вы действительно контролируете домен, для которого вы запросили сертификат.

Если всё прошло успешно, `certbot` спросит, как вы хотите настроить конфигурацию HTTPS.
```
Please choose whether or not to redirect HTTP traffic to HTTPS, removing HTTP access.
-------------------------------------------------------------------------------
1: No redirect - Make no further changes to the webserver configuration.
2: Redirect - Make all requests redirect to secure HTTPS access. Choose this for
new sites, or if you're confident your site works on HTTPS. You can undo this
change by editing your web server's configuration.
-------------------------------------------------------------------------------
Select the appropriate number [1-2] then [enter] (press 'c' to cancel):
```
Выберите подходящий вариант и нажмите `ENTER`. Конфигурация будет обновлена, а Apache перезапущен для применения изменений. `certbot` выдаст сообщение о том, что процесс прошёл успешно, и где хранятся ваши сертификаты:
```
IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/example.com/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/example.com/privkey.pem
   Your cert will expire on 2020-09-20. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot again
   with the "certonly" option. To non-interactively renew *all* of
   your certificates, run "certbot renew"
 - Your account credentials have been saved in your Certbot
   configuration directory at /etc/letsencrypt. You should make a
   secure backup of this folder now. This configuration directory will
   also contain certificates and private keys obtained by Certbot so
   making regular backups of this folder is ideal.
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le
```
Ваши сертификаты загружены, установлены и работают. Попробуйте перезагрузить ваш сайт с использованием `https://` и вы увидите значок безопасности в браузере. Он означает, что соединение с сайтом зашифровано.

## Шаг 3 - Проверка автоматического обновления сертификата
Сертификаты Let’s Encrypt действительны только 90 дней. Это сделано для того, чтобы пользователи автоматизировали процесс обновления сертификатов. Пакет `certbot`, который мы установили, делает это путём добавления скрипта обновления в `/etc/cron.d`. Этот скрипт запускается раз в день и автоматически обновляет любые сертификаты, которые закончатся в течение ближайших 30 дней.

Для тестирования процесса обновления мы можем сделать “сухой” запуск (dry run) `certbot`:
```
$ sudo certbot renew --dry-run
```
Если вы не видите каких-либо ошибок в результате выполнения этой команды, то всё в полном порядке. При необходимости `certbot` будет обновлять ваши сертификаты и перезагружать Apache для применения изменений. Если автоматическое обновление по какой-либо причине закончится ошибкой, Let’s Encrypt отправит электронное письмо на указанный вами адрес электронной почты с информацией о сертификате, который скоро закончится.

## Заключение
В этом руководстве мы рассмотрели процесс установки клиента Let’s Encrypt `certbot`, загрузили SSL сертификаты для вашего домена, настроили Apache для использования этих сертификатов и настроили процесс автоматического обновления сертификатов.