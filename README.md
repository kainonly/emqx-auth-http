# emqx-auth-http

EMQX HTTP authentication and ACL control service middleware

[![Github Actions](https://img.shields.io/github/workflow/status/kain-lab/emqx-auth-http/release?style=flat-square)](https://github.com/kain-lab/emqx-auth-http/actions)
[![GitHub go.mod Go version](https://img.shields.io/github/go-mod/go-version/kain-lab/emqx-auth-http?style=flat-square)](https://github.com/kain-lab/emqx-auth-http)
[![Image Size](https://img.shields.io/docker/image-size/kainonly/emqx-auth-http?style=flat-square)](https://hub.docker.com/r/kainonly/emqx-auth-http)
[![Docker Pulls](https://img.shields.io/docker/pulls/kainonly/emqx-auth-http.svg?style=flat-square)](https://hub.docker.com/r/kainonly/emqx-auth-http)
[![GitHub license](https://img.shields.io/badge/license-MIT-blue.svg?style=flat-square)](https://raw.githubusercontent.com/kain-lab/emqx-auth-http/master/LICENSE)

## Setup

Example using docker compose

```yaml
version: "3.8"
services: 
  emqx-auth-http:
    image: kainonly/emqx-auth-http
    restart: always
    environment:
      GIN_MODE: release
    volumes:
      - ./emqx/auth:/app/config
    ports:
      - 8080:8080
```

## Configuration

For configuration, please refer to `config/config.example.yml`

- **listen** `string` grpc server listening address
- **redis** `redis.Options` reference https://github.com/go-redis/redis
- **key** reference [#Key](#key)

## Key

### auth `hash`

Like this, hkey is equivalent to the `username` of emqx, and value is the HS256 key of the token generated by the username is equivalent to the `password` of emqx

| hkey             | value            |
| ---------------- | ---------------- |
| zZ11v3G6DrqjeMSo | q!%QlIXvXNpZ1bPe |
| QiqhdD3wKWJE6rgK | XuEUnzEjCTz3*&nE |

### super `set`

For example, `zZ11v3G6DrqjeMSo` is a super user, he will allow to subscribe and publish all topics

| value            |
| ---------------- |
| zZ11v3G6DrqjeMSo |

### acl `set`

Suppose we set `REDIS_KEY_FOR_ACL` to `mqtt-acl`, when the ACL is set for the `QiqhdD3wKWJE6rgK` user, he will generate it, `mqtt-acl:QiqhdD3wKWJE6rgK` collection cache, the collection content is the topic that it is allowed to subscribe to

| value  |
| ------ |
| notice |
| tests  |

## EMQX Configuration

Example using docker compose

```yaml
version: "3.8"
services: 
  emqx:
    image: emqx/emqx
    restart: always
    environment:
      EMQX_NAME: emqx
      EMQX_ALLOW_ANONYMOUS: 'false'
      EMQX_AUTH__HTTP__REQUEST__RETRY_TIMES: 3
      EMQX_AUTH__HTTP__REQUEST__RETRY_INTERVAL: 1s
      EMQX_AUTH__HTTP__REQUEST__RETRY_BACKOFF: 2.0
      EMQX_AUTH__HTTP__AUTH_REQ: http://emqx-auth-http:8080/auth
      EMQX_AUTH__HTTP__AUTH_REQ__METHOD: post
      EMQX_AUTH__HTTP__AUTH_REQ__PARAMS: username=%u,token=%P
      EMQX_AUTH__HTTP__SUPER_REQ: http://emqx-auth-http:8080/super
      EMQX_AUTH__HTTP__SUPER_REQ__PARAMS: username=%u
      EMQX_AUTH__HTTP__SUPER_REQ__METHOD: post
      EMQX_AUTH__HTTP__ACL_REQ: http://emqx-auth-http:8080/acl
      EMQX_AUTH__HTTP__ACL_REQ__METHOD: post
      EMQX_AUTH__HTTP__ACL_REQ__PARAMS: username=%u,topic=%t
      EMQX_LISTENER__TCP__EXTERNAL: 1883
      EMQX_LISTENER__WS__EXTERNAL: 8083
    ports:
      - 1883:1883
```