# Пояснительная записка к дипломной работе по профессии "DevOps-инженер"

## Оглавление
1. [Введение](#введение)  
2. [Цели проекта](#цели-проекта)  
3. [Этапы выполнения](#этапы-выполнения)  
   - [Создание облачной инфраструктуры](#создание-облачной-инфраструктуры)  
   - [Создание Kubernetes кластера](#создание-kubernetes-кластера)  
   - [Создание тестового приложения](#создание-тестового-приложения)  
   - [Подготовка системы мониторинга и деплой приложения](#подготовка-системы-мониторинга-и-деплой-приложения)  
   - [Установка и настройка CI/CD](#установка-и-настройка-cicd)  
4. [Заключение](#заключение)  
5. [Используемые ресурсы](#используемые-ресурсы)  

---

## Введение

Данный проект представляет собой практическую реализацию облачной инфраструктуры на базе **Yandex.Cloud** с использованием современных технологий, таких как **Terraform**, **Kubernetes**, **Docker**, **Prometheus**, **Grafana** и **CI/CD-инструментов**. Целью работы является автоматизация процессов создания, настройки и мониторинга облачной инфраструктуры, а также развёртывания тестового приложения.

---

## Цели проекта

Основные цели проекта:
1. Подготовка облачной инфраструктуры на базе Yandex.Cloud.  
2. Создание отказоустойчивого Kubernetes кластера.  
3. Настройка системы мониторинга с использованием Prometheus и Grafana.  
4. Автоматизация сборки и развёртывания тестового приложения через CI/CD.  

---

## Этапы выполнения

### Создание облачной инфраструктуры

На этом этапе была подготовлена облачная инфраструктура в **Yandex.Cloud** с использованием **Terraform**. Были выполнены следующие шаги:

1. **Создание сервисного аккаунта**  
   Создан сервисный аккаунт с необходимыми правами для управления инфраструктурой. Файл конфигурации: [`DevOps-Engineer-Diploma-Terraform/sa-terraform/sa-terraform.tf`](https://github.com/iveisberg/DevOps-Engineer-Diploma-Terraform/blob/main/sa-terraform/sa-terraform.tf).  
   
#### bash-команды
```bash
terraform init
terraform apply
```
```bash
yc iam service-account list
+----------------------+---------------------------+--------+---------------------+-----------------------+
|          ID          |           NAME            | LABELS |     CREATED AT      | LAST AUTHENTICATED AT |
+----------------------+---------------------------+--------+---------------------+-----------------------+
| ajej118jc5hiidtf7skt | vm-svr-usr                |        | 2025-03-09 07:32:23 | 2025-04-13 09:20:00   |
| ajeo5i3oec2rcqedgau0 | terraform-service-account |        | 2025-04-13 09:20:49 | 2025-04-13 09:50:00   |
+----------------------+---------------------------+--------+---------------------+-----------------------+
```
![sa](<Создание облачной инфраструктуры/images/image.png>)

2. **Настройка хранилища для хранения Terraform backend**  
   Настроено хранилище в Yandex Object Storage для будущего хранения стейт-файлов Terraform. Конфигурация находится в файле: [`DevOps-Engineer-Diploma-Terraform/bucket-ts/bucket-ts.tf`](https://github.com/iveisberg/DevOps-Engineer-Diploma-Terraform/blob/main/bucket-ts/bucket-ts.tf).  

#### bash-команды
```bash
terraform init
terraform apply
```
```bash
yc storage bucket list
+-----------+----------------------+------------+-----------------------+---------------------+
|   NAME    |      FOLDER ID       |  MAX SIZE  | DEFAULT STORAGE CLASS |     CREATED AT      |
+-----------+----------------------+------------+-----------------------+---------------------+
| ts-bucket | b1gue9v1tapk50i3uj7m | 1073741824 | STANDARD              | 2025-04-13 09:44:53 |
+-----------+----------------------+------------+-----------------------+---------------------+
```
![bucket](<Создание облачной инфраструктуры/images/image-1.png>)

3. **Настройка backend для Terraform**  
   Настроен S3 bucket в Yandex Object Storage для хранения стейт-файлов Terraform. Конфигурация находится в файле: [`DevOps-Engineer-Diploma-Terraform/k8s-infra/providers.tf`](https://github.com/iveisberg/DevOps-Engineer-Diploma-Terraform/blob/main/k8s-infra/providers.tf).  

#### bash-команды
```bash
terraform init -backend-config="/home/red_usr/DevOps-Engineer-Diploma-Terraform/sa-terraform/output_file/secret.backend.tfvars"
```
![backend](<Создание облачной инфраструктуры/images/image-3.png>)

4. **Создание VPC и подсетей**  
   Создана виртуальная сеть с подсетями в разных зонах доступности. Конфигурация: [`DevOps-Engineer-Diploma-Terraform/k8s-infra/network.tf`](https://github.com/iveisberg/DevOps-Engineer-Diploma-Terraform/blob/main/k8s-infra/network.tf).  

#### bash-команды
```bash
terraform init
terraform apply
```
```bash
yc vpc subnet list
+----------------------+-----------------------+----------------------+----------------------+---------------+-----------------+
|          ID          |         NAME          |      NETWORK ID      |    ROUTE TABLE ID    |     ZONE      |      RANGE      |
+----------------------+-----------------------+----------------------+----------------------+---------------+-----------------+
| e2ltak5eqr9lduu6cokb | default-ru-central1-b | enpu3mcauv9s6ho13dbe |                      | ru-central1-b | [10.129.0.0/24] |
| e2lu80e4ps3h3qb807do | subnet-b              | enppbma16gh1ccaou2iq | enpflo6v47ab1n674agh | ru-central1-b | [10.0.2.0/24]   |
| e9bgaqfdp2v6p4ueda6q | subnet-a              | enppbma16gh1ccaou2iq | enpflo6v47ab1n674agh | ru-central1-a | [10.0.1.0/24]   |
| e9bkd0cjthnjrgce8tnv | default-ru-central1-a | enpu3mcauv9s6ho13dbe |                      | ru-central1-a | [10.128.0.0/24] |
| fl8dvkhusfsr81nec48g | subnet-d              | enppbma16gh1ccaou2iq | enpflo6v47ab1n674agh | ru-central1-d | [10.0.3.0/24]   |
| fl8tfv2grdnjn5q619sm | default-ru-central1-d | enpu3mcauv9s6ho13dbe |                      | ru-central1-d | [10.130.0.0/24] |
+----------------------+-----------------------+----------------------+----------------------+---------------+-----------------+
```
![vpc](<Создание облачной инфраструктуры/images/image-2.png>)

5. **Проверка работоспособности Terraform**  
   Убедились, что команды `terraform apply` и `terraform destroy` выполняются без ошибок.  

Репозиторий с конфигурацией Terraform: [terraform-config](https://github.com/iveisberg/DevOps-Engineer-Diploma-Terraform).  

---

### Создание Kubernetes кластера

Kubernetes кластер был создан с использованием **Terraform** и **Ansible (Kubespray)**. Основные шаги:

1. **Подготовка виртуальных машин**  
   С помощью Terraform созданы 3 виртуальные машины для Kubernetes. Конфигурация: [`DevOps-Engineer-Diploma-Terraform/k8s-infra/k8s-cluster.tf`](https://github.com/iveisberg/DevOps-Engineer-Diploma-Terraform/blob/main/k8s-infra/k8s-cluster.tf).  

#### bash-команды
```bash
terraform apply
```
```bash
yc compute instance list
+----------------------+-------------------+---------------+---------+----------------+-------------+
|          ID          |       NAME        |    ZONE ID    | STATUS  |  EXTERNAL IP   | INTERNAL IP |
+----------------------+-------------------+---------------+---------+----------------+-------------+
| epd0h1av5c3qlqrsc9b9 | k8s-worker-node-1 | ru-central1-b | RUNNING | 51.250.103.208 | 10.0.2.21   |
| fhmkrglq4nbs7jh3g252 | k8s-master        | ru-central1-a | RUNNING | 158.160.50.1   | 10.0.1.22   |
| fhms2djq8d8t22s4l0r6 | microk8s          | ru-central1-a | STOPPED |                | 10.128.0.17 |
| fv45crtrr4o1tt93hfun | k8s-worker-node-2 | ru-central1-d | RUNNING | 130.193.58.177 | 10.0.3.22   |
+----------------------+-------------------+---------------+---------+----------------+-------------+
```
![create VM](<Создание облачной инфраструктуры/images/image-4.png>)

2. **Настройка Ansible**  
   Подготовлены Ansible плейбуки для установки Kubernetes с использованием Kubespray. Файлы находятся в директории: [`kubespray/inventory/k8s-cluster/`](https://github.com/iveisberg/kubespray/tree/master/inventory/k8s-cluster).  

#### bash-команды
```bash
pip install -r requirements.txt
cp -rfp inventory/sample inventory/k8s-cluster
```
```bash
ansible-playbook -i inventory/k8s-cluster/inventory.ini --become -v --user ubuntu test_sudo.yml
```
![check sudo](<Создание Kubernetes кластера/images/image.png>)

3. **Деплой Kubernetes**  
   Выполнен деплой Kubernetes на подготовленные инстансы. Конфигурация сохранена в файле `~/.kube/config`.  

#### bash-команды
```bash
ansible-playbook -i inventory/k8s-cluster/inventory.ini cluster.yml -b -v --user ubuntu
ssh ubuntu@158.160.50.1
ubuntu@master:~$ sudo cat /etc/kubernetes/admin.conf
```
![create k8s](<Создание Kubernetes кластера/images/image-1.png>)

![check k8s](<Создание Kubernetes кластера/images/image-2.png>)

Репозиторий с конфигурацией Ansible: [kubespray/ansible.cfg](https://github.com/iveisberg/kubespray/blob/master/ansible.cfg).  

---

### Создание тестового приложения

Тестовое приложение было создано с использованием **Nginx** и **Docker**. Основные шаги:

1. **Создание Git-репозитория**  
   Создан репозиторий с конфигурацией Nginx и Dockerfile. Репозиторий: [DevOps-Engineer-Diploma-APP/my-app/](https://github.com/iveisberg/DevOps-Engineer-Diploma-APP/tree/main/my-app).  

2. **Сборка Docker образа**  
   Собран Docker образ и загружен в Docker Hub: [`iveisberg/test-app`](https://hub.docker.com/repository/docker/iveisberg/test-app/general).  
   Файл Dockerfile: [`DevOps-Engineer-Diploma-APP/my-app/Dockerfile`](https://github.com/iveisberg/DevOps-Engineer-Diploma-APP/blob/main/my-app/Dockerfile).  

#### bash-команды
```bash
docker build -t iveisberg/test-app:latest .
```
Вход в DockerHub:
```bash
docker login
```        
Пуш, локально собранного образа, в DockerHub:
```bash
docker push iveisberg/test-app:latest
```

---

### Подготовка системы мониторинга и деплой приложения

На этом этапе была настроена система мониторинга и задеплоено тестовое приложение. Основные шаги:

1. **Установка kube-prometheus**  
   Установлен пакет kube-prometheus для мониторинга Kubernetes. Конфигурация: [`kube-prometheus/manifests/`](https://github.com/iveisberg/kube-prometheus/tree/main/manifests).  

#### bash-команды
```bash
kubectl apply --server-side -f manifests/setup
kubectl wait \
	--for condition=Established \
	--all CustomResourceDefinition \
	--namespace=monitoring
kubectl apply -f manifests/
```
```bash
kubectl get pod -o wide -n monitoring
```
[monitoring pods](<Подготовка системы мониторинга и деплой приложения/images>)

![grafana UI](<Подготовка системы мониторинга и деплой приложения/image-1.png>)

![grafana dashboards](<Подготовка системы мониторинга и деплой приложения/image-2.png>)

![grafana k8s](<Подготовка системы мониторинга и деплой приложения/image-3.png>)

2. **Деплой тестового приложения**  
   Приложение задеплоено в Kubernetes. Манифесты находятся в директории: [`DevOps-Engineer-Diploma-Kubernetes/nginx-deployment.yaml`](https://github.com/iveisberg/DevOps-Engineer-Diploma-Kubernetes/blob/main/nginx-deployment.yaml).  

#### bash-команды
```bash
kubectl apply -f nginx-deployment.yaml
kubectl get pod -o wide -n app 
```
![nginx app](<Подготовка системы мониторинга и деплой приложения/image-4.png>)

![web app](<Подготовка системы мониторинга и деплой приложения/image-5.png>)

![web app-2](<Подготовка системы мониторинга и деплой приложения/image-6.png>)

![nginx app k8s](<Подготовка системы мониторинга и деплой приложения/image-7.png>)

3. **Установка Atlantis**  
   Приложение задеплоено в Kubernetes. Манифесты находятся в директории: [`DevOps-Engineer-Diploma-Kubernetes/atlantis-statefulset.yaml`](https://github.com/iveisberg/DevOps-Engineer-Diploma-Kubernetes/blob/main/atlantis-statefulset.yaml).  

   Atlantis позволяет автоматизировать применение изменений Terraform через pull requests в GitHub. Для установки и настройки Atlantis необходимо выполнить следующие шаги:

   **Создание секретов:**  
      - Создаем Personal Access Token в GitHub для доступа к GitHub, используем потом в переменной: `ATLANTIS_GH_TOKEN`
      - Создаем Webhook Secrets и заносим в репозиторий "DevOps-Engineer-Diploma-Terraform" GitHub, используем потом в переменной: `ATLANTIS_GH_WEBHOOK_SECRET`
      - Создаем секрет `atlantis-vcs` в Kubernetes для хранения ключей переменных: `ATLANTIS_GH_TOKEN`, `ATLANTIS_GH_WEBHOOK_SECRET`
      - Создаем секрет `yc-credentials` в Kubernetes для хранения ключей от сервисного аккаунта облака в переменной: `YC_KEY_SECRET`
      - Создаем секрет `secret.backend.tfvars` в Kubernetes для хранения ключей от бакета с backend конфигурацией terraform. В последующем будет использоваться в поде `Atlantis` для инициализации terraform с помощью команды `terraform init -backend-config=/home/atlantis/terraform_files/secret.backend.tfvars` 
      - Создаем секрет `personal.auto.tfvars` в Kubernetes для хранения параметров подключения к облаку.
      - Создаем configmap `atlantis-repo-config.yaml` в Kubernetes для серверных настроек `Atlantis`в поде `Atlantis`. Там же используются наши команды при коммитах в `GitHub`
      - Создаем configmap `terraformrc` в Kubernetes для передачи файла `.terraformrc` в под `Atlantis`. В последующем будет копироваться в поде `Atlantis` в домашнюю директорию с помощью команды `- run: cp /home/atlantis/terraform_files/.terraformrc /home/atlantis/` 
      - Создаем configmap `id-rsa.pub` в Kubernetes для передачи файла `id_ed25519.pub` в под `Atlantis`. В последующем будет использоваться в поде `Atlantis` для передачи открытого ключа на серверы.

   **Создание `atlantis.yml`:**  
      - Создаем файл `atlantis.yml` для настроек `Atlantis` в самом репозитории. Кладем в наш репозитрой с файлами terraform - [`DevOps-Engineer-Diploma-Terraform/atlantis.yaml`](https://github.com/iveisberg/DevOps-Engineer-Diploma-Terraform/blob/main/atlantis.yaml).
      Данный файл содержит конфигурацию для управления автоматизацией инфраструктуры с помощью Atlantis, в нашем случае для Terraform.

![secrets](<Подготовка системы мониторинга и деплой приложения/image-8.png>)

![configmap](<Подготовка системы мониторинга и деплой приложения/image-9.png>)

   **Настройка Webhook в GitHub**
   - Перейти в настройки репозитория с файлами конфигурации Terraform на GitHub.
   - Нажать Webhooks → Add webhook.
   - Указать URL Atlantis (например, http://<node-ip>:30414/events).
   - Указать секрет, который совпадает с ATLANTIS_GH_WEBHOOK_SECRET.
   - Выбрать тип событий: Pull requests и Pushes.

![webhooks](<Подготовка системы мониторинга и деплой приложения/image-10.png>)

#### bash-команды
```bash
ruby -rsecurerandom -e 'puts SecureRandom.hex(32)'
<webhook-secret>

echo -n "yourtoken" > token
echo -n "yoursecret" > webhook-secret

kubectl create secret -n atlantis generic atlantis-vcs --from-file=token --from-file=webhook-secret

kubectl create secret -n atlantis generic yc-credentials --from-file=/home/red_usr/DevOps-Engineer-Diploma-Terraform/sa-terraform/output_file/sa-key.json

kubectl create secret -n atlantis generic secret.backend.tfvars --from-file=/home/red_usr/DevOps-Engineer-Diploma-Terraform/sa-terraform/output_file/secret.backend.tfvars

kubectl create secret -n atlantis generic personal.auto.tfvars --from-file=/home/red_usr/DevOps-Engineer-Diploma-Kubernetes/personal.auto.tfvars

kubectl create configmap -n atlantis terraformrc --from-file=.terraformrc=/home/red_usr/.terraformrc

kubectl create configmap -n atlantis id-rsa.pub --from-file=id_ed25519.pub=/home/red_usr/.ssh/id_ed25519.pub

kubectl apply -f atlantis-repo-config.yaml
kubectl apply -f atlantis-statefulset.yaml 
```

4. **Проверка работы Atlantis**  
   После настройки Atlantis будет автоматически комментировать pull requests с результатами выполнения команд Terraform (`plan` и `apply`). Пример:
   ```bash
   atlantis apply -p k8s-infra
   atlantis plan -p k8s-infra
   ```

![atlantis](<Подготовка системы мониторинга и деплой приложения/image-11.png>)

![atlantis](<Подготовка системы мониторинга и деплой приложения/image-12.png>)

![atlantis](<Подготовка системы мониторинга и деплой приложения/image-13.png>)

![atlantis](<Подготовка системы мониторинга и деплой приложения/image-14.png>)

![atlantis](<Подготовка системы мониторинга и деплой приложения/image-15.png>)

#### На скриншоте ниже отображена ошибка `atlantis/pre_workflow_hook: Pre workflow hook #1 — failed.` при повторном выполнении pre_workflow_hooks:
```bash
- run: wget https://github.com/yandex-cloud/terraform-provider-yandex/releases/download/v0.140.1/terraform-provider-yandex_0.140.1_linux_amd64.zip
```
Причина - файл terraform-provider-yandex_0.140.1_linux_amd64.zip уже бы лскачен. На функционал работы не влияет.

![error](<Подготовка системы мониторинга и деплой приложения/image-17.png>)

#### И самая главная ошибка на скриншоте ниже, которую так и не получилось решить. Просьба принять работу так.
```bash
Error: failed to upload state: operation error S3: PutObject, https response error StatusCode: 501, RequestID: c0fe9da97ac36de3, HostID: , api error NotImplemented: A header you provided implies functionality that is not implemented
```
- Причина - нет доступа к backend S3 для обновления состояния конфигурации Terraform. Причину так и не нашел.

![atlantis](<Подготовка системы мониторинга и деплой приложения/image-16.png>)

- На скриншоте ниже представлена ВМ "public-vm-1",  у которой изменился размер диска с 10 до 15Гб при работе `Atlantis`, т.е. не отработал только один момент по обновлению состояния конфигурации Terraform. Файл Terraform для ее создания - [`DevOps-Engineer-Diploma-Terraform/k8s-infra/main.tf`](https://github.com/iveisberg/DevOps-Engineer-Diploma-Terraform/blob/main/k8s-infra/main.tf).


![test vm](<Подготовка системы мониторинга и деплой приложения/image-18.png>)

#### Сам репозиторий с конфигурацией Kubernetes: [DevOps-Engineer-Diploma-Kubernetes](https://github.com/iveisberg/DevOps-Engineer-Diploma-Kubernetes).  

---

### Установка и настройка CI/CD

Настроен **CI/CD** с использованием **GitHub Actions**. Основные шаги:

#### **1. Подготовка репозитория**
Первым шагом необходимо подготовить репозиторий на GitHub [`DevOps-Engineer-Diploma-APP`](https://github.com/iveisberg/DevOps-Engineer-Diploma-APP), чтобы он содержал все необходимые файлы для автоматизации процесса сборки, тестирования и развертывания приложения.

- **Dockerfile**: Создаем файл `Dockerfile` (был создан на этапе "Создание тестового приложения").
- **Манифест Kubernetes**: Добавляем файл манифеста Kubernetes [`DevOps-Engineer-Diploma-Kubernetes/nginx-deployment.yaml`](https://github.com/iveisberg/DevOps-Engineer-Diploma-Kubernetes/blob/main/nginx-deployment.yaml) для описания того, как приложение будет развернуто в кластере Kubernetes. Этот файл определяют параметры деплоя, такие как количество реплик, порты, образ Docker и тип сервиса.

#### **2. Настройка GitHub Actions**
GitHub Actions позволяет автоматизировать процессы сборки, тестирования и развертывания. Для этого создаем конфигурационный файл [`DevOps-Engineer-Diploma-APP/.github/workflows/deploy.yml`](https://github.com/iveisberg/DevOps-Engineer-Diploma-APP/blob/main/.github/workflows/deploy.yml).

- **Клонирование репозитория**: Первый шаг — клонирование репозитория в среду выполнения GitHub Actions.
- **Авторизация в Docker Hub**: Настраиваем вход в Docker Hub, используя секреты, которые хранятся в настройках репозитория GitHub - `DOCKERHUB_USERNAME` и `DOCKERHUB_TOKEN`.
- **Сборка Docker-образа**: После авторизации выполняется сборка Docker-образа с уникальным тегом (хеш коммита).
- **Публикация образа в Docker Hub**: Собранный образ отправляется в Docker Hub для дальнейшего использования.
- **Обновление манифестов Kubernetes**: Автоматическое обновление манифеста Kubernetes.
- **Развертывание в Kubernetes**: Применяем обновленные манифесты в кластере Kubernetes, используя команду `kubectl apply`.

#### **3. Настройка Kubernetes**
Для успешного развертывания приложения в Kubernetes необходимо убедиться, что кластер настроен корректно.

- **Настраиваем доступ к кластеру**: Загружаем файл конфигурации Kubernetes (`kubeconfig`) и добавляем его в секреты GitHub - `KUBECONFIG`. Этот файл содержит информацию о подключении к кластеру.

#### **4. Описание ключевых переменных**

1. **`DOCKERHUB_USERNAME`**:
   - **Описание**: Имя пользователя Docker Hub.
   - **Цель**: Используется для аутентификации в Docker Hub при публикации образа.
   - **Как создать**: Регистрируемся на [Docker Hub](https://hub.docker.com/) и используем свое имя пользователя.

2. **`DOCKERHUB_TOKEN`**:
   - **Описание**: Токен доступа Docker Hub.
   - **Цель**: Предоставляет доступ к Docker Hub для выполнения операций (например, публикации образов). Использование токена вместо пароля повышает безопасность.
   - **Как создать**: 
     1. Заходим в настройки безопасности аккаунта Docker Hub.
     2. Создаем новый токен доступа Personal access tokens с правами на публикацию образов.

3. **`KUBECONFIG`**:
   - **Описание**: Файл конфигурации Kubernetes, содержащий информацию о подключении к кластеру.
   - **Цель**: Позволяет GitHub Actions взаимодействовать с вашим кластером Kubernetes.
   - **Как создать**:
     1. На мастер хосте кластера Kubernetes выполняем команду:
        ```bash
        cat /etc/kubernetes/admin.conf | base64
        ```
        Это преобразует файл конфигурации в формат Base64.

#### **5. Тестирование CI/CD**
После настройки всех этапов тестируем пайплайн:

1. **Делаем коммит в основную ветку**: Любое изменение в ветке `main` должно автоматически запустить пайплайн GitHub Actions.
2. **Проверяем выполнение шагов**: В разделе **Actions** репозитория на GitHub отслеживаем выполнение каждого шага.
3. **Проверяем развертывание в Kubernetes**: Используем команды `kubectl get pods` и `kubectl get svc`, чтобы убедиться, что приложение успешно развернуто и доступно.

#### **6. Этапы работы CI/CD**
1. **Сборка**: Приложение собирается в виде Docker-образа.
2. **Публикация**: Образ публикуется в Docker Hub.
3. **Обновление**: Манифесты Kubernetes обновляются для использования нового образа.
4. **Развертывание**: Приложение разворачивается в кластере Kubernetes.

#### **7. Преимущества подхода**
- **Автоматизация**: Все этапы, от сборки до развертывания, выполняются автоматически.
- **Надежность**: Каждое изменение проходит через одинаковые этапы, что снижает вероятность ошибок.
- **Масштабируемость**: Подход легко масштабируется для сложных приложений и больших команд.

#### **8. Заключение**
Настройка CI/CD с использованием GitHub Actions, Docker Hub и Kubernetes позволяет создать надежный и автоматизированный процесс разработки, тестирования и развертывания приложений. Этот подход упрощает работу разработчиков и повышает стабильность системы, обеспечивая быстрое внедрение изменений. 

![actions app](<Создание тестового приложения/image.png>)

![image](<Создание тестового приложения/image-1.png>)

![web app](<Создание тестового приложения/image-2.png>)

![k8s app](<Создание тестового приложения/image-3.png>)

Репозиторий с CI/CD конфигурацией: [DevOps-Engineer-Diploma-APP](https://github.com/iveisberg/DevOps-Engineer-Diploma-APP).  

---

## Заключение

В рамках данного проекта была успешно создана облачная инфраструктура на базе **Yandex.Cloud**, развернут Kubernetes кластер, настроена система мониторинга и автоматизирован процесс сборки и развёртывания тестового приложения. Все конфигурации и исходные коды доступны в соответствующих репозиториях.  

---

## Используемые ресурсы

1. [Terraform Documentation](https://www.terraform.io/docs)  
2. [Kubernetes Documentation](https://kubernetes.io/docs)  
3. [Prometheus Documentation](https://prometheus.io/docs)  
4. [Grafana Documentation](https://grafana.com/docs)  
5. [GitHub Actions Documentation](https://docs.github.com/en/actions)  
6. [Atlantis Documentation](https://www.runatlantis.io/docs/)
