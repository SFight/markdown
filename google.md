# 特别注意：

1. **./moli-with-gbm-0da9b4410d62.json文件和./oauth2l放在同一目录下；**

2. **./moli-with-gbm-0da9b4410d62.json该文件为创建账户密钥时下载的json文件，务必保存，不然后边无法操作；**

3. **oauth2l 下载地址：[https://github.com/google/oauth2l](https://github.com/google/oauth2l)**

4. **如果遇到401OAuth鉴权失败问题，执行以下代码：**

   ```shell
   ./oauth2l reset
   ```

# 创建Brands

```shell
curl -X POST "https://businesscommunications.googleapis.com/v1/brands" \
-H "Content-Type: application/json" \
-H "$(./oauth2l header --json ./moli-with-gbm-0da9b4410d62.json businesscommunications)" \
-d "{
    'displayName': 'Motorola'
}"
```

# 获取Brands

```shell
curl -X GET "https://businesscommunications.googleapis.com/v1/brands/e3b45830-b195-4e53-be54-bac2cbba5abe" \
-H "Content-Type: application/json" \
-H "$(./oauth2l header --json  ./moli-with-gbm-0da9b4410d62.json businesscommunications)"
```

# 创建Agent

```shell
curl -X POST "https://businesscommunications.googleapis.com/v1/brands/e3b45830-b195-4e53-be54-bac2cbba5abe/agents" \
-H "Content-Type: application/json" \
-H "$(./oauth2l header --json ./moli-with-gbm-0da9b4410d62.json businesscommunications)" \
-d "{
    'displayName': 'Moli',
    'businessMessagesAgent': {
        'logoUrl': 'https://moli.lenovo.com/pcf/adapter/google-moli.png',
        'customAgentId': 'motorola-ikuwr0w0',
        'conversationalSettings': {
            'en': {
                'welcomeMessage': {
                    'text': 'Hello there. Welcome to Motorola Support. I am Moli, our virtual agent.How can I help you today?',
                },
                'privacyPolicy': {
                    'url': 'https://www.lenovo.com/us/en/privacy/',
                },
            },
        },
        'primaryAgentInteraction': {
            'interactionType': 'BOT',
            'botRepresentative': {
                'botMessagingAvailability': {
                    'hours': [
                        {
                            'startTime': {
                                'hours': 0,
                                'minutes': 0
                            },
                            'endTime': {
                                'hours': 23,
                                'minutes': 59
                            },
                            'timeZone': 'America/Los_Angeles',
                            'startDay': 'MONDAY',
                            'endDay': 'SUNDAY',
                        },
                    ],
                },
            },
        },
    },
}"
```

# 获取Agent

```shell
curl -X GET "https://businesscommunications.googleapis.com/v1/brands/e3b45830-b195-4e53-be54-bac2cbba5abe/agents" \
-H "Content-Type: application/json" \
-H "$(./oauth2l header --json  ./moli-with-gbm-0da9b4410d62.json businesscommunications)"
```

# 获取签名

```shell
curl -X POST \
"https://businesscommunications.googleapis.com/v1/brands/e3b45830-b195-4e53-be54-bac2cbba5abe/agents/c22f26d6-555a-4704-be87-56c1ab0b7aea:requestVerification" \
-H "Content-Type: application/json" \
-H "$(./oauth2l header --json ./moli-with-gbm-0da9b4410d62.json businesscommunications)" \
-d "{
    'agentVerificationContact': {
        'partnerName': 'Motorola',
        'partnerEmailAddress': 'suld1@lenovo.com',
        'brandContactName': 'Motorola',
        'brandContactEmailAddress': 'suld1@lenovo.com',
        'brandWebsiteUrl': 'https://www.motorola.com/us/',
    },
}"
```

# 验证签名

```shell
curl \
"https://businesscommunications.googleapis.com/v1/brands/e3b45830-b195-4e53-be54-bac2cbba5abe/agents/c22f26d6-555a-4704-be87-56c1ab0b7aea/verification" \
-H "Content-Type: application/json" \
-H "$(./oauth2l header --json  ./moli-with-gbm-0da9b4410d62.json businesscommunications)"
```
