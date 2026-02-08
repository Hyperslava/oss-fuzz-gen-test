oss-fuzz-gen. Установка и запуск на примере Kali Linux.
1. Установка зависимостей
Для работы oss-fuzz-gen необходимо установить следующие зависимости:
python3.11, git Docker, Google Cloud SDK

В моём случае python3.11 я устанавливал как альтернативную версию Python, так как в системе уже имелся python3.13.
Для этого я проделал следующие шаги:
wget https://www.python.org/ftp/python3.11.14/Python-3.11.14.tgz
 ![Alt text](images/image(3).png, "Image3")
Скопировать репозиторий:
git clone https://github.com/google/oss-fuzz-gen.git

