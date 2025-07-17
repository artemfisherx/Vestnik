- Проект представляет собой агрегатор новостей с ресурсов: ria.ru, starhit.ru, vc.ru
- Добавлена колонка с гороскопами, которые берутся с сайта 1001goroskop.ru.
- Добавлена колонка "В этот день", информация для которой берется в ru.wikipedia.org.
- В целом, проект функционирует, остались некоторые мелочи, которые хотелось бы реализовать, но пока нет желания, ибо занят другими проектами (см. projectTasks.txt).

Технологический стек:
- Java 17
- Spring Framework 6.1
- PostgreSQL 15
- Thymeleaf
- HTML
- jQuery

В планах еще все покрыть тестами с помощью JUnit.

Из интересного:
- Используется SSE для оповещения пользователей о новых новостях, подгруженных с ria.ru (см. метод checkRiaNews в классе MainController))
- Используется WebSocket для оповещения пользователей о новых новостях, добавленных в ручную(см. класс BrokerConfigurer)
- Поиск реализован с помощью полнотекстового поиска PostgreSQL. Функция выглядит следующим образом:
~~~
declare
res record;
res2 record;
begin

IF numnode(websearch_to_tsquery($1))=0 THEN
RETURN;
END IF;

FOR res IN 
select orig_table, orig_id, ts_rank_cd(title, websearch_to_tsquery($1)) as r1,
ts_rank_cd(txt, websearch_to_tsquery($1),1) as r2
from text_search
where title @@ websearch_to_tsquery($1)
or txt @@ websearch_to_tsquery($1)
order by r1 desc, r2 desc
LOOP

IF(res.orig_table='videos') THEN
execute 'select id, ts_headline(name,websearch_to_tsquery($1)) as title, "" as txt, "videos" as orig_table from videos where id=$2'
into res2
using str, res.orig_id;
ELSE
execute format('select id, ts_headline(title,websearch_to_tsquery($1)) as title, ts_headline(txt,websearch_to_tsquery($1)) as txt, $2 as orig_table from %I where id=$3', res.orig_table)
into res2
using str, res.orig_table, res.orig_id;
END IF;

RETURN NEXT res2;

END LOOP;

end;
~~~ 


