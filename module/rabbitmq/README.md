# docker 环境 rabbitmq 集群搭建

# 创建DNS
docker create \
  --name dnsmasq \
  -p 53:53/udp \
  -p 5380:8080 \
  -v ~/data/dnsmasq/dnsmasq.conf:/etc/dnsmasq.conf \
  --log-opt "max-size=100m" \
  -e "HTTP_USER=admin" \
  -e "HTTP_PASS=admin" \
  --restart always \
  jpillora/dnsmasq

# 创建第一个节点
sudo docker run -d \
 --hostname rabbitmq1 \
 --name myrabbitmq1 \
 -p 4369:4369 \
 -p 5671:5671 \
 -p 5672:5672 \
 -p 15672:15672 \
 -v ~/data/rabbitmq1:/var/lib/rabbitmq \
 -e RABBITMQ_NODENAME=rabbit \
 -e RABBITMQ_DEFAULT_USER=tratao \
 -e RABBITMQ_DEFAULT_PASS=tratao \
 -e RABBITMQ_ERLANG_COOKIE='trataocookie' \
 rabbitmq:3.7.8-management

# 创建第二个节点
sudo docker run -d \
  --hostname rabbitmq2 \
  --name myrabbitmq2 \
  --dns $(sudo docker inspect -f '{{.NetworkSettings.IPAddress}}' dnsmasq) \
  -p 5673:5672 \
  -p 15673:15672 \
  -v ~/data/rabbitmq2:/var/lib/rabbitmq \
  -e RABBITMQ_NODENAME=rabbit \
  -e RABBITMQ_DEFAULT_USER=tratao \
  -e RABBITMQ_DEFAULT_PASS=tratao \
  -e RABBITMQ_ERLANG_COOKIE='trataocookie' \
  rabbitmq:3.7.8-management

# 创建第三个节点
sudo docker run -d \
  --hostname rabbitmq3 \
  --name myrabbitmq3 \
  --dns $(sudo docker inspect -f '{{.NetworkSettings.IPAddress}}' dnsmasq) \
  -p 5674:5672 \
  -p 15674:15672 \
  -v ~/data/rabbitmq3:/var/lib/rabbitmq \
  -e RABBITMQ_NODENAME=rabbit \
  -e RABBITMQ_DEFAULT_USER=tratao \
  -e RABBITMQ_DEFAULT_PASS=tratao \
  -e RABBITMQ_ERLANG_COOKIE='trataocookie' \
  rabbitmq:3.7.8-management

# 创建第四个节点
sudo docker run -d \
  --hostname rabbitmq4 \
  --name myrabbitmq4 \
  -p 5675:5672 \
  -p 15675:15672 \
  -v ~/data/rabbitmq4:/var/lib/rabbitmq \
  -e RABBITMQ_NODENAME=rabbit \
  -e RABBITMQ_DEFAULT_USER=tratao \
  -e RABBITMQ_DEFAULT_PASS=tratao \
  -e RABBITMQ_ERLANG_COOKIE='trataocookie' \
  rabbitmq:3.7.8-management


#把二、三节点加入到主节点一
rabbitmqctl stop_app
rabbitmqctl join_cluster rabbit@rabbitmq1
rabbitmqctl start_app

# 出现访问不了节点 4369 问题
# 则修改/etc/hosts， 把主节点 IP hostname 添加到子节点

# 镜像队列（添加策略 policy）
# Name: test // 策略名称
# Pattern：^t // 匹配的规则，这里表示匹配开头的队列，如果是匹配所有的队列，那就是^.
# Definition:
# ha-mode=all // 使用ha-mode模式中的all，也就是同步所有匹配的队列。
# ha-sync-mode=automatic  // 这个参数有2中取值:automatic, manual。 镜像队列被设置成automatic,当一个新slave加入时，slave会自动同步master中的所有消息，直到所有消息被同步完成之前，所有的操作都会被阻塞（blocking）
# ha-promote-on-shutdown=always // 有2个取值：when-synced，always。 设置为when-synced(默认)时，通过rabbitmqctl stop命令停止或者优雅关闭OS，那么slave不会接管master，也就是此时镜像队列不可用；但是如果VM或者OS crash了，那么slave会接管master。如果设置为always,那么不论master因为何种原因停止，slave都会接管master，优先保证可用性。

# +显示红色的， 需要在任意节点上执行同步 rabbitmqctl sync_queue test_queue




# test 
sudo docker run -d \
 --hostname rabbitmq1 \
 --name myrabbitmq1 \
 -p 5672:5672 \
 -p 15672:15672 \
 -v ~/data/rabbitmq:/var/lib/rabbitmq \
 -e RABBITMQ_NODENAME=rabbit \
 -e RABBITMQ_DEFAULT_USER=tratao \
 -e RABBITMQ_DEFAULT_PASS=tratao \
 rabbitmq:3.7.8-management
