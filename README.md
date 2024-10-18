### PHP ProcTitle extension
forked from MagicalTux/proctitle  
updated to 7.4-fpm  

### 0. Установка сопутствующего ПО

Для Debian (Ubuntu. Mint)
```bash
sudo apt install php-dev
sudo apt install build-essential
```

Для RedHeat (CentOS, Rocky)
```bash
sudo yum install php-devel
sudo yum groupinstall "Development Tools"
```


### 1. Убедитесь, что модуль собран для правильной версии PHP: 
Проверьте, что версия PHP, для которой вы собираете модуль, совпадает с установленной версией PHP.  
Выполните следующие команды, чтобы убедиться, что используется правильная версия PHP:
```bash
php -v
phpize --version
```

### 2. Соберите модуль 
```bash
phpize --clean
phpize
./configure
make clean
make
sudo make install
```

### 3. Проверьте файл конфигурации PHP
Поместите в директорию `/etc/php.d` файл `proctitle.ini` с содержимым 
```ini
extension=proctitle.so
```
Здесь стоит обратить внимание что `make install` может помещать библиотеку не в ту папку, он напишет путь и лучше в файле указать полный путь или же переместить файл.  
Возможные варианты:
```code
/usr/lib/php/20190902
/usr/lib64/php/modules/
/opt/remi/php74/root/usr/lib64/php/modules/
```

### 4. Перезапустите веб-сервер
```bash
sudo systemctl restart php7.4-fpm
```

### 5. Проверка работоспособности
```bash
php -m | grep proctitle
```
Должен выдать `proctitle` без ошибок и предупреждений.  

```bash
php -i | grep proctitle
```
Также должен без ошибок и предупреждений выдать:
```code
/etc/php.d/proctitle.ini
proctitle
proctitle support => enabled
```

Если все еще не работает или работает только в режиме `cli` то проверьте не было ли ошибок при запуске процесса
```bash
journalctl -xe | grep php-fpm
```

### 6. Использование в коде PHP
Разместите условие в начале гланого скрипта `index.php`
```php
if (function_exists('setproctitle')) {
    setproctitle("php-fpm: pool " . getenv('USER') . " " . $_SERVER['SCRIPT_FILENAME'] . $_SERVER['REQUEST_URI']);
}
```
но так как переименование процесса делает это навсегда, а пул fpm процессов продолжает работать,  
то лучше вернуть старое название (не обязательно)
```php
function shutdown()
{
    if (function_exists('setproctitle')) {
        setproctitle("php-fpm: pool www");
    }
}
register_shutdown_function('shutdown');
```
### 7. Отслеживание процессов в консоли Linux
После успешного добавления модуля, в перечне процессов top (нажать "с") и htop мы будем видеть с каким файлом работает fpm процесс и какой RQUEST_URI

Также удобно буде использовать команду:
```bash
watch -n 1 "ps -eo pid,comm,args | grep php-fpm"
```

### References:

http://bugs.php.net/29479  
http://wikipedia.svn.sourceforge.net/viewvc/wikipedia/trunk/extensions/pecl-proctitle/  
http://en.wikipedia.org/wiki/User:Midom  

