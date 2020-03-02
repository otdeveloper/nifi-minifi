Описание процесса развёртывания Apache miNiFi.
==============================================

[fig01]: illustration/miNiFi01.png
[fig02]: illustration/fig02.png
[fig03]: illustration/fig03.png
[fig04]: illustration/fig04.png
[fig05]: illustration/fig05.png
[fig06]: illustration/fig06.png
[fig07]: illustration/fig07.png
[fig08]: illustration/fig08.png
[fig09]: illustration/fig09.png
[fig10]: illustration/fig10.png
[fig11]: illustration/fig11.png
[fig12]: illustration/fig12.png

Содержание.
----------
```
Описание процесса развёртывания Apache miNiFi.
    Особенности.
    Полезные ресурсы.
    Архитектура.
    Установка и настройка NiFi серверов.
        Установка
        Настройка
    Установка и настройка C2-сервера
        Установка minifi-c2-0.5.0
        Установка minifi-c2 при сборке из исходников
        Настройка.
        Запуск
    Об именовании шаблонов.
    Подготовка шаблонов для приёма данных от miNiFi.
    Подготовка шаблонов для miNiFi.
        Скрытые поля
        Remote Process Group
    miNiFi сервер.
        Установка релиза 0.5.0
        Установка при сборке из исходников
        Настройка
    miNiFi Toolkit.
        Установка релиза 0.5.0
        Установка при сборке из исходников
        Использование
    Краткая сводка.
        Тестовая среда
        NiFi сервер для приёма данных.
        NiFi сервер для подготовки шаблонов.
        RPG minifi-xp-from-db
        RPG Unicredit_MQ_miNiFi
        C2-сервер
        miNiFi 1
        miNiFi 2
    Сборка miNiFi из исходников под CentOS.

```

На 28.02.2020 последний официальный релиз Apache miNiFi – 0.5.0.

При сборке из исходников получаем версию 0.6.0-SNAPSHOT.

Вариант miNiFi собранный на C++ не рассматривается.

Программное обеспечение (далее – ПО) по умолчанию устанавливается в директорию
`/opt`.

Особенности.
------------

ПО Apache miNiFi 0.5.0 собрано с библиотеками nar версии 1.7.0, это накладывает
ограничения на сервер подготовки конфигураций – рекомендуется использование
Apache NiFi 1.7.0. ПО Apache miNiFi 0.6.0-SNAPSHOT собрано с библиотеками nar
версии 1.8.0, соответственно для сервера подготовки конфигураций рекомендуется
использование Apache NiFi 1.8.0. Т.е. необходимо использовать сервер той же
версии, что и nar-файлы в miNiFi. Версию библиотек можно посмотреть командой `ls
lib/nifi-*.jar` – цифры укажут на версию Apache NiFi.

В данном документе дано общее описание работы Apache miNiFi (далее - miNiFi)
независимо от его версии. В случае, если есть выявленные нюансы работы разных
версий, это указывается.

Для сервера Apache NiFi, (далее - NiFi) принимающего данные, ограничений по
версионности нет.

Адреса взаимодействующих серверов должны быть в dns или прописаны в файле hosts
(`/etc/hosts` – unix; `C:\WINDOWS\system32\drivers\etc\hosts` – Windows) на
каждом сервере.

При сохранении шаблонов на NiFi-сервере не сохраняются пароли. Их надо
прописывать или в xml или, после преобразования в yml, вручную (см раздел
«Подготовка шаблонов для miNiFi»).

При преобразовании шаблонов в yml для miNiFi не сохраняются переменные
(Varibles). Соответственно, везде где они использовались в шаблонах меняем на их
значения.

При развёртывании одинаковых установок miNiFi на разных компах, лучше копировать
соответствующую папку с настроенного и работающего сервера. Это позволит
избежать ошибок повторных настроек и будут установлены необходимые библиотеки и
nar'ники. Не забывать переносить внешние библиотеки (JDBC Driver, JavaDDE и
т.п.), если они используются.

При добавлении внешних библиотек и добавления их в conf/bootstrap.conf путь
должен быть прописан в unix-стиле, например, `java.arg.21=-Djava.library.path=C:/JavaDDE`.

При подготовке шаблонов под Windows в процессорах пути указываются как
`C:\\minifi\\logs`. Даже если где-то сработает `C:/minifi/logs`, это не значит,
что сработает везде.

