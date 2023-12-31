### 12.8 «Резервное копирование баз данных» - "Сергей ШУльга"

### Задание 1. Резервное копирование
Кейс
Финансовая компания решила увеличить надёжность работы баз данных и их резервного копирования.
Необходимо описать, какие варианты резервного копирования подходят в случаях:

#### 1.1. Необходимо восстанавливать данные в полном объёме за предыдущий день.
Для восстановления данных за предыдущий день подойдет дифференциальное резервное копирование. Оно позволяет быстрее восстанавливать данные по сравнению с инкрементным резервным копированием, потому что для этого требуется всего две части резервной копии: полная резервная копия и последняя дифференциальная резервная копия. Данное резервное копирование быстрее, чем полное, но медленнее, чем инкрементное.

#### 1.2. Необходимо восстанавливать данные за час до предполагаемой поломки.
Для восстановления данных за час до предполагаемой поломки подойдет инкрементное резервное копирование. Инкрементное резервное копирование использует полную копию, как начальную точку. Затем выполняется резервное копирование только блоков данных, которые были изменены с момента последнего резервного копирования, с заданным периодом выполнения. Инкрементное резервное копирование можно выполнять так часто, как требуется, так как сохраняются только копии последних изменений. 

#### 1.3.* Возможен ли кейс, когда при поломке базы происходило моментальное переключение на работающую или починенную базу данных.
Использовать репликацию master-slave
Идея репликации основана на том, что кроме «главного» сервера («Мастера») постоянно работают ведомые сервера («слейвы»), которые получают инкрементные бэкапы с мастера в режиме реального времени.

### Задание 2. PostgreSQL
#### 2.1. С помощью официальной документации приведите пример команды резервирования данных и восстановления БД (pgdump/pgrestore).
pg_dump — это программа для создания резервных копий базы данных PostgreSQL. Она создаёт целостные копии, даже если база параллельно используется. Программа pg_dump не препятствует доступу других пользователей к базе данных (ни для чтения, ни для записи).
Следующая команда делает дамп базы данных, используя специальный формат дампа:
```
pg_dump -Fc bd_name > dump.sql
```
Утилита pg_restore предназначена для восстановления базы данных PostgreSQL из архива, созданного командой pg_dump в любом из не текстовых форматов. Она выполняет команды, необходимые для восстановления того состояния базы данных, в котором база была сохранена. Утилита pg_restore может работать в двух режимах. Если указывается имя базы данных, pg_restore подключается к этой базе данных и восстанавливает содержимое архива непосредственно в неё. 
Пример:
```
pg_restore –d bd_name dump.sql
```
#### 2.1.* Возможно ли автоматизировать этот процесс? Если да, то как?
Для резервного копирования по расписанию можно использовать скрипт, например, /scripts/postgresql_dump.sh и создаем задание в планировщике:
```
PATH=/etc:/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin

PGPASSWORD=password
export PGPASSWORD
pathB=/backup
dbUser=dbuser
database=db

find $pathB \( -name "*-1[^5].*" -o -name "*-[023]?.*" \) -ctime +61 -delete
pg_dump -U $dbUser $database | gzip > $pathB/pgsql_$(date "+%Y-%m-%d").sql.gz

unset PGPASSWORD
```
crontab -e
5 0 * * * /scripts/postgresql_dump.sh
Скрипт будет запускаться каждый день в 05:00.

#### Задание 3. MySQL
#### 3.1. С помощью официальной документации приведите пример команды инкрементного резервного копирования базы данных MySQL.
mysqlbackup --defaults-file=/home/dbadmin/my.cnf \
  --incremental --incremental-base=history:last_backup \
  --backup-dir=/home/dbadmin/temp_dir \
  --backup-image=incremental_image1.bi \
   backup-to-image
или через утилиту XtraBackup (утилита горячего резервного копирования с открытым исходным кодом для MySQL):
xtrabackup --backup --target-dir=/data/backups/inc1 --incremental-basedir=/data/backups/base

#### 3.1.* В каких случаях использование реплики будет давать преимущество по сравнению с обычным резервным копированием?
Обеспечение непрерывного доступа к критически важным приложениям и приложениям, ориентированным на клиентов.
- Фокус на аварийном восстановлении.
- Высокая доступность.
- Быстрое возобновление бизнес-операций после сбоя.

