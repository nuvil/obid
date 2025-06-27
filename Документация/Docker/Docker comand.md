doПросмотреть версию докера:
docker version
Просмотреть список всех запущенных контейнеров  на хосте:
docker ps
docker ps -a
Просмотреть Image доступные на хосте:
docker images
Удаление образа (остановить образ):
docker rmi name
Для запуска контейнера из образ:
docker run
docker run ubuntu sleep 5
Для остановки:
docker stop id names
для удаление контейнера
docker rm id names
Для загрузки только image:
docker pull name

Для выполнения команды в контейнере
docker exec distracted_mcclintock cat /etc/hosts

запуск docker в фоновом режиме (-d):
docker run **-d** name

для подключение к docker c
docker attach id name 

Запуск с помощью тега:
docker run redis:4.0

Запустить определенный контейнер 
docker run --name redis -d redis:alpine

docker run -i

Подключение к псевдотерминалу 
docker run -it

Отображение порты, Публикация портов на контейнере

docker run -p 80:5000 name

Чтобы сохранить данные, перед удалением контейнера нужно сопоставить каталоги:
docker run -v /opt/datadirl:/var/lib/mysql mysql

Дополнительные сведения о контейнере:
docker inspect name

Просмотр логов контейнера:
docker logs name

Для указания значения для переменной среды:
docker run -e name  dockername
env - переменные среды в docker

Cоздание собственного образа docker
![[Pasted image 20250522180643.png]]

docker build Dockerfile -t mmumshad/my-custom-app
docker push mmushad/my-custom-app


Просмотреть данные
docker history image-name

CMD VS Entrypoint

From Ubuntu 
CMD sleep 5/ ENTYPOINT ["sleep"]

Entypoint ["sleep"]
CMD ["5"]











