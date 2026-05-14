# Pipeline 项目

以下示例适用于在 `Pipeline` 中调用 `wechatWork` 步骤发送消息。

`robot` 表示机器人 ID，可在 `Lark Notice` 机器人配置中查看。

## 1. 文本消息

用于发送纯文本通知，并支持通过 `ats` 指定提醒对象。

```groovy
pipeline {
    agent any
    stages {
        stage('text') {
            steps {
                echo '发送企业微信文本消息...'
            }
            post {
                success {
                    wechatWork(
                        robot: 'robot-wecom',
                        type: 'TEXT',
                        text: [
                            '构建完成通知',
                            '请相关同学关注 Jenkins 结果'
                        ],
                        ats: [
                            'zhangsan',
                            '13800138000'
                        ]
                    )
                }
            }
        }
    }
}
```

## 2. Markdown 消息

企业微信推荐使用 `MARKDOWN` 展示结构化构建信息。

```groovy
pipeline {
    agent any
    stages {
        stage('markdown') {
            steps {
                echo '发送企业微信 Markdown 消息...'
            }
            post {
                success {
                    wechatWork(
                        robot: 'robot-wecom',
                        type: 'MARKDOWN',
                        title: 'Jenkins 构建通知',
                        text: [
                            ">**任务名称**: [${JOB_NAME}](${JOB_URL})",
                            ">**任务编号**: [${BUILD_DISPLAY_NAME}](${BUILD_URL})",
                            ">**构建状态**: <font color=\"info\">${currentBuild.currentResult}</font>",
                            ">**构建用时**: ${currentBuild.duration} ms",
                            ">**执行人**: ${env.BUILD_USER}"
                        ],
                        ats: [
                            'zhangsan'
                        ]
                    )
                }
            }
        }
    }
}
```

## 3. 卡片消息

企业微信的 `CARD` 会发送为 `template_card.news_notice` 模板卡片，适合构建汇总通知。

```groovy
pipeline {
    agent any
    stages {
        stage('card') {
            steps {
                echo '发送企业微信卡片消息...'
            }
            post {
                success {
                    wechatWork(
                        robot: 'robot-wecom',
                        type: 'CARD',
                        title: 'Jenkins 构建通知',
                        text: [
                            '本次构建执行完成，请查看变更记录和控制台日志。'
                        ],
                        messageUrl: "${BUILD_URL}",
                        buttons: [
                            [
                                title: '更改记录',
                                url: "${BUILD_URL}changes"
                            ],
                            [
                                title: '控制台',
                                url: "${BUILD_URL}console"
                            ]
                        ],
                        picUrl: 'https://www.jenkins.io/images/logos/jenkins/jenkins.png'
                    )
                }
            }
        }
    }
}
```

## 4. 企业微信平台限制说明

- `wechatWork` 步骤主要适用于 `TEXT`、`MARKDOWN`、`CARD`。
- 如果传入 `LINK` 或 `POST`，插件会回退为 `MARKDOWN` 发送。
- `IMAGE` 不是企业微信独立支持的消息类型，直接按图片消息发送会失败。
- 不支持飞书那种 `SHARE_CHAT` 群名片消息。
- 卡片最多显示 3 个跳转按钮。
- `picUrl` 或 `topImg.imgKey` 只有在值为可访问的 `http/https` 图片地址时，才会作为卡片图片生效。
- 如果未配置按钮，插件会默认补充 `更改记录` 和 `控制台` 按钮。
