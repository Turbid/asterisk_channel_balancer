# asterisk_channel_balancer
Балансировщик для железок, не имеющих внутреннего балансировщика ( к примеру серия GoIP). 
Суть простая-Asterisk лезет в базу и выбирает каналы у которых не окончился дневной лимит минут и не помеченные как занятые и выстраивает последовательность в зависимости от израсходованных минут по убывающей.
Помечает канал как занятый и пытается звонить. Если получилось- из счетчика вычитаются использованные минуты и они же прибавляются к счетчику usage ( для статистики ). Если нет-
берет следующий канал из списка и звонит в него. Если перебрали все и ничего не получилось- попытается уйти на контекст FALLBACKCONTEXT ( если надо чтобы обработка продолжилась-
задайте переменную перед переходом в балансер).

!! ВНИМАНИЕ !! Уже начавшийся вызов не ограничен по длине, из-за чего возможен перерасход! Если кому будет нужно добавлю ограничение длины звонка на оставшийся лимит.


Установка в чистый Asterisk -
Зависимости:
-должен быть установлен unixODBC и драйвер myODBC
-Asterisk должен быть собран с поддержкой ODBC ( модули func_odbc и res_odbc)
-обязательно должны быть включены модули Asterisk   app_stack, func_math и pbx_ael

Как установить -
Поскольку это разработка для себя, то и делал все руками.
1. Добавляем код AEL из balanced_call.ael   в extensions.ael
2  Проверяем содержимое файлов odbc.ini  и odbcinst.ini  в /etc, добавляем в odbc.ini  ast_routes , при необходимости меняем в ней название базы
3. Создаем базу для астериска или используем готовую и заливаем в нее таблицу balanced_call_channels   из balanced_call_channels.sql . 
4. Заливаем в свои func_odbc.conf  и res_odbc.conf содержимое из соответствующих файлов  ( в res_odbc.conf вписываем свои логин и пароль к базе ).

Перестартуем модули res_odbc , func_odbc и  ael.
Проверяем статус подключения к odbc   - "odbc show all"  - в разделе asterisk_routes  в строке Connected : должно быть yes

В базу заливаем данные - уж как угодно, phpMyAdmin , самописная страничка, консоль. Пример для консоли - группа MSK , канал - SIP-учетка SIPUNI, лимит 150 минут в день
 insert into channels values ('MSK','СИПЮНИ ','SIP/sipuni/','150','0','150','0');
 
 Для вызова балансера предварительно задаем переменную TRUNKGROUP:
 Set(TRUNKGROUP="MSK");
 После этого вызываем сам балансер -
  jump ${EXTEN}@balanced_call;
  
Вроде все.
