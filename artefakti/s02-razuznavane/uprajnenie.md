## Подготовка

Целта на упражнението беше да се направи основно разузнаване (reconnaissance) с nmap и ffuf, да се определи към кои техники от MITRE ATT&CK Navigator спада използването им и да се сканира хост с известни уязвимости, на виртуалните машини, настроени в предишното упражнение.

За целта трябваше да настроя NAT на gateway виртуалната машина, за да може server виртуалната машина да има достъп до интернет. След това разгърнах Docker среда с уязвимостите CVE-2025-3248 и CVE-2019-17564 от vulhub. След това инсталирах nmap и ffuf на gateway виртуалната машина, за да мога да ги използвам за откриване на уязвимостите (като се преструвам, че не знам номера на CVE от vulhub).

## Процес на търсене на уязвимости

Сканирах за услуги и техните версии, работещи на сървъра, със следната nmap команда:

`nmap -p1-10000 -sV 192.168.100.3`

която ми даде този изход:

```
Nmap scan report for 192.168.100.3
Host is up (0.00024s latency).
Not shown: 65531 closed tcp ports (conn-refused)
PORT     STATE SERVICE   VERSION
22/tcp   open  ssh       OpenSSH 10.0p2 Debian 7+deb13u4 (protocol 2.0)
2181/tcp open  eforward?
7860/tcp open  http      Uvicorn
8080/tcp open  http      Apache Tomcat (language: en)
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port2181-TCP:V=7.95%I=7%D=10/6%Time=6AC560E9%P=x86_64-pc-linux-gnu%r(Me
SF:mcache,39,"stat\x20is\x20not\x20executed\x20because\x20it\x20is\x20not\
SF:x20in\x20the\x20whitelist\.\n");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 19.28 seconds
```

Забелязах, че сървърът има http услуга на порт 7680, затова реших да направя curl към тази страница, което ми даде следния изход:

```
<!doctype html>
<html lang="en">
  <head>
    <base href="/" />
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <link rel="icon" href="./assets/favicon-new-BaG6bPWd.ico" />
    <link rel="preconnect" href="https://fonts.googleapis.com" />
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
    <link
      href="https://fonts.googleapis.com/css2?family=Chivo:ital,wght@0,100..900;1,100..900&family=Inter:ital,opsz,wght@0,14..32,100..900;1,14..32,100..900&family=JetBrains+Mono:ital,wght@0,100..800;1,100..800&display=swap"
      rel="stylesheet"
    />
    <title>Langflow</title>
    <script type="module" crossorigin src="./assets/index-hNpyfBay.js"></script>
    <link rel="stylesheet" crossorigin href="./assets/index-CwAwSIfd.css">
  </head>
  <body id="body" class="dark" style="width: 100%; height: 100%">
    <noscript>You need to enable JavaScript to run this app.</noscript>
    <div style="width: 100vw; height: 100vh" id="root"></div>
  </body>
</html>
```

Това е съобщение за грешка, което ми казва, че трябва да включа JavaScript, защото curl сам по себе си не изпълнява JavaScript елементи, но ми показа и че заглавието на услугата е „Langflow". След като потърсих информация, разбрах, че това е някаква локална AI платформа и че има CVE запис – CVE-2025-3248 (същият от началото). От там научих, че екземплярите на Langflow преди версия 1.3.0 имат уязвимост, затова направих curl към http://192.168.100.3:7860/api/v1/version, което ми даде изхода:

```
{"version":"1.2.0","main_version":"1.2.0","package":"Langflow"}
```

Това потвърждава, че сървърът работи с уязвима версия на Langflow. Това е една от двете големи уязвимости.

## Открити уязвимости

**CVE-2025-3248**
Версиите на Langflow преди 1.3.0 са податливи на инжектиране на код в крайната точка `/api/v1/validate/code`. Отдалечен и неавтентикиран атакуващ може да изпраща специално създадени HTTP заявки, за да изпълни произволен код.

**CVE-2019-17564**
Небезопасна десериализация възниква в приложение на Dubbo, което има включено HTTP remoting. Атакуващ може да изпрати POST заявка с Java обект в нея, за да компрометира напълно Provider екземпляр на Apache Dubbo, ако този екземпляр има включен HTTP. Проблемът засяга Apache Dubbo от 2.7.0 до 2.7.4, от 2.6.0 до 2.6.7 и всички версии 2.5.x.
