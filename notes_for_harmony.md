# Harmony Notes
## Module的几种形态
- Ability类型的Module，编译后生成.hap，分为entry和feature类型，可在设备上独立安装运行；
  - entry类型：一个应用中包含唯一一个entry类型的hap包，也可以不包含；
  - feature类型：一个应用中包含一个或多个feature类型的hap包，也可以不包含；
- Library类型的Module
  - Static Library：编译后生成.har，多个模块各自会拷贝一份，配置文件中不可配置pages，不可在设备上独立安装运行；
  - Shared Library：编译后多模块共用一份拷贝，不可在设备上独立安装运行；
## 开发态和编译态的对应关系
<img width="1142" height="747" alt="image" src="https://github.com/user-attachments/assets/2e201855-810c-4aa8-a4ff-d47112994b6f" />

- resources目录下如果有重名资源，以AppScope下的资源为准

## 编译发布与上架部署流程图
<img width="836" height="835" alt="image" src="https://github.com/user-attachments/assets/14853d5a-d9ef-45a1-8c17-da4233d5fcb3" />



