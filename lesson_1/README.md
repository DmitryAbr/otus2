-   создать новый проект в Google Cloud Platform, Яндекс облако или на любых ВМ, докере
	Разворачиваем виртуальную машину под управлением esxi
-   далее создать инстанс виртуальной машины с дефолтными параметрами

-   добавить свой ssh ключ в metadata ВМ
-   зайти удаленным ssh (первая сессия), не забывайте про ssh-add
-   поставить PostgreSQL
-   зайти вторым ssh (вторая сессия)
-   запустить везде psql из под пользователя postgres
-   выключить auto commit  
    ***\set AUTOCOMMIT OFF;***  
  	а теперь стоит убедиться что он выключен  
    ***\echo :AUTOCOMMIT;***  
-   сделать  
    в первой сессии новую таблицу и наполнить ее данными  
    ***create table persons(id serial, first_name text, second_name text);  
     insert into persons(first_name, second_name) values('ivan', 'ivanov');  
     insert into persons(first_name, second_name) values('petr', 'petrov');  
     commit;***  
-   посмотреть текущий уровень изоляции: ***show transaction isolation level***
-   начать новую транзакцию в обоих сессиях с дефолтным (не меняя) уровнем изоляции
-   в первой сессии добавить новую запись ***insert into persons(first_name, second_name) values('sergey', 'sergeev');***
-   сделать select * from persons во второй сессии
-   видите ли вы новую запись и если да то почему?  
      к как открыта транзакция, мы не увидим изменения в таблице до её завершения.  
	> Read Committed — уровень изоляции транзакции, выбираемый в Postgres Pro по умолчанию. В транзакции, работающей на этом уровне, запрос SELECT (без предложения FOR UPDATE/SHARE) видит только те данные, которые были зафиксированы до начала запроса; он никогда не увидит незафиксированных данных или изменений, внесённых в процессе выполнения запроса параллельными транзакциями. По сути запрос SELECT видит снимок базы данных в момент начала выполнения запроса. Однако SELECT видит результаты изменений, внесённых ранее в этой же транзакции, даже если они ещё не зафиксированы. Также заметьте, что два последовательных оператора SELECT могут видеть разные данные даже в рамках одной транзакции, если какие-то другие транзакции зафиксируют изменения после запуска первого SELECT, но до запуска второго
-   завершить первую транзакцию - commit;
-   сделать select * from persons во второй сессии
-   видите ли вы новую запись и если да то почему?  
	Была завершина транзакция, при уровне изоляции **read commited** во второй сессии, теперь мы можем увидеть изменения в таблице.
-   завершите транзакцию во второй сессии
-   начать новые но уже repeatable read транзации - ***set transaction isolation level repeatable read;***
![](/lesson_1/pic/isolation_repeateble_read.jpg)
-   в первой сессии добавить новую запись ***insert into persons(first_name, second_name) values('sveta', 'svetova');***
-   сделать select * from persons во второй сессии
-   видите ли вы новую запись и если да то почему?
  	
-   завершить первую транзакцию - commit;
-   сделать select * from persons во второй сессии
-   видите ли вы новую запись и если да то почему?
-   завершить вторую транзакцию
-   сделать select * from persons во второй сессии
-   видите ли вы новую запись и если да то почему?
