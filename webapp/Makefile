export GO111MODULE=on
DB_HOST:=127.0.0.1
DB_PORT:=3306
DB_USER:=
DB_PASS:=
DB_NAME:=

MYSQL_CMD:=mysql -h$(DB_HOST) -P$(DB_PORT) -u$(DB_USER) -p$(DB_PASS) $(DB_NAME)

NGX_LOG:=/tmp/access.log
MYSQL_LOG:=/tmp/slow-query.log

PROJECT_ROOT:= # /home/isucon/isucari
BUILD_DIR:= # /home/isucon/isucari/webapp/go
BIN_NAME:= # isucari
BENCH=#./bin/benchmarker

CA:=-o /dev/null -s -w "%{http_code}\n"

.PHONY: bench
bench: commit before slow-on ${BENCH}

.PHONY: commit
commit:
	cd $(PROJECT_ROOT); \
	git add .; \
	git commit --allow-empty -m "bench"

.PHONY: alp
anal: # ここにalpのコマンドを書く

.PHONY: before
before:
	$(eval when := $(shell date "+%s"))
	mkdir -p ~/logs/$(when)
	@if [ -f $(NGX_LOG) ]; then \
		sudo mv -f $(NGX_LOG) ~/logs/$(when)/ ; \
	fi
	# @if [ -f $(MYSQL_LOG) ]; then \
	# 	sudo mv -f $(MYSQL_LOG) ~/logs/$(when)/ ; \
	# fi
	sudo systemctl restart nginx
	sudo systemctl restart mysql

.PHONY: slow
slow: 
	sudo pt-query-digest $(MYSQL_LOG)

PPROF:=go tool pprof http://localhost:6060/debug/pprof/profile

.PHONY: pprof
pprof:
	$(PPROF)

.PHONY: slow-on
slow-on:
	sudo mysql -e "set global slow_query_log_file = '$(MYSQL_LOG)'; set global long_query_time = 0; set global slow_query_log = ON;"
	# sudo $(MYSQL_CMD) -e "set global slow_query_log_file = '$(MYSQL_LOG)'; set global long_query_time = 0; set global slow_query_log = ON;"

.PHONY: slow-off
slow-off:
	sudo mysql -e "set global slow_query_log = OFF;"
	# sudo $(MYSQL_CMD) -e "set global slow_query_log = OFF;"

.PHONY: setup
setup:
	echo 'install alp'
	wget -O alp.zip https://github.com/tkuchiki/alp/releases/download/v1.0.6/alp_linux_amd64.zip
	unzip alp.zip
	install alp /usr/local/bin/alp
	rm -rf alp
	echo 'install pt-query-digest'
	wget https://github.com/percona/percona-toolkit/archive/3.0.5-test.tar.gz
	tar zxvf 3.0.5-test.tar.gz
	./percona-toolkit-3.0.5-test/bin/pt-query-digest --version
	sudo mv ./percona-toolkit-3.0.5-test/bin/pt-query-digest /usr/local/bin/pt-query-digest
	echo 'install pprof'
	go get -u github.com/google/pprof