Из-за ошибки регэкспе в сервере Apache NiFi miNiFi Command and Control (C2)
Server (далее - C2-сервер) релиза 0.5.0 раздающимся с [официального сайта](<https://nifi.apache.org/minifi/download.html>), по состоянию на 20.02.2020),
версии шаблонов определяются только по одной цифре, т.е. в интервале от 0 до 9.
Версии шаблонов начиная от 10 не воспринимаются. Ошибка устранена в текущем
SNAPSHOT’е.

Полезные ресурсы.
-----------------

<https://nifi.apache.org/minifi/minifi-java-agent-quick-start.html> - дан список
процессоров, идущих в составе miNiFi.

<https://nifi.apache.org/minifi/system-admin-guide.html> - описание Automatic
Warm-Redeploy.

<https://github.com/apache/nifi-minifi> – исходники.

Архитектура.
------------

Каждая установка miNiFi работает только с одной конфигурацией. Теоретически
можно на одном сервере развернуть несколько копий разнеся их по портам, но это
не проверялось.

Шаблоны, которые в дальнейшем будут преобразованы конфигурационный файлы для
работы miNiFi, подготавливаются на NiFi сервере подготовки шаблонов. C2-сервер
периодически опрашивает сервер подготовки шаблонов на наличие новых версий.
Сервера miNiFi настраиваются на получение шаблонов с описанием процесса
обработки данных (далее - конфигурация) с C2-сервера.

СерверNiFi, на который данные будут передаваться с miNiFi, указывается в
конфигурации.

C2-сервер может быть настроен на один из трёх режимов получения новых шаблонов:

1.  Из собственного хранилища.
2.  От NiFi сервера подготовки шаблонов.
3.  Из хранилища Amazon S3. В нашем случае не рассматривается.

На рисунке показана обобщённая схема.
![fig01]

Краткая сводка необходимых настроек приведена в разделе «Краткая сводка»

Установка и настройка NiFi серверов.
------------------------------------

Установка показана на примере версии 1.11.1 для сервера получения данных и 1.8.0
для сервера подготовки шаблонов.

### Установка

Установка различных версий NiFi серверов для подготовки шаблонов и для приёма данных одинакова.

Для приёма данных:
```
wget http://apache-mirror.rbc.ru/pub/apache/nifi/1.11.1/nifi-1.11.1-bin.tar.gz
tar xzf nifi-1.11.1-bin.tar.gz –С /opt
chown -R root:root /opt/nifi-1.11.1
```

Для подготовки шаблонов:
```
wget https://archive.apache.org/dist/nifi/1.8.0/nifi-1.8.0-bin.tar.gz
tar xzf nifi-1.8.0-bin.tar.gz –С /opt
chown -R root:root /opt/nifi-1.8.0
```

### Настройка

Основные настройки сводятся к указанию порта на котором сервер будет слушать.
Если порт по умолчанию (`8080`) необходимо сменить (например, если сервера
развёрнуты на одной машине), то меняем значение параметра `nifi.web.http.port` в
`conf/nifi.properties` соответствующего NiFi.

Установка и настройка C2-сервера
--------------------------------

Штатно ПО C2-сервера ставится в директорию с указанием номера версии, например,
`minifi-c2-0.5.0`. Что бы не возникали вопросы версионности, показана установка в
директорию `minifi-c2`.

Как было сказано в разделе «Особенности», в релизе 0.5.0 есть ошибка в работе с
номером версии шаблона. Поэтому, рекомендуется использовать версию
0.6.0-SNAPSHOT или старше.

### Установка minifi-c2-0.5.0
```
wget
http://us.mirrors.quenda.co/apache/nifi/minifi/0.5.0/minifi-c2-0.5.0-bin.tar.gz
tar xzf minifi-c2-0.5.0-bin.tar.gz –C /opt
mv /opt/minifi-c2-0.5.0 /opt/minifi-c2
chown -R root:root /opt/minifi-c2
```

### Установка minifi-c2 при сборке из исходников

Процесс показан на примере установки minifi-c2-0.6.0-SNAPSHOT.

После сборки miNiFi из исходников (см. «Сборка miNiFi из исходников»), архив
C2-сервера лежит в `minifi-c2/minifi-c2-assembly/target/`:
```
tar xzf minifi-c2/minifi-c2-assembly/target/minifi-c2-0.6.0-SNAPSHOT-bin.tar.gz
–C /opt
mv /opt/minifi-c2-0.6.0-SNAPSHOT /opt/minifi-c2
```

### Настройка.

Основные требуемые настройки находятся в файле `minifi-c2/conf/minifi-c2-context.xml`.

«Из коробки» C2-сервер настроен на работу со своим хранилищем:
`CacheConfigurationProvider`. Путь к директории, где будут искаться шаблоны для
передачи их на сервера miNiFi находятся в секции `<bean
class="org.apache.nifi.minifi.c2.cache.filesystem.FileSystemConfigurationCache">` - по умолчанию: `files/`.

При использовании этого варианта работы, в этой папке должна быть создана
директория с названием шаблона, который будет раздаваться на сервера miNiFi. В
эту директорию помещаются файлы с именем `config.text.yml.vN`, где N – номер
версии.

Файл `config.text.yml` формируется из сохранённого шаблона и преобразованного в
формат yml с использованием miNiFi Toolkit (см. описание ниже).

Для настройки работы с автоматическим получением шаблонов c NiFi сервера
подготовки шаблонов необходимо или закомментировать секцию `<bean
class="org.apache.nifi.minifi.c2.provider.cache.CacheConfigurationProvider">`
или перенести её в конец секции `<list>`.

В секции `<bean
class="org.apache.nifi.minifi.c2.provider.nifi.rest.NiFiRestConfigurationProvider">` 
указываем адрес NiFi сервера (nifi-server-with-template) и порт (port) на котором будет
проверяться наличие новых версий:
```
<constructor-arg>
    <value>http://<nifi-server-with-template>:<port>/nifi-api</value>
</constructor-arg>
```

В этом случае выкачанные и автоматически преобразованные шаблоны будут
сохранятся в папке `./cache`. При необходимости, этот путь так же можно изменить.
Для каждого шаблона будет создана своя поддиректория с его именем где и будут
сохраняться версии шаблонов.

Для проверки доступности можно использовать следующие команды API NiFi:
```
http://<nifi-server-with-template>:<port>/nifi-api/flow/about
http://<nifi-server-with-template>:<port>/nifi-api/flow/templates
```

### Запуск

```
/opt/minifi-c2/bin/c2.sh start > /opt/minifi-c2/c2.log 2>&1
```

Об именовании шаблонов.
-----------------------

Имя шаблона, который будет выполняться на miNiFi задаётся в конфигурационном
файле `conf/bootstrap.conf` miNiFi сервера (см описание ниже) в параметре
`nifi.minifi.notifier.ingestors.pull.http.query`. Оно указывается как значение
параметра `class`. Именно это название будет запрашиваться на C2-сервере.
C2-сервер, получив запрос, проверяет наличие шаблона с этим именем или в `files/`
или в `cache/`, в зависимости от настроек.

При настройке C2-сервера на автоматическое получение шаблонов c NiFi сервера
подготовки шаблонов, он будет периодически опрашивать этот сервер на наличие
новых версий шаблонов с этим именем.

В настройках C2-сервера имя шаблона указывается без расширения .v*N*.

Подготовка шаблонов для приёма данных от miNiFi.
------------------------------------------------

Основная особенность при подготовке приёма данных от miNiFi заключается в том,
что изначально InputPort должен находится в корне рабочего стола NiFi, а не
внутри группы. Внешне, порт, готовый принимать данные от удалённых серверов
выглядит так:

![fig02]

Дополнительных настроек не требуется.

Для шага подготовки шаблонов для miNiFi, этот порт должен быть активен и в
запущенном состоянии.

После создания цепочки обработки входящих данных из неё можно создать группу и
InputPort сохранит свои свойства.

Подготовка шаблонов для miNiFi.
-------------------------------

Желательно минимизировать обработку данных на miNiFi.

На одном сервере можно размещать несколько шаблонов с разными именами для разных
miNiFi серверов.

[Смотрим список процессоров](https://nifi.apache.org/minifi/minifi-java-agent-quick-start.html),
идущих «в коробке» с miNiFi. По необходимости,
конечно, можно добавить необходимые nar-файлы для расширения функционала. Но
рекомендуется не злоупотреблять и использовать только самые необходимые.

Так же, для исключения несовместимости конфигурации процессоров в разных
версиях, NiFi сервер для подготовки шаблонов должен иметь ту же версию, что и
библиотеки, используемые в miNiFi.

Для версии 0.5.0 в одном шаблоне не работает отправка данных в два разных порта.
Т.е. вариант показанный на рисунке ниже не заработал, а в версии 0.6.0-SNAPSHOT
он работает. Более того, 0.6.0 позволяет слать данные не только на разные
InputPort на одном сервере, но и на разных серверах. Настройка портов показана
ниже.

![fig03]

### Скрытые поля

Как было сказано в разделе «Особенности», при сохранении шаблонов не сохраняются
пароли. Т.е. само поле есть, но значения в нём нет. В xml это выглядит так:

```
<entry>
    <key>Password</key>
</entry>
```

В преобразованном yml’е:

```
    Password:
```

Соответственно, если yml-файл готовится вручную с использование NiFi Toolkit’а,
то можно в xml добавить требуемое значение:

```
<entry>
    <key>Password</key>
    <value>my_pass</value>
</entry>
```

Если же шаблоны для miNiFi преобразуются автоматически, то нужные правки можно
внести в yml-файл:

```
    Password: ‘my_pass’
```

Даже если новые шаблоны будут скачаны miNiFi серверами до правки, после
сохранения внесённых изменений, обновлённые шаблоны будут снова отданы miNiFi.
Т.е. ручного изменения номера версии не требуется.

### Remote Process Group

Для соединения с портом удалённого сервера используется Remote Process Group
(RPG).

![fig04]

При перемещении значка на рабочее поле будет показан экран конфигурации. В нём
необходимо заполнить требуемые поля. В основном, это поле URL и Transport
Protocol. Остальные заполняются по необходимости.

![fig05]

После внесения данных, этот процессор необходимо соединить с предыдущим, иначе
подключение и настройка удалённых портов будет недоступна:

![fig06]

При установлении связи между процессором и RPG необходимо выбрать порт, на
который будут отсылаться данные:

![fig07]

Настроим удалённый порт:

![fig08]

Сначала проведём необходимую конфигурацию (1), затем «включим» (2) порт

![fig09]

Включение порта аналогично включению передачи данных, так что можно
воспользоваться и этим пунктом меню:

![fig10]

Если есть возможность, то желательно протестировать работу шаблона перед его
сохранением как новой версии.

C2-сервер, при опросе NiFi сервера подготовки шаблонов ищет шаблоны строго
определённого имени и содержащие в конце имени символы «.v», а затем цифры.
Поэтому, при необходимости шаблоны можно сохранять с другими именами и
тестировать их на других машинах.

После тестирования, сохраняем шаблон с именем, которое задано в настройках
miNiFi и очередным номером версии. Например, `template_name.v7`.

miNiFi сервер.
--------------

Штатно miNiFi ставится в директорию с указанием номера версии, например,
minifi-0.5.0. Что бы не возникали вопросы версионности, показана установка в
директорию minifi.

### Установка релиза 0.5.0
```
wget http://apache-mirror.rbc.ru/pub/apache/nifi/minifi/0.5.0/minifi-0.5.0-bin.tar.gz
tar xzf minifi-0.5.0-bin.tar.gz –C /opt
mv /opt/minifi-0.5.0 /opt/minifi
chown -R root:root /opt/minifi
```

### Установка при сборке из исходников

Процесс показан на примере установки minifi-0.6.0-SNAPSHOT.

После сборки miNiFi из исходников (см. «Сборка miNiFi из исходников»), архив
miNiFi лежит в `minifi-assembly/target/`:
```
tar xzf minifi-assembly/target/minifi-0.6.0-SNAPSHOT-bin.tar.gz –C /opt
mv /opt/minifi-0.6.0-SNAPSHOT /opt/minifi
```

### Настройка

В директории `/opt/minify` в файле `conf/bootstrap.conf` раскомментировать следующие
строки:
```
nifi.minifi.notifier.ingestors=org.apache.nifi.minifi.bootstrap.configuration.ingestors.PullHttpChangeIngestor
# Hostname on which to pull configurations from
nifi.minifi.notifier.ingestors.pull.http.hostname=c2-server.host.name
# Port on which to pull configurations from
nifi.minifi.notifier.ingestors.pull.http.port=10080
# Path to pull configurations from
nifi.minifi.notifier.ingestors.pull.http.path=/c2/config
# Query string to pull configurations with
nifi.minifi.notifier.ingestors.pull.http.query=class=template_name
```

И внести необходимые изменения в параметры
`nifi.minifi.notifier.ingestors.pull.http.hostname` и 
`nifi.minifi.notifier.ingestors.pull.http.query`.

Для тестирования время опроса (в милисекундах)
`nifi.minifi.notifier.ingestors.pull.http.period.ms` можно уменьшить.

miNiFi Toolkit.
---------------

Устанавливается по необходимости.

### Установка релиза 0.5.0
```
wget http://apache-mirror.rbc.ru/pub/apache/nifi/minifi/0.5.0/minifi-toolkit-0.5.0-bin.tar.gz
tar xzf minifi-toolkit-0.5.0-bin.tar.gz –C /opt
mv /opt/minifi-toolkit-0.5.0 /opt/minifi-toolkit
chown -R root:root /opt/minifi-toolkit
```

### Установка при сборке из исходников

Процесс показан на примере установки minifi-0.6.0-SNAPSHOT.

После сборки miNiFi из исходников (см. «Сборка miNiFi из исходников»), архив
miNiFi Toolkit лежит в `minifi-toolkit/minifi-toolkit-assembly/target/`:
```
tar xzf minifi-toolkit/minifi-toolkit-assembly/target/minifi-toolkit-0.6.0-SNAPSHOT-bin.tar.gz –C /opt
mv /opt/minifi-toolkit-0.6.0-SNAPSHOT /opt/minifi-toolkit
```

### Использование

```
/opt/minifi-toolkit/bin/config.sh transform template_name.v1.xml config.text.yml.v1
```

где `template_name.v1.xml` – пример имени сохранённого шаблона.

Краткая сводка.
---------------

**Примечание**. При установке на Windows XP последняя работающая версия Java 8 -
`jdk-8u152-windows-i586.exe`.

### Тестовая среда

- C2-сервер: `mb-tst2.dmr.local`
- NiFi для шаблонов: `mb-tst2.dmr.local`
- NiFi для приёма данных: `mb-tst2.dmr.local`
- miNiFi 1 (Win XP): `mb-tst7.dmr.local`
- miNiFi 2 (CentOS): `mb-tst1.dmr.local`

На miNiFi 1 отдаётся шаблон `minifi-xp-from-db`, на miNiFi 2 -
`Unicredit_MQ_miNiFi`.

### NiFi сервер для приёма данных.

Используется NiFi из пакета OTP, порт оставлен стандартным.

conf/nifi.properties:
```
nifi.web.http.port=8080
```

### NiFi сервер для подготовки шаблонов.

conf/nifi.properties:
```
nifi.web.http.port=9080
```

### RPG minifi-xp-from-db

![fig11]

### RPG Unicredit_MQ_miNiFi

![fig12]

### C2-сервер

conf/minifi-c2-context.xml:
```
<?xml version="1.0" encoding="UTF-8"?>
<beans default-lazy-init="true"
       xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.1.xsd">

    <bean id="configService" class="org.apache.nifi.minifi.c2.service.ConfigService" scope="singleton">
        <constructor-arg>
            <list>
<!--
                <bean class="org.apache.nifi.minifi.c2.provider.cache.CacheConfigurationProvider">
                    <constructor-arg>
                        <list>
                            <value>text/yml</value>
                        </list>
                    </constructor-arg>
                    <constructor-arg>
                        <bean class="org.apache.nifi.minifi.c2.cache.filesystem.FileSystemConfigurationCache">
                            <constructor-arg>
                                <value>./files</value>
                            </constructor-arg>
                            <constructor-arg>
                                <value>${class}/config</value>
                            </constructor-arg>
                        </bean>
                    </constructor-arg>
                </bean>
-->
                <bean class="org.apache.nifi.minifi.c2.provider.nifi.rest.NiFiRestConfigurationProvider">
                    <constructor-arg>
                        <bean class="org.apache.nifi.minifi.c2.cache.filesystem.FileSystemConfigurationCache">
                            <constructor-arg>
                                <value>./cache</value>
                            </constructor-arg>
                            <constructor-arg>
                                <value>${class}/${class}</value>
                            </constructor-arg>
                        </bean>
                    </constructor-arg>
                    <constructor-arg>
                        <value>http://mb-tst2.dmr.local:9080/nifi-api</value>
                    </constructor-arg>
                    <constructor-arg>
                        <value>${class}.v${version}</value>
                    </constructor-arg>
                </bean>

            </list>
        </constructor-arg>
        <constructor-arg>
            <bean class="org.apache.nifi.minifi.c2.security.authorization.GrantedAuthorityAuthorizer">
                <constructor-arg value="classpath:authorizations.yaml"/>
            </bean>
        </constructor-arg>
    </bean>
</beans>
```

### miNiFi 1

conf/bootstrap.conf:
```
…
#Pull HTTP change notifier configuration

# Hostname on which to pull configurations from
nifi.minifi.notifier.ingestors.pull.http.hostname=mb-tst2.dmr.local
# Port on which to pull configurations from
nifi.minifi.notifier.ingestors.pull.http.port=10080
# Path to pull configurations from
nifi.minifi.notifier.ingestors.pull.http.path=/c2/config
# Query string to pull configurations with
nifi.minifi.notifier.ingestors.pull.http.query=class=minifi-xp-from-db
# Period on which to pull configurations from, defaults to 5 minutes if commented out
#nifi.minifi.notifier.ingestors.pull.http.period.ms=300000
nifi.minifi.notifier.ingestors.pull.http.period.ms=60000
…
```

### miNiFi 2

conf/bootstrap.conf:
```
…
#Pull HTTP change notifier configuration

# Hostname on which to pull configurations from
nifi.minifi.notifier.ingestors.pull.http.hostname=mb-tst2.dmr.local
# Port on which to pull configurations from
nifi.minifi.notifier.ingestors.pull.http.port=10080
# Path to pull configurations from
nifi.minifi.notifier.ingestors.pull.http.path=/c2/config
# Query string to pull configurations with
nifi.minifi.notifier.ingestors.pull.http.query=class= Unicredit_MQ_miNiFi
# Period on which to pull configurations from, defaults to 5 minutes if commented out
#nifi.minifi.notifier.ingestors.pull.http.period.ms=300000
nifi.minifi.notifier.ingestors.pull.http.period.ms=60000
…
```

Сборка miNiFi из исходников под CentOS.
----------------------------

Установить Java. Добавить, например, в `<home_directory>/.bashrc`:
```
export JAVA_HOME=$(dirname $(dirname $(readlink $(readlink $(which java)))))
```

Выполнить
```
source \<home_directory\>/.bashrc
```

По необходимости ставим git:
```
yum install -y git
```

По необходимости ставим maven:
```
yum install -y maven
```

Проверяем версию
```
mvn –version
```

Если версия ниже 3.1.0, то берём с сайта
<https://downloads.apache.org/maven/maven-3/> последнюю версию. Например, для
3.6.3:
```
wget https://www-us.apache.org/dist/maven/maven-3/3.6.3/binaries/apache-maven-3.6.3-bin.tar.gz -P /tmp
tar xf /tmp/apache-maven-3.6.3-bin.tar.gz -C /opt
ln -s /opt/apache-maven-3.6.3 /opt/maven
```

Создаём или правим файл `/etc/profile.d/maven.sh`:
```
export JAVA_HOME=/usr/lib/jvm/jre-openjdk
export M2_HOME=/opt/maven
export MAVEN_HOME=/opt/maven	
export PATH=${M2_HOME}/bin:${PATH}
```

Выполняем
```
source /etc/profile.d/maven.sh
```

И снова проверяем версию.

Если используется прокси, то для работы maven необходимо прописать в
конфигурационный файл `<home_directory>/.m2/settings.xml` следующее, заполнив
поля `host`, `username`, `password` и `nonProxyHosts`:

```
<settings>
    …
    <proxies>
         <!-- Proxy for HTTP -->
         <proxy>
              <id>1</id>
              <active>true</active>
              <protocol>http</protocol>
              <username></username>
              <password></password>
              <host>proxy.dmr.local</host>
              <port>3128</port>
              <nonProxyHosts>local.net</nonProxyHosts>
         </proxy>
        
         <!-- Proxy for HTTPS -->
         <proxy>
              <id>2</id>
              <active>true</active>
              <protocol>https</protocol>
              <username></username>
              <password></password>
              <host>proxy.dmr.local</host>
              <port>3128</port>
              <nonProxyHosts>local.net</nonProxyHosts>
         </proxy>
    </proxies>
    …
</settings>
```

Получим исходники miNiFi и приступим к сборке:
```
git clone <https://github.com/apache/nifi-minifi.git>
cd nifi-minifi/
mvn clean install
```

В процессе сборки mvn не смог почему-то выкачать
`commons-daemon-1.2.1-bin-windows.zip`, пришлось ему помочь:
```
wget https://repo1.maven.org/maven2/commons-daemon/commons-daemon/1.2.1/commons-daemon-1.2.1-bin-windows.zip -P /tmp/
```
