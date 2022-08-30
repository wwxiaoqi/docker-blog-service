# Docker-Blog-Service

Hugo + AliDNS 的 Dockerfile，自动通过阿里云申请 SSL 证书。

本项目基于 [docker-caddy2-hugo-alidns](https://github.com/LadderOperator/docker-caddy2-hugo-alidns) 修改而成，致谢！

## 使用方式

GitHub：
```shell
git clone https://github.com/wwxiaoqi/docker-blog-service
```
配置 `config.json` 文件 (需配置项已高亮)
```diff
{
    "apps": {
        "tls": {
            "certificates": {
                "automate": [
+                   "xxx.com"
                ]
            },
            "automation": {
                "policies": [
                    {
                        "issuers": [
                            {
                                "module": "acme",
                                "challenges": {
                                    "dns": {
                                        "provider": {
                                            "name": "alidns",
+                                           "access_key_id": "alidns id",
+                                           "access_key_secret": "alidns secret"
                                        }
                                    }
                                }
                            }
                        ]
                    }
                ]
            }
        },
        "http": {
            "servers": {
                "caddy-hugo": {
                    "listen": [
                        ":443"
                    ],
                    "routes": [
                        {
                            "match": [
                                {
                                    "host": [
+                                       "xxx.com"
                                    ]
                                }
                            ],
                            "handle": [
                                {
                                    "handler": "file_server",
                                    "root": "/public"
                                }
                            ]
                        }
                    ]
                }
            }
        }
    }
}
```
配置好 `config.json` 文件之后
```shell
docker-compose up -d
```


然后使用同步工具将 `hugo` 生成的 `file` 同步至 `/public` 里面即可。

在这里我使用 Github Actions 同步，例：

```yml
name: 🚀 Deploy Website on Push

# 在 main 分支更新时触发构建
on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest
    env:
      TZ: Asia/Shanghai

    steps:
      - name: 🚚 Get latest code
        uses: actions/checkout@v2
        with:
          submodules: 'recursive'

        ## 省略 Hugo Build 相关

      - uses: webfactory/ssh-agent@v0.5.4
        with:
          ssh-private-key: |
            ${{ secrets.BLOG_DEPLOY_KEY }}

      - name: 🎈Deploy
        run: |
          rsync -av -e"ssh -i $SSH_AUTH_SOCK" --delete site/public root@xxx.com:/mnt/docker-blog-service
```

享受！