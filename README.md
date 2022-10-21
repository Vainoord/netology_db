# Домашнее задание к занятию "6.5. Elasticsearch"

## Задача 1

В этом задании вы потренируетесь в:
- установке elasticsearch
- первоначальном конфигурировании elastcisearch
- запуске elasticsearch в docker

Используя докер образ [elasticsearch:7](https://hub.docker.com/_/elasticsearch) как базовый:

- составьте Dockerfile-манифест для elasticsearch
- соберите docker-образ и сделайте `push` в ваш docker.io репозиторий
- запустите контейнер из получившегося образа и выполните запрос пути `/` c хост-машины

Требования к `elasticsearch.yml`:
- данные `path` должны сохраняться в `/var/lib`
- имя ноды должно быть `netology_test`

В ответе приведите:
- текст Dockerfile манифеста
- ссылку на образ в репозитории dockerhub
- ответ `elasticsearch` на запрос пути `/` в json виде

Подсказки:
- при сетевых проблемах внимательно изучите кластерные и сетевые настройки в elasticsearch.yml
- при некоторых проблемах вам поможет docker директива ulimit
- elasticsearch в логах обычно описывает проблему и пути ее решения
- обратите внимание на настройки безопасности такие как `xpack.security.enabled`
- если докер образ не запускается и падает с ошибкой 137 в этом случае может помочь настройка `-e ES_HEAP_SIZE`
- при настройке `path` возможно потребуется настройка прав доступа на директорию

Далее мы будем работать с данным экземпляром elasticsearch.

#### Ответ

Dockerfile:
```
FROM elasticsearch:7.17.6

RUN mkdir /var/lib/elasticsearch &&\
    mkdir /var/log/elasticsearch &&\
    chown -R elasticsearch:elasticsearch /var/lib/elasticsearch &&\
    chown -R elasticsearch:elasticsearch /var/log/elasticsearch

USER elasticsearch

COPY elasticsearch.yml /usr/share/elasticsearch/config/elasticsearch.yml

EXPOSE 9200 9300

CMD ["/usr/sbin/initctl"]

CMD ["/usr/share/elasticsearch/bin/elasticsearch"]
```
Образ elasticsearch:  
https://hub.docker.com/repository/docker/vainoord/elasticsearch/tags?page=1&ordering=last_updated
```
{
  "name" : "netology_test",
  "cluster_name" : "netology_cluster",
  "cluster_uuid" : "-1g8r64ZT4-os_0yOZGFKg",
  "version" : {
    "number" : "7.17.6",
    "build_flavor" : "default",
    "build_type" : "docker",
    "build_hash" : "f65e9d338dc1d07b642e14a27f338990148ee5b6",
    "build_date" : "2022-08-23T11:08:48.893373482Z",
    "build_snapshot" : false,
    "lucene_version" : "8.11.1",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```
## Задача 2

В этом задании вы научитесь:
- создавать и удалять индексы
- изучать состояние кластера
- обосновывать причину деградации доступности данных

Ознакомтесь с [документацией](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-create-index.html)
и добавьте в `elasticsearch` 3 индекса, в соответствии со таблицей:

| Имя | Количество реплик | Количество шард |
|-----|-------------------|-----------------|
| ind-1| 0 | 1 |
| ind-2 | 1 | 2 |
| ind-3 | 2 | 4 |

Получите список индексов и их статусов, используя API и **приведите в ответе** на задание.

Получите состояние кластера `elasticsearch`, используя API.

Как вы думаете, почему часть индексов и кластер находится в состоянии yellow?

Удалите все индексы.

**Важно**

При проектировании кластера elasticsearch нужно корректно рассчитывать количество реплик и шард,
иначе возможна потеря данных индексов, вплоть до полной, при деградации системы.

#### Ответ

Создание индексов:
```
PUT /ind-1
{
  "settings": {
    "index": {
      "number_of_shards": 1,  
      "number_of_replicas": 0
    }
  }
}
PUT /ind-2
{
  "settings": {
    "index": {
      "number_of_shards": 2,  
      "number_of_replicas": 1
    }
  }
}
PUT /ind-3
{
  "settings": {
    "index": {
      "number_of_shards": 4,  
      "number_of_replicas": 2
    }
  }
}
```
Список индексов. `GET _cat/indices/ind-*?v=true&s=index `:
```
health status index uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   ind-1 w63hG2hXSheG51RtrToREA   1   0          0            0       226b           226b
yellow open   ind-2 bdl88b7GRj-OnL0DxigmIA   2   1          0            0       452b           452b
yellow open   ind-3 8dMn8HujRbSl8YdYKmybLQ   4   2          0            0       904b           904b
```
Статус индекса `GET _cluster/health/ind-1`:
```
{
  "cluster_name" : "netology_cluster",
  "status" : "green",
  "timed_out" : false,
  "number_of_nodes" : 1,
  "number_of_data_nodes" : 1,
  "active_primary_shards" : 1,
  "active_shards" : 1,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 0,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 100.0
}
```
Статус индекса `GET _cluster/health/ind-2`:
```
{
  "cluster_name" : "netology_cluster",
  "status" : "yellow",
  "timed_out" : false,
  "number_of_nodes" : 1,
  "number_of_data_nodes" : 1,
  "active_primary_shards" : 2,
  "active_shards" : 2,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 2,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 62.96296296296296
}
```
Статус индекса `GET _cluster/health/ind-3`:
```
{
  "cluster_name" : "netology_cluster",
  "status" : "yellow",
  "timed_out" : false,
  "number_of_nodes" : 1,
  "number_of_data_nodes" : 1,
  "active_primary_shards" : 4,
  "active_shards" : 4,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 8,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 62.96296296296296
}
```
Состояние кластера `GET _cluster/health`:
```
{
  "cluster_name" : "netology_cluster",
  "status" : "yellow",
  "timed_out" : false,
  "number_of_nodes" : 1,
  "number_of_data_nodes" : 1,
  "active_primary_shards" : 17,
  "active_shards" : 17,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 10,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 62.96296296296296
}
```
Кластер в состоянии `yellow`, т.к. часть индексов в состоянии `yellow` - один или несколько шардов в индексе не выделены узлу.

Удаляем индексы `DELETE /ind-*`:
```
{
  "acknowledged" : true
}
```
## Задача 3

В данном задании вы научитесь:
- создавать бэкапы данных
- восстанавливать индексы из бэкапов

Создайте директорию `{путь до корневой директории с elasticsearch в образе}/snapshots`.

Используя API [зарегистрируйте](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-register-repository.html#snapshots-register-repository)
данную директорию как `snapshot repository` c именем `netology_backup`.

**Приведите в ответе** запрос API и результат вызова API для создания репозитория.

Создайте индекс `test` с 0 реплик и 1 шардом и **приведите в ответе** список индексов.

[Создайте `snapshot`](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-take-snapshot.html)
состояния кластера `elasticsearch`.

**Приведите в ответе** список файлов в директории со `snapshot`ами.

Удалите индекс `test` и создайте индекс `test-2`. **Приведите в ответе** список индексов.

[Восстановите](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-restore-snapshot.html) состояние
кластера `elasticsearch` из `snapshot`, созданного ранее.

**Приведите в ответе** запрос к API восстановления и итоговый список индексов.

Подсказки:
- возможно вам понадобится доработать `elasticsearch.yml` в части директивы `path.repo` и перезапустить `elasticsearch`

#### Ответ
Добавлен параметр `path:repo /var/lib/elasticsearch/snapshots` в `elasticsearch.yml`
```
PUT /_snapshot/netology_backup
{
  "type": "fs",
  "settings": {
    "location": "/var/lib/elasticsearch/snapshots"
  }
}
```
Результат `http://localhost:9200/_snapshot/netology_backup?pretty`:
```
{
  "netology_backup" : {
    "type" : "fs",
    "settings" : {
      "location" : "/var/lib/elasticsearch/snapshots"
    }
  }
}
```
Индекс `test` создан:
```
health status index uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   test  W5GX-ofGR8-WCQ_6gRH9GQ   1   0          0            0       226b           226b
```
Snapshot создан командой:
```
PUT _snapshot/netology_backup/elasticsearc?wait_for_completion=true
{
  "indices": "test",
  "include_global_state": false
}
```
Результат `GET _snapshot/netology_backup/elasticsearch`:
```
{
  "snapshot" : {
    "snapshot" : "elasticsearc",
    "uuid" : "mImgMc7aRU-k4CFQCf4L8Q",
    "repository" : "netology_backup",
    "version_id" : 7170699,
    "version" : "7.17.6",
    "indices" : [
      "test"
    ],
    "data_streams" : [ ],
    "include_global_state" : false,
    "state" : "SUCCESS",
    "start_time" : "2022-10-21T14:43:01.420Z",
    "start_time_in_millis" : 1666363381420,
    "end_time" : "2022-10-21T14:43:01.420Z",
    "end_time_in_millis" : 1666363381420,
    "duration_in_millis" : 0,
    "failures" : [ ],
    "shards" : {
      "total" : 1,
      "failed" : 0,
      "successful" : 1
    },
    "feature_states" : [ ]
  }
}
```
Добавлен индекс `test-2`, удален 'test':
```
health status index  uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   test-2 hfmzsepkRiuqf9Lbr3YHjw   1   0          0            0       226b           226b
```
Бэкап восстановлен `POST _snapshot/netology_backup/elasticsearc/_restore?pretty`:
```
{
  "accepted" : true
}
```
Итоговый список индексов:
```
health status index  uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   test-2 hfmzsepkRiuqf9Lbr3YHjw   1   0          0            0       226b           226b
green  open   test   ipTWf6TES_KOCiX8TzXtFw   1   0          0            0       226b           226b
```

---

### Как cдавать задание

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---
