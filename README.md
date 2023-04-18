# Mariadb Galera Cluster

### 1. Create .env from tempage and edit .env
```bash
cp .env.default .env
```
```bash
nano .env
```

### 2. Generate self signed certificate for mariadb
```bash
mkdir -p /apps/certs/mariadb/standalone && cd /apps/certs/mariadb/standalone
```

`--- CA Certificate ---`
```bash
openssl genrsa 2048 > ca-key.pem
```
```bash
openssl req -new -x509 -nodes -days 36500 \
-key ca-key.pem -out ca-cert.pem
```
`--- Server Certificate ---`
```bash
openssl req -newkey rsa:2048 -days 36500 \
-nodes -keyout server-key.pem -out server-req.pem
```
```bash
openssl rsa -in server-key.pem -out server-key.pem
```
```bash
openssl x509 -req -in server-req.pem -days 36500 \
-CA ca-cert.pem -CAkey ca-key.pem -set_serial 01 \
-out server-cert.pem
```
`--- Client Certificate ---`
```bash
openssl req -newkey rsa:2048 -days 36500 \
-nodes -keyout client-key.pem -out client-req.pem
```
```bash
openssl rsa -in client-key.pem -out client-key.pem
```
```bash
openssl x509 -req -in client-req.pem -days 36500 \
-CA ca-cert.pem -CAkey ca-key.pem -set_serial 01 \
-out client-cert.pem
```
`--- Verifying the Certificates ---`
```
openssl verify -CAfile ca-cert.pem \
server-cert.pem client-cert.pem
```

### 3. Change owner certificate
```bash
chown -R 1001:1001 *
```

### 4. Change owner data and backup dir
```
cd -
chown 1001:1001 data backup
```

### 5. Copy my_custom.cnf (Optional) (enable ssl)
```bash
cp example/conf/my_custom.cnf conf
```

### 6. Run Docker
```bash
docker-compose up -d
```

### 7. Enable ssl for user root
```
docker exec -it mariadb-standalone bash
mysql -u root -p$MARIADB_ROOT_PASSWORD -e "SHOW VARIABLES LIKE '%ssl%'";
mysql -u root -p$MARIADB_ROOT_PASSWORD -e "ALTER USER 'root'@'%' REQUIRE SSL; ";
mysql -u root -p$MARIADB_ROOT_PASSWORD -e "SELECT host, user, ssl_type FROM mysql.user";
mysql -u root -p$MARIADB_ROOT_PASSWORD -e "SHOW CREATE USER 'root'";
mysql -u root -p$MARIADB_ROOT_PASSWORD -e "status"|grep -i 'cipher'
exit
```
### 8. Test Exec
```
docker exec -it mariadb-standalone bash
mysql -u root -p$MARIADB_ROOT_PASSWORD
show databases;
quit;
exit
```