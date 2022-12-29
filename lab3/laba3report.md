University: [ITMO University](https://itmo.ru/ru/)
Faculty: [FICT](https://fict.itmo.ru)
Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies)
Year: 2022/2023
Group: 
Author: Albert
Lab: Lab3
Date of create: 
Date of finished: 
## Решение:
### Ликбез по основным терминам, используемым в лабораторной работе.
- `configMap` — это объект API, используемый для хранения неконфиденциальных данных в парах ключ-значение. 
ConfigMap позволяет отделить конфигурацию конкретной среды от конфигурации образы контейнеров, чтобы ваши приложения были легко переносимыми. Шаблон манифеста: https://kubernetes.io/docs/concepts/configuration/configmap/
- `replicaSet` - определяется полями, включая селектор, который определяет, как идентифицировать модули, которые он может получить, число. реплик, указывающих, сколько модулей он должен поддерживать, и шаблона модуля, указывающего данные новых модулей он должен создаваться в соответствии с критериями количества реплик. Его задача - поддержание стабильного набора реплик Pods, работающих в любой момент времени. Шаблон манифеста: https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/
- `ingress` — это объект API, определяющий правила, разрешающие внешний доступ. к службам в кластере. Контроллер Ingress выполняет правила, заданные в Ingress. Шаблон: https://kubernetes.io/docs/tasks/access-application-cluster/ingress-minikube/
- `TLS сертификат`- цифровой сертификат, соответствующий стандарту X.509, который предъявляется в процессе использования криптографического протокола TLS. В структуре сертификата не задаётся конкретный алгоритм шифрования для подписи сертфиката, а также алгоритм открытого ключа. Как использовать в minikube с ingress addon: https://minikube.sigs.k8s.io/docs/tutorials/custom_cert_ingress/
- `FQDN`  — это группа полных доменных имен (FQDN), связанных с известными службами Майкрософт. Теги FQDN можно использовать в правилах приложений, чтобы разрешить нужный исходящий сетевой трафик через брандмауэр.
### 1. Создать configMap с переменными: `REACT_APP_USERNAME`, `REACT_APP_COMPANY_NAME`.
- `kubectl apply -f albertconfigmap.yaml` - развернули манифест.
### 2. создать replicaSet с 2 репликами контейнера ifilyaninitmo/itdt-contained-frontend:master и используя ранее созданный configMap передать переменные `REACT_APP_USERNAME`, `REACT_APP_COMPANY_NAME`.
- При помощи переменной окружения `env` добавляем в манифест нужные нам переменные со значениями из `configMap`.
- `kubectl create -f albertrs.yaml` - развернули манифест.
### 3. Включить minikube addons enable ingress и сгенерировать TLS сертификат, импортировать сертификат в minikube.
- Для генерации TLS сертификата воспользуемся инструментом командной строки OpenSSL. 
Чтобы утилитой можно было пользоваться из cmd, добавляем в переменную среды PATH путь к папке `bin`
- Генерируем приватный ключ RSA. Опция -out указывает на имя файла для сохранения ключа, а число 2048 - размер ключа в битах (по умолчанию 512).
`genrsa -out albertlaba3.key 2048`
- Для того чтобы получить сертификат, который можно использовать нужно этот ключ подписать. А для этого надо создать запрос на подпись.
`req -key albert.laba3.key -new -out albert.laba3.csr`
- При создании запроса на подпись нужно указать необходимую информацию. Обязательное поле здесь - это Common Name, здесь мы указываем наше доменное имя (FQDN), то самое, по которому с помощью Ingress, мы будем заходить на сервер - albert.laba3.cloud
![сертификат тлс](https://user-images.githubusercontent.com/121129118/209859427-445d184a-e69c-4e6e-ae6e-4268f82ad0ea.png)
- Можно подписать сертификат тем же ключом, с помощью которого он был создан.
`x509 -signkey albert.laba3.key -in albert.laba3.csr -req -days 30 -out albert.laba3.crt`
![подпись сертификата ключом](https://user-images.githubusercontent.com/121129118/209859388-dc910bcd-32a7-45a9-81dc-064547639a10.png)
- Проверяем наличие сертификата в корневой папке винды
![наличие сертификата](https://user-images.githubusercontent.com/121129118/209859958-a622d504-d06b-4137-b460-8fa25422d794.png)
#### Создание Secret
- Создаём secret командой `kubectl create secret tls albert.lab3-tls --cert="C:\Windows\System32\albert.laba3.crt" --key="C:\Windows\System32\albert.laba3.key"`
### 4. Создать ingress в minikube, где указан ранее импортированный сертификат, FQDN по которому вы будете заходить и имя сервиса который вы создали ранее.
- Подключаем ingress в minikube при помощи двух команд `minikube addons enable ingress`, и `minikube addons enable ingress-dns`
- Пишем манифест для ingress по шаблону, указывая в полях hosts и host - FQDN (albert.laba3.cloud), и в поле secretName - имя только что созданного нами сикрета
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: albertingress
spec:
  tls:
  - hosts:
      - albert.cloud
    secretName: lab3-tls
  rules:
  - host: albert.cloud
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: albertsvc
            port:
              number: 3000
```
### 5. В hosts пропишите FQDN и IP адрес вашего ingress и попробуйте перейти в браузере по FQDN имени.
>Note: Keep in mind that TLS will not work on the default rule because the certificates would have to be issued for all the possible sub-domains. Therefore, hosts in the tls section need to explicitly match the host in the rules section. After the addon is enabled, please run "minikube tunnel" and your ingress resources would be available at "127.0.0.1"
- Исходя из замечания в документации, для работы TLS сертификата нам нужно добавить IP адрес  и FQDN, а именно `127.0.0.1 albert.cloud` в файл hosts, лежащий в: C:\Windows\System32\drivers\etc.
- image.png
### 6. Войдите в веб приложение по вашему FQDN используя HTTPS и проверьте наличие сертификата.
- После добавления разворачиваем ingress `kubectl apply -f albertingress.yaml` и подключаемся к нему при помощи тунеллирования `minikube tunnel`.
- Переходим на страницу нашего FQDN `https://albert.laba3.cloud`, ReactApp выдаёт белый экран. Как я понял, это связано в с тем, что запуск ведётся на Windows, где это реализовано проблемно.
- Сертификат: ![albertcert](https://user-images.githubusercontent.com/121129118/209859326-3988397a-319d-403d-9021-f5c1f3eb2568.png)
### 7. Схема организации контейнеров и сервисов, нарисованная в draw.io
- ![image](https://user-images.githubusercontent.com/121129118/209955763-72dfc5b5-ccc4-486e-8135-918fee372539.png)
