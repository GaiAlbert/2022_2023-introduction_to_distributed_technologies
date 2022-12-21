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
1. Разворачиваем minikube cluster, взаимодействуем с ним через minikube kubectl
2. Пишем манифест на основе образа HashiCorp Vault, манифест в папке lab1, прокидываем порт 8200
манифест:
![image](https://user-images.githubusercontent.com/116584865/205444130-8dc7411d-9efc-46e7-b6db-a4531b38aa52.png)
3. Создаём сервис доступа к контейнеру
![image](https://user-images.githubusercontent.com/116584865/205443905-b7a983af-fc37-42e3-9c9c-69df6c99af0b.png)
4. Прокидываем порт компьютера в контейнер и попадаем в vault по ссылке http://localhost:8200
![signin](https://user-images.githubusercontent.com/116584865/205444001-7152169b-63c5-4277-bcb9-9200ce843c17.png)
5. Открываем логи и находим токен для входа в Vault.
6. Останавливаем minikube cluster
Схема организации контейнеров и сервисов:
![image](https://user-images.githubusercontent.com/116584865/205444106-b82d713f-91b1-47e1-8a08-2c95e6e22b1b.png)