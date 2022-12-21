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
Предварительно был установлен и запущен Docker а также установлен и запущен Minikube согласно инструкции.

Запуск пода
После запуска кластера и проверки подключения, был создан манифест vault.yaml. Для развертывания пода была использована следущая команда:

minikube kubectl -- apply -f vault.yaml
Убедились, что под был успешно запущен.
манифест:
![image](https://user-images.githubusercontent.com/121129118/208902702-a52b0d32-2a5a-41f7-8ec7-078671730496.png)
3. Создаём сервис доступа к контейнеру с помощью команды
minikube kubectl -- expose pod vault --type=NodePort --port=8200
![image](https://user-images.githubusercontent.com/116584865/205443905-b7a983af-fc37-42e3-9c9c-69df6c99af0b.png)
4. Прокидываем порт компьютера в контейнер и попадаем в vault по ссылке http://localhost:8200
![signin](https://user-images.githubusercontent.com/116584865/205444001-7152169b-63c5-4277-bcb9-9200ce843c17.png)
5. Открываем логи и находим токен для входа в Vault.
6. Останавливаем minikube cluster
Схема организации контейнеров и сервисов:
![image](https://user-images.githubusercontent.com/116584865/205444106-b82d713f-91b1-47e1-8a08-2c95e6e22b1b.png)
