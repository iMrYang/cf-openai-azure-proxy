# cf-openai-azure-proxy

## 说明

用于Azure ChatGPT转为OpenAI ChatGPT提供标准API接口

[原仓库](https://github.com/haibbo/cf-openai-azure-proxy)

[Docker镜像](https://hub.docker.com/r/imryang/cf-openai-azure-proxy)

## 新增

+ 新增0301、0613、text-embedding-ada-002模型的映射

## 配置

### 资源名

> 资源名作为环境变量RESOURCE_NAME的值

![资源名](https://github.com/iMrYang/cf-openai-azure-proxy/assets/35570845/148e2671-db4e-4527-b252-8d3d9ec31b24)

### 密钥

> 密钥作为API的KEY（类似标准OpenAI形如sk-XXX的KEY）

下图是Azure创建资源时设置的资源名，以及随机产生的密钥。

![资源名和密钥](https://github.com/iMrYang/cf-openai-azure-proxy/assets/35570845/1e168868-89d7-4c82-9d19-cca82f028942)

### 模型部署名

> 部署名分别作为环境变量DEPLOY_NAME_GPT35、DEPLOY_NAME_GPT35_16K、DEPLOY_NAME_GPT4、DEPLOY_NAME_GPT4_32K、DEPLOY_TEXT_EMBEDDING_ADA_002的值

下图是点击【新建部署】时自定义的部署名（我直接复制的模型名称）

![模型部署名](https://github.com/iMrYang/cf-openai-azure-proxy/assets/35570845/419fecb0-ff8d-4037-ae9a-2b549d2a8b67)

## 启动

### Cloudflare Worker启动（无需服务器）

> 注意本仓库新增了DEPLOY_TEXT_EMBEDDING_ADA_002模型的环境变量

[见原仓库说明](https://github.com/haibbo/cf-openai-azure-proxy#%E4%BD%BF%E7%94%A8%E8%AF%B4%E6%98%8E)

### Docker启动

```sh
docker run -d -p 8787:8787 -t cf-azure-openai-proxy \
  --env RESOURCE_NAME=[your azure resource name] \
  --env DEPLOY_NAME_GPT35=gpt-35-turbo \
  --env DEPLOY_NAME_GPT35_16K=gpt-35-turbo-16k \
  --env DEPLOY_NAME_GPT4=gpt-4 \
  --env DEPLOY_NAME_GPT4_32K=gpt-4-32k \
  --env DEPLOY_TEXT_EMBEDDING_ADA_002=text-embedding-ada-002 \
  imryang/cf-openai-azure-proxy:latest
```

### Docker Compose启动

```
version: '3'
services:
  cf-azure-openai-proxy:
    image: imryang/cf-openai-azure-proxy:latest
    ports:
      - 8787:8787
    environment:
      - RESOURCE_NAME=[your azure resource name]
      - DEPLOY_NAME_GPT35=gpt-35-turbo
      - DEPLOY_NAME_GPT35_16K=gpt-35-turbo-16k
      - DEPLOY_NAME_GPT4=gpt-4
      - DEPLOY_NAME_GPT4_32K=gpt-4-32k
      - DEPLOY_TEXT_EMBEDDING_ADA_002=text-embedding-ada-002
```

### Docker Compose搭配ChatGPT Next Web启动

```
version: '3'
services:
  # Azure OpenAI Proxy
  cf-azure-openai-proxy:
    image: imryang/cf-openai-azure-proxy:latest
    container_name: azuregpt
    restart: always
    environment:
      - RESOURCE_NAME=[your azure resource name]
      - DEPLOY_NAME_GPT35=gpt-35-turbo
      - DEPLOY_NAME_GPT35_16K=gpt-35-turbo-16k
      - DEPLOY_NAME_GPT4=gpt-4
      - DEPLOY_NAME_GPT4_32K=gpt-4-32k
      - DEPLOY_TEXT_EMBEDDING_ADA_002=text-embedding-ada-002
    networks:
      - chatgpt-ns
  # ChatGPT Next Web
  chatgpt:
    image: yidadaa/chatgpt-next-web:latest
    container_name: chatgpt
    restart: always
    environment:
      - BASE_URL=http://cf-azure-openai-proxy:8787
      - OPENAI_API_KEY=[your azure key]
      - CODE=[your code]
      - HIDE_USER_API_KEY=1
      - HIDE_BALANCE_QUERY=1
    networks:
      - chatgpt-ns
    ports:
      - "3000:3000"
    depends_on:
      - cf-azure-openai-proxy
    links:
      - cf-azure-openai-proxy
# 组网
networks:
  chatgpt-ns:
    driver: bridge
```

