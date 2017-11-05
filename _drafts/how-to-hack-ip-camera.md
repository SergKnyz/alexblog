---
layout: post
title: "Как я ip камеру настраивал"
permalink: ip-camera-hack
tags: 
---



---

rtsp://admin:Paspaspaspas@192.168.1.179:10554/tcp/av0_0
web page: http://192.168.1.179:28843/admin.htm
webview: http://192.168.1.179:28843

Как похакать ip camera Vstarcam c7837wip

Берем web ui отсюда(http://4pda.ru/forum/lofiversion/index.php?t782299-340.html), архив CH-app-EN53.8.1.14_VSTARCAM

	(Для камер с прошивкой 48.53.72.80 и выше: Начиная с прошивки 48.53.72.80 в камере отключен telnet. Запускаем следующим образом: Зайти в вебморду через браузер, перейти к вкладке "Настройка FTP" и в поле сервера ввести: $(telnetd), сохранить и нажать кнопочку "Тест". telnet запущен.
	В последних вебмордах отсутствуют Mail Settings и FTP Settings. Решается заменой Веб интерфейса на CH-app-EN53.8.1.14_VSTARCAM из шапки. Замена через стандартный "Upgrade Web UI".
