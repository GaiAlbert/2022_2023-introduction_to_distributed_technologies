University: [ITMO University](https://itmo.ru/ru/)    
Faculty: [FICT](https://fict.itmo.ru)    
Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies)    
Year: 2022/2023    
Group:    
Author: Albert    
Lab: Lab2    
Date of create:    
Date of finished: 
## Ход работы:
1. Создаём deployment с двумя репликами контейнера https://hub.docker.com/repository/docker/ifilyaninitmo/itdt-contained-frontendи и передаём в эти реплики следующие данные: `REACT_APP_USERNAME`, `REACT_APP_COMPANY_NAME`.
- `kubectl apply -f lab2_deploy.yaml` - команда, с помощью которой создаём деплой.
- `kubectl get pods` - проверяем созданные поды, их должно быть 2 штуки
![image](https://user-images.githubusercontent.com/121129118/209105761-057205a7-3709-484e-bba0-6f209e45bf83.png)
2. Создаём сервис, через который получим доступ к этим подам. Тип: LoadBalancer
- `kubectl apply -f lab2_service.yaml` - команда для деплоя сервиса
- `kubectl get svc` - проверяем созданный сервис
![image](https://user-images.githubusercontent.com/121129118/209106265-a0112e31-a8d9-4a8e-a82b-0ac35b5b55d6.png)
3. Запускаем в `minikube` режим проброса портов и подключаемся к нашим контейнерам через браузер.
- `minikube service laba2` - с помощью этой команды подключаемся к контейнеру
![image](https://user-images.githubusercontent.com/121129118/209110430-bcf9e878-371d-41dd-bf0b-c837a1eb3dab.png)
![image](https://user-images.githubusercontent.com/121129118/209110498-40c18acc-6575-4938-b9d6-da6162d447ed.png)
4. Проверяем переменные `REACT_APP_USERNAME`, `REACT_APP_COMPANY_NAME` и `Container name`. Первые 2 не меняются, потому что мы их статично прописали в манифесте, третье  изменяется после обновления страницы в следствие того, что сервис LoadBalancer распределяет нагрузку между двумя нодами.
5. Логи контейнеров:
![image](https://user-images.githubusercontent.com/121129118/209111389-670a9802-d18e-40e2-894e-1662256996ab.png)
![image](https://user-images.githubusercontent.com/121129118/209111430-52e7d0d9-1b70-4367-bc33-9fc066c09cae.png)
6. Схема организации контейнеров:
![image](https://user-images.githubusercontent.com/121129118/209111585-1cc01c2b-7796-4282-99b4-82aa8984563b.png)
