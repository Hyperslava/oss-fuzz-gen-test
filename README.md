# oss-fuzz-gen. Установка и запуск на примере Kali Linux.
## 1. Установка зависимостей
Для работы oss-fuzz-gen необходимо установить следующие зависимости:
python3.11, git Docker, Google Cloud SDK
### Установка Python 3.11
В моём случае python3.11 я устанавливал как альтернативную версию Python, так как в системе уже имелся python3.13.
Для этого я проделал следующие шаги:
Скачал архив с python3.11:
```
wget https://www.python.org/ftp/python3.11.14/Python-3.11.14.tgz
```
 ![Alt text](images/image(3).png "Image3")
 
 Распаковал его:
```
sudo tar xzf Python-3.11.14.tgz
```
Перед сборкой python3.11 установил библиотеку libssl-dev, которая необходима для работы pip:
```
sudo apt install libssl-dev
```
 ![Alt text](images/image(2).png "Image3")
 
Выполнил скрипт configure:
```
sudo ./configure --enable-optimizations
```
 ![Alt text](images/image(5).png "Image3")
 
Запустил процесс сборки и установки python3.11 в систему как альтернативную версию Python:
```
sudo make altinstall
```
![Alt text](images/image(6).png "Image3")

Через некоторое время процесс установки завершился:
![Alt text](images/image(7).png "Image3")

### Установка Docker
Для установки Docker на Kali Linux я добавил репозиторий Docker в apt, и уже через него установил все необходимые компоненты для Docker:
```
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian trixie stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list
curl -fsSL https://download.docker.com/linux/debian/gpg |
  sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
![Alt text](images/image(9).png "Image3")
![Alt text](images/image(10).png "Image3")
![Alt text](images/image(11).png "Image3")

После установки необходимо сделать так, чтобы команды docker можно было запускать без sudo, как как в будущем docker будет запускаться при работа oss-fuzz-gen, и запуск через sudo приведёт к тому, что будет использоваться не виртуальное окружение python3.11, а системное.
```
sudo usermode -aG docker $USER
newgrp docker
```
### Установка Google Cloud SDK (Google Cloud CLI)
По аналогии с Docker установил через APT:
```
curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo gpg --dearmor -o /usr/share/keyrings/cloud.google.gpg
echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] https://packages.cloud.google.com/apt cloud-sdk main" | sudo tee -a /etc/apt/sources.list.d/google-cloud-sdk.list
sudo apt update
sudo apt install google-cloud-cli
```
![Alt text](images/image(12).png "Image3")
![Alt text](images/image(13).png "Image3")
![Alt text](images/image(14).png "Image3")
## Установка oss-fuzz-gen
Первым делом нужно скопировать репозиторий:
```
git clone https://github.com/google/oss-fuzz-gen.git
```
![Alt text](images/image(15).png "Image3")

Перейти в папку oss-fuzz-gen, создать и активировать виртуальное окружение python3.11:
```
cd oss-fuzz-gen
python3.11 -m venv .venv
source senv/bin/activate
```
Установить зависимости из requirements.txt:
```
pip install -r requirements.txt
```
![Alt text](images/image(17).png "Image3")
![Alt text](images/image(18).png "Image3")

Установить пакет oss-fuzz-gen:
```
pip install .
```
![Alt text](images/image(19).png "Image3")

Сообщение Successfully built OSS-Fuzz-gen означает, что установка oss-fuzz-gen успешно выполнена:
![Alt text](images/image(20).png "Image3")
## Авторизация в Google Cloud SDK
Проект oss-fuzz-gen поддерживает 2 способа использовать LLM: через Google Cloud SDK (модели Vertex AI Gemini/Claude), или через OpenAI API (модели GPT). Я решил настроить Google Cloud SDK, так как при регистрации активируется бонус на 300$ для использования сервисов Google Cloud, в том числе Vertex AI.

Для начала нужно создать аккаунт Google, если его нет, зайти на сайт https://console.cloud.google.com/welcome, войти в аккаунт и авторизовать сервис Billing.

Далее необходимо активировать Vertex AI API, для этого нужно перейти по адресу https://docs.cloud.google.com/vertex-ai/docs/start/cloud-environment, на странице нажать на Enable the API, и уже на странице проекта в Google Cloud подтвердить активацию Vertex AI API:
![Alt text](images/image(21).png "Image3")

Далее нужно залогиниться в Google Cloud CLI:
```
gcloud auth login
```
![Alt text](images/image(22).png "Image3")
Перейдя по ссылке, необходимо войти в свой аккаунт Google:
![Alt text](images/image(23).png "Image3")

Далее таким же способом залогиниться в Google Auth Library:
```
gcloud auth application-default login
```
![Alt text](images/image(24).png "Image3")

Далее указать свой проект, в котором активирован Vertex AI API:
```
gcloud auth application-default set-quota-project <id проекта в Google Cloud>
```
![Alt text](images/image(25).png "Image3")

Далее необходимо записать в переменные окружения id проекта и регионы, которые доступны VertexAI API в проекте Google Cloud (через запятую, можно указать один - us-central1)
```
export CLOUD_ML_PROJECT_ID=<id проекта в Google Cloud>
export VERTEX_AI_LOCATIONS=us-central1
```
![Alt text](images/image(26).png "Image3")
## Подготовка репозитория
Я буду проверять работу oss-fuzz-gen на репозитории cJSON, на конкретном коммите 3a7bd69.

Для этого я форкнул оригинальный репозиторий https://github.com/DaveGamble/cJSON, создал на Kali Linux SSH-ключ для возможности внести изменения в репозиторий, добавил этот ключ в свой аккаунт GitHub,
клонировал форкнутый репозиторий, откатил состояния до коммита 3a7bd69 и запушил:

![Alt text](images/image(37).png "Image3")
![Alt text](images/image(38).png "Image3")
## Запуск генерации oss-fuzz-gen
Для начала нужно указать цель - репозиторий с проектом, ссылку на него я записал в файл input.txt:
```
echo "https://github.com/Hyperslava/cJSONtest.git" > input.txt
```
![Alt text](images/image(39).png "Image3")

Далее я запустил генерацию, указав входной файл и модель, я выбрал vertex_ai_gemini-2-flash-chat:
```
oss-fuzz-gen generate-full -i input.txt -m vertex_ai_gemini-2-flash-chat
```
![Alt text](images/image(28).png "Image3")

В процессе работы был собран docker-образ:
![Alt text](images/image(29).png "Image3")

Можно увидеть текстовый промтп, отправленный модели:

![Alt text](images/image(30).png "Image3")
![Alt text](images/image(31).png "Image3")

Здесь можно увидеть, что модели была отправлена команда генерации: 
![Alt text](images/image(32).png "Image3")

Можно увидеть результат выполнение одной из промежуточных команд:
![Alt text](images/image(33).png "Image3")

Данный код, который отобразился в логах, походит на сгенерированный с помощью модели:
![Alt text](images/image(34).png "Image3")

Здесь можно увидеть, как модель отвечала текстом:
![Alt text](images/image(35).png "Image3")

По итогу процесс работы был завершён данным образом, что может указывать на то, что в процессе генерации возникли какие-то ошибки, однако видно, что была успешно создана директория с результатами (Wrote data directory for OFG experiments):

![Alt text](images/image(42).png "Image3")
![Alt text](images/image(43).png "Image3")
![Alt text](images/image(44).png "Image3")

Создался такой файл build.sh:
![Alt text](images/image(46).png "Image3")

И такой Dockerfile:

![Alt text](images/image(47).png "Image3")

Но, тем не менее, получилось заставить модель начать генерацию, как было видно по логам.


