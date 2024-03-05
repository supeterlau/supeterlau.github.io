---
title: "Missing Steps of Heroku"
date: 2019-09-17T15:19:17+08:00
draft: False
---

### Docker

根据 Dockerfile 构建镜像：

`docker build . -t <your_image_name>`

`heroku container:push <your_image_name>`

`heroku container:release <your_image_name>`

`heroku open`

Got Error: Expected response to be successful, got 422
Name is invalid

`HEROKU_DEBUG=1 heroku container:release <your_organization>/<your_image_name>:<your_image_tag>`

https://stackoverflow.com/questions/57846015/heroku-name-is-invalid-when-deploy-docker-image-how-to-debug

`heroku open`

Real-time tail

`heroku logs --tail`

https://stackoverflow.com/questions/41804507/h14-error-in-heroku-no-web-processes-running

原因：没有启动 dyno Heroku 最小计算单元

如果使用 Procfile 可以用命令

heroku ps:scale web=1

对应 Procfile

web: gunicorn -w 4 -b 0.0.0.0:$PORT -k gevent main:app

没有用 Procfile 可以去应用 Resources 下

https://dashboard.heroku.com/apps/pure-fjord-97245/resources

修改 dyno 状态

heroku login -i 保持在终端下登录

heroku apps

docker-compose build

Your Ruby version is 2.5.6, but your Gemfile specified 2.5.3

Dockerfile
FROM ruby:2.5 -> FROM ruby:2.5.3

docker-compose run --rm web rails db:create

Deployment to Heroku
$ heroku update beta  
$ heroku plugins:install @heroku-cli/plugin-manifest
$ heroku apps:create --manifest
$ heroku addons:create heroku-postgresql:hobby-dev


heroku addons:create heroku-postgresql:hobby-dev -a bebopx_hello-heroku.1

heroku addons:create heroku-postgresql:hobby-dev -a pure-fjord-97245

$ heroku config:set RAILS_MASTER_KEY=`cat config/master.key`
$ heroku config:set RACK_ENV=production RAILS_ENV=production
$ heroku config:set RAILS_SERVE_STATIC_FILES=enabled
$ heroku config:set WEB_CONCURRENCY=2
$ heroku config:set LANG=en_US.UTF-8

https://blog.teamtreehouse.com/deploy-static-site-heroku