# Домашнее задание к занятию "10.06. "

## Постмортем

### Краткое описание инцидента (краткая выжимка о инциденте)
- В результате регламентных работ по замене вышедшего из строя оптического оборудования 100G была потеряна связь между нашим сетевым концентратором на восточном побережье США и нашим основным центром обработки данных на восточном побережье США. Связь между этими точками была восстановлена за 43 секунды, но этот кратковременный сбой вызвал цепочку событий, которые привели к ухудшению качества обслуживания на 24 часа 11 минут.
### Предшествующие события (что произошло перед инцидентом)
- Производились регламентные работы по замене вышедшего из строя оптического оборудования 100G между сетевым концентратором на восточном побережье США и основным центром обработки данных на восточном побережье США
### Причина инцидента (из-за чего возник инцидент)
- Потеря связи между сетевым концентратором на восточном побережье США и основным центром обработки данных на восточном побережье США
### Воздействие (на что повлиял инцидент)
- Связь между этими точками была восстановлена за 43 секунды но за это время Оркестратор начал  процесс отмены выбора главной БД. ЦОД на западном побережье США и публичное облако на восточном побережье США смогли установить кворум и начать аварийное переключение кластеров для направления операций записи в ЦОД на западном побережье.
- Когда подключение было восстановлено, уровень приложений немедленно начал направлять трафик записи на серверы на западном побережье.
- Серверы баз данных в ЦОД на восточном побережье содержали короткий период записи, который не был реплицирован на западное побережье. Базы данных в обоих центрах обработки данных теперь содержали записи, которых не было в другом ЦОД.
- В следствии чего многие пользователи не могли получить доступ к своим данным по запросам, и получали несогласованные данные при взаимодействии со службами т.к. Базы данных не были согласованы и в момент разрыва связи и после в процессе решения проблемы
### Обнаружение (когда и как инцидент был обнаружен)
- 2018 21 октября 22:54 UTC внутренние системы мониторинга начали генерировать предупреждения, указывающие на многочисленные сбои в системах.
### Реакция (кто ответил на инцидент, кто был привлечен, какие каналы коммуникации были задействованы)
- First responce team
- Incident coordinator
- Database engineering team
- Incident response team
- Коммуникация с пользователями осуществлялась через твиттер аккаунт, и блог GitHub
### Восстановление (описание действий по устранению инцидента и поведение системы)
- Было принято решение частично остановить обработку некоторых функций сайта для того чтобы обеспечить целостность уже полученных данных от пользователей.
- Составлен план согласно которому, базы данных восстанавливались из бэкапов, синхронизировались в обоих направлениях, восстанавливалась рабочая топология сервиса и возобновлялась работа пользователей.
- Предпринимались шаги по ускорению процессов восстановления БД с внешних носителей, распределению возрастающей нагрузки на уже восстановленные кластеры БД, ускорению проведения отложенных операций записи.
### Таймлайн (последовательное описание ключевых событий инцидента с указанием времени)
- 2018 21 октября 22:52 UTC Разрыв связи, перестроение оркестратора на кластеры западного побережья.
- 2018 21 октября 22:54 UTC Обнаружение многочисленных сбоев в системах.
- 2018 21 октября 23:07 UTC Остановка внутреннего инструмента развертывания.
- 2018 21 октября 23:09 UTC Присвоение «Желтого» статуса обслуживания.
- 2018 21 октября 23:11 UTC Присвоение «Красного» статуса обслуживания.
- 2018 22 октября 00:05 UTC Разработка плана по восстановлению сервиса.
- 2018 22 октября 00:41 UTC Запуск процесса восстановления из резервной копии все застронутых кластеров, поиск возможности ускорить этот процесс.
- 2018 22 октября 06:51 UTC Часть кластеров завершила процесс восстановления и начало репликацию данных с западного побережья.
- 2018 22 октября 11:12 UTC Завершен процесс восстановления всех БД на восточном побережье.
- 2018 22 октября 16:24 UTC Завершен процесс синхронизации реплик, восстановлена топология сервиса.
- 2018 22 октября 16:45 UTC Начало обработки отложенных процессов.
- 2018 22 октября 23:03 UTC Все ожидающие сборки были обработаны, была подтверждена целостность и правильная работа всех систем. Статус сайта был обновлен до зеленого.
### Последующие действия (что нужно предпринять, чтобы инцидент не повторялся)
- Настройка конфигурации Оркестратора таким образом, чтобы предотвратить переход головных баз через границу региона
-	Ускорение перехода на новый механизм сообщения о статусе обслуживания на более понятном пользователю языке, т.к. статусы «зеленый», «желтый», «красный» не дают реальной картины происходящего
-	Предполагается в будущем поддержка резервирования N+1 на уровне объекта. Это даст возможность допустить полный отказ одного центра обработки данных без воздействия на пользователя
