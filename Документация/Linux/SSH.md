secure shell protocol - это защищённый протокол передачи данных через "незащищённую" сеть, он устанавливает соединение между клиентом и сервером, по умолчанию соединение устанавливается на 22 порту по TCP и сервер в свою очередь аутентифицирует клиента и проверяет правильные ли данные для подключения ему были переданы, в качестве данных для подключения могут быть выбраны: 
1. логин и пароль (не безопасно)
2. private (храним как зеницу ока) and public keys (можем светить куда угодно, он используется для того, чтобы показать, что клиент подключается с правильным ключом)
![[Pasted image 20250828231032.png]]
создание ssh ключа:
ssh-keygen -t ed25519, выбираем куда сохраняем, вводим пароль (best practice)
ssh-keygen -t ed25519 -f ~/.ssh/key2 -C "new key"
-f указать сразу директорию
-С указать комментарий
Два способа добавить ssh key на сервер:
1.ssh-copy-id user@server
2.Заходим на сервер и если нет директории .ssh, то создаем её
Дальше в ней создаем файлик authorized_keys
nano .ssh/authorized_keys
и копируем свой ключ в файл authorized_keys
3.Проверяем права на данную директорию https://gist.github.com/etoosamoe/6cdb3e60b341930706084937649f177f
`chown -R $USER:$USER ~/.ssh|`
`chmod 700 ~/.ssh`
`chmod 644 ~/.ssh/*`
`chmod 600 ~/.ssh/id_* ~/.ssh/authorized_keys`
4.Настройка сервера https://gist.github.com/etoosamoe/ecf6e28f9362436bb5f0db7998be3037
cd /etc/ssh
nano /etc/sshd_config
(для безопасности стоит перетаскивать порт на любой другой)
пишем no в authentication: PermitRootLogin
PasswordAuthentication - Запрещаем подключаться по ключам, только после добавление публичных ключей
5.Перезапускаем наш ssh demon
systemctl restart sshd

Для того, чтобы ssh сам понимал куда нам подключаться https://gist.github.com/etoosamoe/07c8ac5cae58d11066ba1fe5ebe9b7a3
1.Создадим файлик config (На Linux за хранение пароля ключа обычно отвечает `ssh-agent`)
**`UseKeychain`** поддерживается только в **OpenSSH для macOS**, начиная с версии 7.3.


Проброс портов
```
ssh -L port_machine:localhost:удаленный_порт сервер_для_подключения
```
## ssh-agent
eval "$(ssh-agent -s)"
ssh-add ~/path/to/ssh