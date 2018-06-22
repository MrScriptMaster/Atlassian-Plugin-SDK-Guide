# Детальное описание этапов разработки Jira плагина
## Первый пример
В официальной документации Atlassian приводится пример создания простого плагина HelloWorld, который будет описан далее в этом разделе. Этот плагин будет работать на Atlassian Server, но не на JIRA Cloud. Этот пример направлен на то, чтобы вы увидели как строится каркас плагина.

Код плагина доступен в Bitbucket по [этой сслыке](https://bitbucket.org/serverecosystem/myplugin?_ga=2.54505192.1744769574.1529575908-1435873545.1488196200 "HelloWorld plugin").

*Примечание:* похожим образом создаются и другие плагины для Atlassian проектов. SDK построена так, что вы легко поймете какую команду использовать. Например, при создании скелета плагина для других продуктов команда неизменно начинается на *atlas-create-...*, а дальше пишется что именно (jira, bitbucket, stash и др.). Структура всех плагинов очень похожа, поэтому можно начать знакомство с любым из них.

### Шаг 1. Создание скелета плагина с помощью SDK
1. Перейдите в каталог проекта, в котором будет хранится исходный код плагина.
2. Введите команду
	atlas-create-jira-plugin  
Эта команда запустит создание maven-проекта с минимальной конфигурацией.
3. Во время создания в командной оболочке необходимо ввести некоторые данные для сборщика (данные архетипа Maven):
  - groupId - уникальный идентификатор плагина (например, "com.atlassian.tutorial").
  - artifactId - имя файла архива (например, "myPlugin").
  - version - номер версии плагина. Можно нажать "ввод", тогда применится строка 1.0.0-SNAPSHOT.
  - package - имя Java пакета, в котором будет уложен код плагина (например, com.atlassian.tutorial.myPlugin).
4. После заполнения полей нужно ввести Y для подтверждения введенных данных.
Базовая структура плагина имеет следующий вид

	.
	├── LICENSE
	├── README
	├── pom.xml
	└── src
	    ├── main
	    │   ├── java
	    │   │   └── com
	    │   │       └── atlassian
	    │   │           └── tutorial
	    │   │               └── myPlugin
	    │   │                   ├── api
	    │   │                   │   └── >MyPluginComponent.java
	    │   │                   └── impl
	    │   │                       └── >MyPluginComponentImpl.java
	    │   └── resources
	    │       ├── META-INF
	    │       │   └── spring
	    │       │       └── plugin-context.xml
	    │       ├── atlassian-plugin.xml
	    │       ├── css
	    │       │   └── myPlugin.css
	    │       ├── images
	    │       │   ├── pluginIcon.png
	    │       │   └── pluginLogo.png
	    │       ├── myPlugin.properties
	    │       └── js
	    │           └── myPlugin.js
	    └── test
	        ├── java
	        │   ├── it
	        │   │   └── com
	        │   │       └── atlassian
	        │   │           └── tutorial
	        │   │               └── myPlugin
	        │   │                   └── MyComponentWiredTest.java
	        │   └── ut
	        │       └── com
	        │           └── atlassian
	        │               └── tutorial
	        │                   └── myPlugin
	        │                       └── MyComponentUnitTest.java
	        └── resources
	            └── atlassian-plugin.xml

Несложно понять, что плагин представляет собой Java-проект, который оборачивает код JavaScript.  

### Шаг 2. Сборка плагина
Для сборки плагина нужно:
1. Перейти в корневой каталог проекта и ввести команду
	atlas-run
После этого будет запущена сборка. Также будет запущен сервер Jira. Он будет работать под текущим терминалом и если он вам больше не нужен, то вы можете его закрыть, нажав Ctrl-C или Ctrl-D.
2. Откройте сервер Jira, который будет работать по адресу localhost:2990/jira. Для входа используйте учетную запись администратора, которая создается по умолчанию (Login: admin, Password: admin).
3. Зайдите в настройки аддонов и в списке выберите только что созданный плагин. Вы увидите, что плагин имеет несколько базовых модулей и в сущности ничего не делает.
4. Закройте сервер Jira.

### Шаг 3. Редактирование плагина
1. Перейдите в pom.xml файл вашего проекта. Первым делом, что вы должны сделать это изменить секцию <organization>, например так
	<organization>
	   <name>Atlassian SDK Tutorial</name>
	   <url>http://developer.atlassian.com/</url>
	</organization>
2. Теперь вы можете закрыть файл и ввести команду *atlas-run* еще раз.
3. Перейдите по [ссылке](localhost:2990/jira/plugins/servlet/upm "Jira Plugin Management"), чтобы посмотреть изменения в вашем плагине. Как видите, все изменения были подхвачены автоматически.

Теперь давайте разберемся, как же продукты Atlassian подхватывают пользовательский код в плагине. В любом плагине Atlassian вы найдете xml-структуру
> /src/main/resources/atlassian-plugin.xml

Эта структура описывает плагин. Например для Jira плагина она имеет такой вид

	<atlassian-plugin key="${atlassian.plugin.key}" name="${project.name}" plugins-version="2">
	  <plugin-info>
	    <description>${project.description}</description>
	    <version>${project.version}</version>
	    <vendor name="${project.organization.name}" url="${project.organization.url}" />
	    <param name="plugin-icon">images/pluginIcon.png</param>
	    <param name="plugin-logo">images/pluginLogo.png</param>
	  </plugin-info>
	<!-- add our i18n resource -->
	  <resource type="i18n" name="i18n" location="myPlugin"/>
	<!-- add our web resources -->
	  <web-resource key="myPlugin-resources" name="myPlugin Web Resources">
	    <dependency>com.atlassian.auiplugin:ajs</dependency>
	    <resource type="download" name="myPlugin.css" location="/css/myPlugin.css"/>
	    <resource type="download" name="myPlugin.js" location="/js/myPlugin.js"/>
	    <resource type="download" name="images/" location="/images"/>
	    <context>myPlugin</context>
	  </web-resource>
	</atlassian-plugin>

Любой плагин имеет по меньшей мере секцию plugin-info, которая нужна для интеграции его в продукты Atlassian. В зависимости от того, для чего плагин нужен, добавляются одна или несколько секций с модулями, которые и представляют собой клиентский код и которые располагаются в каталоге с ресурсами.

С помощью SDK вы можете сгененировать в вашем проектк модули разных типов, на каждый из которых имеется документация. Например, модуль Web Resource (секция web-resource) позволяет добавить в окно Jira сверху дополнительную кнопку, для добавления возможности выгрузки web-ресурсов.

Давайте посмотрим, как создаются модули.

1. Для начала, находясь в каталоге проекта нужно ввести команду
  > atlas-create-jira-plugin-module
2. Далее нужно выбрать один из поддерживаемых модулей, скажем, модуль 30, который называется Web Section.
3. После этого нужно задать параметры модуля, например
  > Enter Plugin Module Name My Web Section: : mySection
  > Enter Location (e.g. system.admin/mynewsection): my-item-link
  > Show Advanced Setup? (Y/y/N/n) N: : N
4. Чтобы создать еще модуль, наберите 'Y'.
5. Создайте модуль 25 со следующими параметрами
  > Enter Plugin Module Name My Web Item: : myItem
  > Enter Section (e.g. system.admin/globalsettings): system.top.navigation.bar
  > Enter Link URL (e.g. /secure/CreateIssue!default.jspa): deleteMe
  > Show Advanced Setup? (Y/y/N/n) N: : N
6. Введите N, чтобы прекратить создание модулей.

После того как вы создали два модуля, XML структура изменится на такую

	<atlassian-plugin key="${atlassian.plugin.key}" name="${project.name}" plugins-version="2">
	 <plugin-info>
	   <description>${project.description}</description>
	   <version>${project.version}</version>
	   <vendor name="${project.organization.name}" url="${project.organization.url}"/>
	   <param name="plugin-icon">images/pluginIcon.png</param>
	   <param name="plugin-logo">images/pluginLogo.png</param>
	 </plugin-info>
	<!-- add our i18n resource -->
	 <resource type="i18n" name="i18n" location="myPlugin"/>
	<!-- add our web resources -->
	 
	 <web-resource key="myPlugin-resources" name="myPlugin Web Resources">
	   <dependency>com.atlassian.auiplugin:ajs</dependency>
	   <resource type="download" name="myPlugin.css" location="/css/myPlugin.css"/>
	   <resource type="download" name="myPlugin.js" location="/js/myPlugin.js"/>
	   <resource type="download" name="images/" location="/images"/>
	   <context>myPlugin</context>
	 </web-resource>
	 
	 <web-section name="mySection" i18n-name-key="my-section.name" key="my-section" location="my-item-link" 
	weight="1000">
	   <description key="my-section.description">The mySection Plugin</description>
	   <label key="my-section.label"/>
	 </web-section>
	 
	 <web-item name="myItem" i18n-name-key="my-item.name" key="my-item" section="system.top.navigation.bar" 
	weight="1000">
	   <description key="my-item.description">The myItem Plugin</description>
	   <label key="my-item.label"></label>
	   <link linkId="my-item-link">deleteMe</link>
	 </web-item>
	</atlassian-plugin>

### Шаг 4. Быстрая перезагрузка плагина
При разработке плагина исполнение команды atlas-run занимает много времени. Существует возможность быстрой перезагрузки плагина, для чего проделайте следующее.

1. Откройте pom.xml файл.
2. Отыщите секцию enableQuickReload и измените в ней значение false на true, и секцию enableFastdev, которой присвойте значение false.
3. Перейдите в каталог проекта и в командной оболочке введите команду *atlas-run*.

Теперь если вы будете добавлять новые модули в плагин, то, чтобы изменения применились мгновенно, просто вводите команду
> atlas-mvn package

После этого в логе вы должны увидеть следующее 
	[INFO] [talledLocalContainer]     Quick Reload Finished (1727 ms) - 'myPlugin-1.0.0-SNAPSHOT.jar'
что означает, что плагин перезагрузился.

