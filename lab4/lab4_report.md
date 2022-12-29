University: [ITMO University](https://itmo.ru/ru/)
Faculty: [FICT](https://fict.itmo.ru)
Course: [Introduction to distributed technologies](https://github.com/itmo-ict-faculty/introduction-to-distributed-technologies)
Year: 2022/2023
Group: 
Author: Albert
Lab: Lab4
Date of create: 
Date of finished: 
## Решение
### Ликбез по основным терминам, используемым в лабораторной работе.
- CNI- container networking interface, проект Cloud Native Cumputing Foundation, состоящий из спецификации и библиотек для написания подключаемых модулей для настройки сетевых интерфейсов в контейнерах Linux, а также ряда поддерживаемых подключаемых модулей. CNI занимается только сетевым подключением контейнеров и удалением выделенных ресурсов при удалении контейнера.
- Calico - сетевой плагин, расширяющий стандартный набор API Kubernetes в плане сетевых политик. Может использоваться в интеграции с Flannel или самостоятельно, покрывая как функции по обеспечению сетевой связности, так и возможности управления доступностью.
- Преимущества Calico относительно стандартного NetworkPolicy:
1. политики могут применяться к любому объекту: pod, контейнер, виртуальная машина или интерфейс;
2. правила могут содержать конкретное действие (запрет, разрешение, логирование);
3. в качестве цели или источника правил может быть порт, диапазон портов, протоколы, HTTP- или ICMP-атрибуты, IP или подсеть (4 или 6 поколения), любые селекторы (узлов, хостов, окружений);
4. дополнительно можно регулировать прохождение трафика с помощью настроек DNAT и политик проброса трафика.
- В общем случае использования ванильного Kubernetes установка CNI сводится к применению файла calico.yaml, скачанного с официального сайта, с помощью kubectl apply -f.
### 1. При запуске minikube установите плагин `CNI=calico` и режим работы `Multi-Node Clusters` одновеременно, в рамках данной лабораторной работы вам нужно развернуть 2 ноды.
- Обратимся к оригинальной инструкции:
- Чтобы создать single-node minikube cluster с плагином Calico, нужно использовать команду `minikube start --network-plugin=cni --cni=calico`
- Чтобы создать multi-node cluster, нужно использовать `minikube start --nodes 2 -p multinode-demo`
- Так как нам нужно установить режим работы и плагин одновременно, объединяем команды в один запрос `minikube start -p multinode-demo --network-plugin=cni --cni=calico --nodes 2`
### 2. Проверьте работу CNI плагина Calico и количество нод, результаты проверки приложите в отчет.
- Получаем список `nodes` командой `kubectl get nodes` 
![image](https://user-images.githubusercontent.com/121129118/209943697-471f8230-5e16-434c-8ebd-ab4dbe514b59.png)
- Проверяем установку Calico в кластере командой `kubectl get pods -l k8s-app=calico-node -A`
![image](https://user-images.githubusercontent.com/121129118/209944410-35afc535-a2fa-44ef-8adc-b10bf1645d42.png)
### 3-4. Для проверки работы Calico мы попробуем одну из функций под названием `IPAM Plugin`. Для проверки режима `IPAM` необходимо для запущеных ранее нод указать `label` по признаку стойки или географического расположения (на ваш выбор).
- Снова обращаемся к оригинальной инструкции- чтобы для ранее запущеных нод указать label, нужно написать следующее (как один из вариантов):
```
kubectl label nodes multinode-demo zone=west
kubectl label nodes multinode-demo-m02 zone=east
```
![image](https://user-images.githubusercontent.com/121129118/209944468-ded30213-851d-4bbb-95bf-d05be2f1387d.png)

### 5. После этого вам необходимо разработать манифест для Calico который бы на основе ранее указанных меток назначал бы IP адреса "подам" исходя из пулов IP адресов которые вы указали в манифесте.
- Как было сказано выше - установка CNI сводится к применению файла calico.yaml, скачанного с официального сайта. Загружаем и разворачиваем манифест `kubectl apply -f calicoctl.yaml`
- ![image](https://user-images.githubusercontent.com/121129118/209946415-c71d66f3-e9a0-4ce4-8858-a44d0afa431b.png)
>192.168.0.0/24 and 192.168.1.0/24 pools for rack-0, rack-1. Let’s get started.
By installing Calico without setting the default IP pool to match, running calicoctl get ippool -o wide shows that Calico created its default IP pool of 192.168.0.0/16. Delete the default IP pool.
- Удаляем дефолтный IPPool `kubectl delete ippools default-ipv4-ippool`, убеждаемся что других нет
- ![image](https://user-images.githubusercontent.com/121129118/209946751-e7a90403-2605-41f6-9eed-0e4543bff6fe.png)
- Создаём новые IPPool ресурсы для наших nodes, для этого создаём yaml файл с манифестом из инструкции, и разворачиваем его
```
apiVersion: projectcalico.org/v3
kind: IPPool
metadata:
   name: zone-east-ippool
spec:
   cidr: 192.168.0.0/24
   ipipMode: Always
   natOutgoing: true
   nodeSelector: zone == "east"
---
apiVersion: projectcalico.org/v3
kind: IPPool
metadata:
   name: zone-west-ippool
spec:
   cidr: 192.168.1.0/24
   ipipMode: Always
   natOutgoing: true
   nodeSelector: zone == "west"
   ```
- ![image](https://user-images.githubusercontent.com/121129118/209947393-b96b9746-d505-4760-8e3b-4c9bb8c28798.png)
- Взаимодействуя с Calico через неймспейс kube-system, проверяем наличие ippool ресурсов `kubectl exec -i -n kube-system calicoctl -- /calicoctl get ippools -o wide`
- ![image](https://user-images.githubusercontent.com/121129118/209948247-b1d82b81-79af-4b8d-b38b-20d9b9aeb36c.png)
### 6-7. Вам необходимо создать `deployment` с 2 репликами контейнера ifilyaninitmo/itdt-contained-frontend:master и передать переменные в эти реплики: `REACT_APP_USERNAME`, `REACT_APP_COMPANY_NAME`. Создать сервис через который у вас будет доступ на эти "поды". Выбор типа сервиса остается на ваше усмотрение.
- `kubectl apply -f deployment.yaml`
- ![image](https://user-images.githubusercontent.com/121129118/209948692-f6120388-f140-4788-ab0c-07a9bac8aecf.png)
- Создадим сервис к контейнеру.
- ![image](https://user-images.githubusercontent.com/121129118/209948857-84e2390f-d00e-4f59-9d85-f090df459648.png)
### 8. Запустить в minikube режим проброса портов и подключитесь к вашим контейнерам через веб браузер.
- `kubectl port-forward service/frontend-service 3000:3000` - запускаем режим проброса портов.
- ![image](https://user-images.githubusercontent.com/121129118/209949389-07946e62-f261-4ac3-95d4-dffe421fc1ab.png)
- Делаем describe pod'a.
- ![image](https://user-images.githubusercontent.com/121129118/209950177-9e5ea0fc-c6d3-40bb-b5b6-9cdf02fd60a5.png)
- Пытаемся испрвить ошибку. Если верить этой схеме, то проблема в CRI или Kubelet.
- ![image](https://user-images.githubusercontent.com/121129118/209949929-def445a1-821d-4b65-b42a-06207fa41184.png)
### 9. Используя kubectl exec зайдите в любой "под" и попробуйте попинговать "поды" используя FQDN имя соседенего "пода", результаты пингов необходимо приложить к отчету.
- Чтобы пропинговать контейнер, нужно использовать команду `kubectl exec -ti frontend-5dbc85d959-b9tmx -- sh`, однако в нашем случае это не поможет, т.к контейнер не может запуститься.
### 10. Схема организации контейнеров и сервисов
![image](https://user-images.githubusercontent.com/121129118/209958609-9a35deb5-4140-4c23-bf64-4ff232acc473.png)
