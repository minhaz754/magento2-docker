### Magento 2.4.8 Production-Grade Dockerized Setup
This guide provides a full, production-grade, Dockerized Magento 2.4.8 setup using:
•	Magento 2.4.8
•	Nginx 1.28
•	PHP 8.3 (FPM)
•	MySQL 8.0
•	Opensearch
•	Redis 

###  Start containers
```bash
docker compose up -d –build
```
### Opensearch configure
## UNBLOCK INDEX CREATION
```bash
docker compose exec php curl -X PUT http://opensearch:9200/_cluster/settings \
  -H "Content-Type: application/json" \
  -d '{
    "persistent": {
      "cluster.blocks.create_index": false
    }
  }'
```
## Disable Disk Threshold (Persistent)
```bash
docker compose exec php curl -X PUT http://opensearch:9200/_cluster/settings \
  -H "Content-Type: application/json" \
  -d '{
    "persistent": {
      "cluster.routing.allocation.disk.threshold_enabled": false
    }
  }'
```
# Clear ALL Cluster Blocks Explicitly
```bash
docker compose exec php curl -X PUT http://opensearch:9200/_cluster/settings \
  -H "Content-Type: application/json" \
  -d '{
    "persistent": {
      "cluster.blocks.read_only": false,
      "cluster.blocks.read_only_allow_delete": false,
      "cluster.blocks.create_index": false
    }
  }'
  ```
# verify
```bash
docker compose exec php curl -s http://opensearch:9200/_cluster/settings?pretty
```
# Expected
•	❌ no create_index: true
•	❌ no read_only*
# Restart OpenSearch
docker compose restart opensearch.

### Redis Configuration
```bash
docker compose exec php bin/magento setup:config:set \
--cache-backend=redis \
--cache-backend-redis-server=redis \
--cache-backend-redis-db=0
docker compose exec php bin/magento setup:config:set \
--session-save=redis \
--session-save-redis-host=redis \
--session-save-redis-port=6379 \
--session-save-redis-db=1
```

### If needed Sample Data installation then run below
```bash
docker compose exec php bin/magento sampledata:deploy
```
## And add below commands while set production or developer mode
```bash
docker compose exec php bin/magento indexer:reindex
docker compose exec php bin/magento maintenance:disable
```

# Magento Auth Keys Required
You must enter:
•	Username → Public Key
•	Password → Private Key
(from https://marketplace.magento.com → My Profile → Access Keys)


## Magento Setup Install
```bash
docker compose exec php bin/magento setup:install \ 
--base-url=http://magentotest.com/ \
--db-host=mysql \
--db-name=magento \
--db-user=magento \
--db-password=magento \
--admin-firstname=Admin \
--admin-lastname=User \
--admin-email=admin@example.com \
--admin-user=admin \
--admin-password=Strong@12345 \
--language=en_US \
--currency=USD \
--timezone=UTC \
--use-rewrites=1 \
--search-engine=opensearch \
--opensearch-host=opensearch \
--opensearch-port=9200 \
--disable-modules=Magento_TwoFactorAuth
```
# Expected
[SUCCESS]: Magento installation complete.

## Check Magento status
```bash
docker compose exec php bin/magento module:status
```
# You should see no missing modules.

## deploy mode set and Optimizations
```bash
docker compose exec php bin/magento deploy:mode:set developer
docker compose exec php bin/magento setup:di:compile
docker compose exec php bin/magento setup:static-content:deploy -f
docker compose exec php bin/magento cache:flush
```
# Fix Ownership and permissions
```bash
docker compose exec php chown -R www-data:www-data var generated pub/static pub/media app/etc
docker compose exec php chmod +x bin/magento
docker compose exec php find var generated pub/static pub/media app/etc -type d -exec chmod 775 {} \;
docker compose exec php find var generated pub/static pub/media app/etc -type f -exec chmod 664 {} \;
```
# Another way to #Fix Ownership and permissions
```bash
docker compose exec php chown -R www-data:www-data .
docker compose exec php chmod +x bin/magento
docker compose exec php find var generated vendor pub/static pub/media app/etc -type f -exec chmod g+w {} +
docker compose exec php find var generated vendor pub/static pub/media app/etc -type d -exec chmod g+ws {} +
```
## Find magento Admin URL
```bash
docker compose exec php bin/magento info:adminuri
```
# Output like
Admin URI: /admin_l2mq9ia
## Checks	
Storefront: http://magentotest.com
Admin url: http://magentotest.com/admin_l2mq9ia

•	Admin: http://magentotest/admin_l2mq9ian_0t##Checks
