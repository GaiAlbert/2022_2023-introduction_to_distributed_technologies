University: [ITMO University](https://itmo.ru/ru/)
Faculty: [FICT](https://fict.itmo.ru)
Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies)
Year: 2022/2023
Group: 
Author: 
Lab: Lab1
Date of create: 
Date of finished: 
Ход решения:
1. Предварительно был установлен и запущен Docker а также установлен и запущен Minikube согласно инструкции.
Запуск пода
2. После запуска кластера и проверки подключения, был создан манифест vault.yaml. Для развертывания пода была использована следущая команда:
minikube kubectl -- apply -f vault.yaml.
4. Убедились, что под был успешно запущен.
5. Манифест:
![image](https://user-images.githubusercontent.com/121129118/208904006-0da09e12-95d4-4eeb-8add-fa0896a34a82.png)
6. Создаём сервис доступа к контейнеру с помощью команды
minikube kubectl -- expose pod vault --type=NodePort --port=8200
![image](https://user-images.githubusercontent.com/121129118/208903068-a98853bc-9e71-45b0-b9cb-20813bdbb508.png)
7. Прокидываем порт компьютера в контейнер и попадаем в vault по ссылке http://localhost:8200
![image](https://user-images.githubusercontent.com/121129118/208903320-d7a366d3-c1ae-4045-9cc4-74b6c202531c.png)
8. Открываем логи и находим токен для входа в Vault при помощи команды minikube kubectl -- logs vault | findstr "Token"
9. Авторизуемся и заходим в vault 
10. ![image](https://user-images.githubusercontent.com/121129118/208904565-9707d9e7-4d10-4391-b2df-ad7bfac9773c.png)
11. Останавливаем minikube cluster
12. Схема организации контейнеров и сервисов:
![image](https://user-images.githubusercontent.com/116584865/205444106-b82d713f-91b1-47e1-8a08-2c95e6e22b1b.png)

