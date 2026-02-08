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
Выполнил скрипт configure:
```
sudo ./configure --enable-optimizations
```
Запустил процесс сборки и установки python3.11 в систему как альтернативную версию Python:
```
sudo make altinstall
```
Через некоторое время процесс установки завершился:

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
После установки необходимо сделать так, чтобы команды docker можно было запускать без sudo, как как в будущем docker будет запускаться при работа oss-fuzz-gen, и запуск через sudo приведёт к тому, что будет использоваться не виртульное окружение python3.11, а системное.
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
## Установка oss-fuzz-gen
Скопировать репозиторий:
```
git clone https://github.com/google/oss-fuzz-gen.git
```
Перейти в папку oss-fuzz-gen, создать и активировать виртульное окружение python3.11:
```
cd oss-fuzz-gen
python3.11 -m venv .venv
source senv/bin/activate
```
Установить зависимости из requirements.txt:
```
pip install -r requirements.txt
```
Установить пакет oss-fuzz-gen:
```
pip install .
```
Сообщение Successfully built Oss-Fuzz-gen означает, что установка oss-fuzz-gen успешно выполнена.
## Авторизация в Google Cloud SDK
Проект oss-fuzz-gen поддерживает 2 способа использовать LLM: через Google Cloud SDK (модели Vertex AI Gemini/Claude), или через OpenAI API (модели GPT). Я решил настроить Google Cloud SDK, так как при регистрации активируется бонус на 300$ для использования сервисов Google Cloud, в том числе Vertex AI.
```
gcloud auth login
```

```gcloud auth application-default login
```

```
gcloud auth application-default set-quota-project <id проекта в Google Cloud>
```
Далее необходимо записать в переменные окружения id проекта и регион, в котором активирован VertexAI API (обычно us-central1)
```
export CLOUD_ML_PROJECT_ID=<id проекта в Google Cloud>
export VERTEX_AI_LOCATIONS=us-central1
```

![Alt text](images/image(4).png "Image3")

git clone https://github.com/google/oss-fuzz-gen.git
## Запуск генерации oss-fuzz-gen
Для начало нужно указать цель - репозиторий с проектом, в моём случае это cJSON, ссылку на него я записал в файл input.txt:
```
echo "https://github.com/DaveGamble/cJSON.git" > input.txt
```
Далее я запустил генерацию, указав входной файл и модель, я выбрал vertex_ai_gemini-2-flash-chat:
```
oss-fuzz-gen generate-full -i input.txt -m vertex_ai_gemini-2-flash-chat
```
Пошёл процесс 
