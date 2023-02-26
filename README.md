# Keycloack cluster

# Зачем это всё
В документации написано, что Keycloak изначально заточен для отказоустойчивых инсталляций. Чтобы соблюдалась High Availability, необходимо хранилище. И таким является Infinispan. Точнее в самом Keycloak встроена реализация распределенного кэша и она построена на базе Infinispan. 

Чтобы инстансы Keycloak могли друг друга увидеть, используется UDP и multicast. Серверы находят друг друга, обмениваются информацией и реплицируют её. Всё это реализовано благодаря JGROUPS. Но в некоторых сетях (особенно в облачных) это не сработает, т.к. мультикаст не разрешён. И для этого у кейклоака всё предусмотрено.

Если инстансы имеют фиксированные адреса, можно использовать другой тип Transport stack - TCP. Там под капотом MPING (uses UDP multicast). Но такой вариант вряд ли подойдёт для контейнеров. Если же Keycloak запущен в Kubernetes, то используется иной транспорт и discovery - TCP + DNS_PING + headless сервис. Также есть специфичные виды транспорта кеша для облачных инстансов в AWS, когда для кеша используется s3 бакет (NATIVE_S3_PING) + GOOGLE_PING2 и AZURE_PING в зависимости от инстанса.

Таким образом, в данном репозитории содержатся примеры запуска нескольких инстансов Keycloak с помощью docker compose и отдельного балансировщика. В качестве транспорта используется udp и мультикаст по умолчанию.

# Запуск

`docker compose up -d`

Будет запущен контейнер с postgresql, nginx и парой keycloak.

Для nginx **важно настроить проксирование всех заголовков** и также указать размер буферов, т.к. с длинными запросами будут проблемы в противном случае.
