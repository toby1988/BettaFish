# MindSpider AI爬虫系统

<cite>
**本文档引用的文件**
- [MindSpider/README.md](file://MindSpider/README.md)
- [MindSpider/main.py](file://MindSpider/main.py)
- [MindSpider/config.py](file://MindSpider/config.py)
- [MindSpider/requirements.txt](file://MindSpider/requirements.txt)
- [MindSpider/BroadTopicExtraction/main.py](file://MindSpider/BroadTopicExtraction/main.py)
- [MindSpider/BroadTopicExtraction/topic_extractor.py](file://MindSpider/BroadTopicExtraction/topic_extractor.py)
- [MindSpider/BroadTopicExtraction/get_today_news.py](file://MindSpider/BroadTopicExtraction/get_today_news.py)
- [MindSpider/BroadTopicExtraction/database_manager.py](file://MindSpider/BroadTopicExtraction/database_manager.py)
- [MindSpider/DeepSentimentCrawling/main.py](file://MindSpider/DeepSentimentCrawling/main.py)
- [MindSpider/DeepSentimentCrawling/keyword_manager.py](file://MindSpider/DeepSentimentCrawling/keyword_manager.py)
- [MindSpider/DeepSentimentCrawling/platform_crawler.py](file://MindSpider/DeepSentimentCrawling/platform_crawler.py)
- [MindSpider/schema/db_manager.py](file://MindSpider/schema/db_manager.py)
- [MindSpider/schema/mindspider_tables.sql](file://MindSpider/schema/mindspider_tables.sql)
- [MindSpider/schema/init_database.py](file://MindSpider/schema/init_database.py)
- [MindSpider/schema/models_sa.py](file://MindSpider/schema/models_sa.py)
- [MindSpider/schema/models_bigdata.py](file://MindSpider/schema/models_bigdata.py)
- [SentimentAnalysisModel/WeiboSentiment_Finetuned/GPT2-Lora/predict.py](file://SentimentAnalysisModel/WeiboSentiment_Finetuned/GPT2-Lora/predict.py)
</cite>

## 目录
1. [简介](#简介)
2. [项目结构](#项目结构)
3. [核心组件](#核心组件)
4. [架构概览](#架构概览)
5. [详细组件分析](#详细组件分析)
6. [依赖关系分析](#依赖关系分析)
7. [性能考虑](#性能考虑)
8. [故障排除指南](#故障排除指南)
9. [结论](#结论)
10. [附录](#附录)

## 简介

MindSpider AI爬虫系统是一个基于Agent技术的智能舆情爬虫系统，通过AI自动识别热点话题，并在多个社交媒体平台进行精准的内容爬取。系统采用模块化设计，能够实现从话题发现到内容收集的全自动化流程。

该系统借鉴了GitHub知名爬虫项目[MediaCrawler](https://github.com/NanmiCoder/MediaCrawler)的设计理念，实现了两步走爬取策略：

- **模块一**：Search Agent从包括微博、知乎、GitHub、酷安等13个社媒平台、技术论坛识别热点新闻，并维护一个每日话题分析表
- **模块二**：全平台爬虫深度爬取每个话题的细粒度舆情反馈

## 项目结构

MindSpider项目采用清晰的模块化架构，主要包含以下核心模块：

```mermaid
graph TB
subgraph "MindSpider核心模块"
MS[main.py<br/>系统主入口]
CFG[config.py<br/>配置管理]
REQ[requirements.txt<br/>依赖管理]
end
subgraph "BroadTopicExtraction<br/>话题提取模块"
BTE_MAIN[main.py<br/>模块主入口]
BTE_NEWS[get_today_news.py<br/>新闻采集器]
BTE_TOPIC[topic_extractor.py<br/>AI话题提取器]
BTE_DB[database_manager.py<br/>数据库管理器]
end
subgraph "DeepSentimentCrawling<br/>深度爬取模块"
DSC_MAIN[main.py<br/>模块主入口]
DSC_KEYWORD[keyword_manager.py<br/>关键词管理器]
DSC_PLATFORM[platform_crawler.py<br/>平台爬虫管理器]
DSC_MEDIA[MediaCrawler/<br/>多平台爬虫核心]
end
subgraph "Schema<br/>数据库架构"
SCHEMA_DB[db_manager.py<br/>数据库管理工具]
SCHEMA_SQL[mindspider_tables.sql<br/>表结构定义]
SCHEMA_INIT[init_database.py<br/>初始化脚本]
SCHEMA_MODELS[models_sa.py & models_bigdata.py<br/>数据模型]
end
subgraph "SentimentAnalysisModel<br/>情感分析模型"
SENT_GPT2[predict.py<br/>GPT2 LoRA情感分析]
end
MS --> BTE_MAIN
MS --> DSC_MAIN
BTE_MAIN --> BTE_NEWS
BTE_MAIN --> BTE_TOPIC
BTE_MAIN --> BTE_DB
DSC_MAIN --> DSC_KEYWORD
DSC_MAIN --> DSC_PLATFORM
DSC_PLATFORM --> DSC_MEDIA
SCHEMA_DB --> SCHEMA_SQL
SCHEMA_INIT --> SCHEMA_MODELS
DSC_MAIN --> SCHEMA_DB
BTE_MAIN --> SCHEMA_DB
SENT_GPT2 --> DSC_MAIN
```

**图表来源**
- [MindSpider/main.py](file://MindSpider/main.py#L1-L520)
- [MindSpider/BroadTopicExtraction/main.py](file://MindSpider/BroadTopicExtraction/main.py#L1-L326)
- [MindSpider/DeepSentimentCrawling/main.py](file://MindSpider/DeepSentimentCrawling/main.py#L1-L282)

**章节来源**
- [MindSpider/README.md](file://MindSpider/README.md#L31-L71)
- [MindSpider/main.py](file://MindSpider/main.py#L34-L46)

## 核心组件

### 系统配置管理

MindSpider采用统一的配置管理系统，支持MySQL和PostgreSQL两种数据库类型：

```mermaid
classDiagram
class Settings {
+string DB_DIALECT
+string DB_HOST
+int DB_PORT
+string DB_USER
+string DB_PASSWORD
+string DB_NAME
+string DB_CHARSET
+string MINDSPIDER_API_KEY
+string MINDSPIDER_BASE_URL
+string MINDSPIDER_MODEL_NAME
}
class MindSpider {
+Path project_root
+Path broad_topic_path
+Path deep_sentiment_path
+bool check_config()
+bool check_database_connection()
+bool check_database_tables()
+bool initialize_database()
+bool run_broad_topic_extraction()
+bool run_deep_sentiment_crawling()
+bool run_complete_workflow()
}
Settings --> MindSpider : "提供配置"
```

**图表来源**
- [MindSpider/config.py](file://MindSpider/config.py#L16-L35)
- [MindSpider/main.py](file://MindSpider/main.py#L34-L181)

### 数据库架构设计

系统采用扩展的MediaCrawler表结构，添加了BroadTopicExtraction模块所需的表：

```mermaid
erDiagram
DAILY_NEWS {
int id PK
varchar news_id UK
varchar source_platform
varchar title
varchar url
text description
text extra_info
date crawl_date
int rank_position
bigint add_ts
bigint last_modify_ts
}
DAILY_TOPICS {
int id PK
varchar topic_id UK
varchar topic_name
text topic_description
text keywords
date extract_date
float relevance_score
int news_count
varchar processing_status
bigint add_ts
bigint last_modify_ts
}
TOPIC_NEWS_RELATION {
int id PK
varchar topic_id FK
varchar news_id FK
float relation_score
date extract_date
bigint add_ts
}
CRAWLING_TASKS {
int id PK
varchar task_id UK
varchar topic_id FK
varchar platform
text search_keywords
varchar task_status
bigint start_time
bigint end_time
int total_crawled
int success_count
int error_count
text error_message
text config_params
date scheduled_date
bigint add_ts
bigint last_modify_ts
}
DAILY_NEWS ||--o{ TOPIC_NEWS_RELATION : "关联"
DAILY_TOPICS ||--o{ TOPIC_NEWS_RELATION : "关联"
DAILY_TOPICS ||--o{ CRAWLING_TASKS : "触发"
```

**图表来源**
- [MindSpider/schema/mindspider_tables.sql](file://MindSpider/schema/mindspider_tables.sql#L12-L106)

**章节来源**
- [MindSpider/schema/mindspider_tables.sql](file://MindSpider/schema/mindspider_tables.sql#L1-L202)
- [MindSpider/schema/db_manager.py](file://MindSpider/schema/db_manager.py#L30-L98)

## 架构概览

MindSpider采用双阶段爬取架构，实现了从热点发现到深度分析的完整流程：

```mermaid
flowchart TB
Start[开始] --> CheckConfig{检查配置}
CheckConfig --> |配置无效| ConfigError[配置错误<br/>请检查.env中的环境变量]
CheckConfig --> |配置有效| InitDB[初始化数据库]
InitDB --> BroadTopic[BroadTopicExtraction<br/>话题提取模块]
BroadTopic --> CollectNews[收集热点新闻]
CollectNews --> NewsSources{新闻源}
NewsSources --> Weibo[微博热搜]
NewsSources --> Zhihu[知乎热榜]
NewsSources --> Bilibili[B站热门]
NewsSources --> Toutiao[今日头条]
NewsSources --> Other[其他平台...]
Weibo --> SaveNews[保存新闻到数据库]
Zhihu --> SaveNews
Bilibili --> SaveNews
Toutiao --> SaveNews
Other --> SaveNews
SaveNews --> ExtractTopic[AI话题提取]
ExtractTopic --> GenerateKeywords[生成关键词列表]
GenerateKeywords --> GenerateSummary[生成新闻摘要]
GenerateSummary --> SaveTopics[保存话题数据]
SaveTopics --> DeepCrawl[DeepSentimentCrawling<br/>深度爬取模块]
DeepCrawl --> LoadKeywords[加载关键词]
LoadKeywords --> PlatformSelect{选择爬取平台}
PlatformSelect --> XHS[小红书爬虫]
PlatformSelect --> DY[抖音爬虫]
PlatformSelect --> KS[快手爬虫]
PlatformSelect --> BILI[B站爬虫]
PlatformSelect --> WB[微博爬虫]
PlatformSelect --> TB[贴吧爬虫]
PlatformSelect --> ZH[知乎爬虫]
XHS --> Login{需要登录?}
DY --> Login
KS --> Login
BILI --> Login
WB --> Login
TB --> Login
ZH --> Login
Login --> |是| QRCode[扫码登录]
Login --> |否| Search[关键词搜索]
QRCode --> Search
Search --> CrawlContent[爬取内容]
CrawlContent --> ParseData[解析数据]
ParseData --> SaveContent[保存到数据库]
SaveContent --> MoreKeywords{还有更多关键词?}
MoreKeywords --> |是| LoadKeywords
MoreKeywords --> |否| GenerateReport[生成爬取报告]
GenerateReport --> End[结束]
style Start fill:#90EE90
style End fill:#FFB6C1
style BroadTopic fill:#87CEEB,stroke:#000,stroke-width:3px
style DeepCrawl fill:#DDA0DD,stroke:#000,stroke-width:3px
style ExtractTopic fill:#FFD700
style ConfigError fill:#FF6347
```

**图表来源**
- [MindSpider/README.md](file://MindSpider/README.md#L77-L145)
- [MindSpider/main.py](file://MindSpider/main.py#L351-L378)

## 详细组件分析

### BroadTopicExtraction模块

BroadTopicExtraction模块负责每日热点话题的自动发现和提取，包含三个核心组件：

#### 新闻采集器 (NewsCollector)

新闻采集器实现了多平台新闻源的统一采集机制：

```mermaid
sequenceDiagram
participant Client as "外部调用"
participant Collector as "NewsCollector"
participant API as "新闻API"
participant DB as "DatabaseManager"
Client->>Collector : collect_and_save_news(sources)
Collector->>Collector : get_popular_news(sources)
loop 遍历每个新闻源
Collector->>API : fetch_news(source)
API-->>Collector : 返回新闻数据
Collector->>Collector : _process_news_results()
end
Collector->>DB : save_daily_news(news_list, date)
DB-->>Collector : 保存结果
Collector-->>Client : 返回处理结果
```

**图表来源**
- [MindSpider/BroadTopicExtraction/get_today_news.py](file://MindSpider/BroadTopicExtraction/get_today_news.py#L154-L207)
- [MindSpider/BroadTopicExtraction/database_manager.py](file://MindSpider/BroadTopicExtraction/database_manager.py#L75-L141)

#### AI话题提取器 (TopicExtractor)

AI话题提取器基于DeepSeek API实现智能关键词提取和新闻总结：

```mermaid
classDiagram
class TopicExtractor {
-OpenAI client
-string model
+extract_keywords_and_summary(news_list, max_keywords) Tuple
-build_news_summary(news_list) string
-build_analysis_prompt(news_text, max_keywords) string
-parse_analysis_result(result_text) Tuple
-manual_parse_result(text) Tuple
-extract_simple_keywords(news_list) string[]
+get_search_keywords(keywords, limit) string[]
}
class OpenAI {
+chat.completions.create() ChatCompletion
}
TopicExtractor --> OpenAI : "使用API"
```

**图表来源**
- [MindSpider/BroadTopicExtraction/topic_extractor.py](file://MindSpider/BroadTopicExtraction/topic_extractor.py#L25-L81)

#### 数据库管理器 (DatabaseManager)

数据库管理器提供了完整的数据持久化解决方案：

```mermaid
flowchart TD
Start[数据操作开始] --> CheckType{操作类型}
CheckType --> |保存新闻| SaveNews[save_daily_news]
CheckType --> |获取新闻| GetNews[get_daily_news]
CheckType --> |保存话题| SaveTopics[save_daily_topics]
CheckType --> |获取话题| GetTopics[get_daily_topics]
SaveNews --> DeleteOld[删除当天旧数据]
DeleteOld --> InsertLoop[逐条插入新数据]
InsertLoop --> Success[保存成功]
GetNews --> QueryDB[查询数据库]
QueryDB --> ReturnNews[返回新闻列表]
SaveTopics --> CheckExist{检查是否存在}
CheckExist --> |存在| UpdateTopic[更新话题数据]
CheckExist --> |不存在| InsertTopic[插入新话题]
UpdateTopic --> TopicSuccess[话题更新成功]
InsertTopic --> TopicSuccess
GetTopics --> QueryTopic[查询话题数据]
QueryTopic --> ParseJSON[解析JSON关键词]
ParseJSON --> ReturnTopic[返回话题数据]
Success --> End[操作结束]
ReturnNews --> End
TopicSuccess --> End
ReturnTopic --> End
```

**图表来源**
- [MindSpider/BroadTopicExtraction/database_manager.py](file://MindSpider/BroadTopicExtraction/database_manager.py#L75-L214)

**章节来源**
- [MindSpider/BroadTopicExtraction/main.py](file://MindSpider/BroadTopicExtraction/main.py#L29-L233)
- [MindSpider/BroadTopicExtraction/get_today_news.py](file://MindSpider/BroadTopicExtraction/get_today_news.py#L45-L283)
- [MindSpider/BroadTopicExtraction/topic_extractor.py](file://MindSpider/BroadTopicExtraction/topic_extractor.py#L25-L271)
- [MindSpider/BroadTopicExtraction/database_manager.py](file://MindSpider/BroadTopicExtraction/database_manager.py#L29-L323)

### DeepSentimentCrawling模块

DeepSentimentCrawling模块基于提取的话题关键词，在各大社交平台进行深度内容爬取：

#### 关键词管理器 (KeywordManager)

关键词管理器实现了智能的关键词获取和分配机制：

```mermaid
classDiagram
class KeywordManager {
-Engine engine
+connect() void
+get_latest_keywords(target_date, max_keywords) string[]
+get_daily_topics(extract_date) Dict
+get_recent_topics(days) Dict[]
+get_all_keywords_for_platforms(platforms, target_date, max_keywords) string[]
+get_keywords_for_platform(platform, target_date, max_keywords) string[]
-filter_keywords_by_platform(keywords, platform) string[]
+get_crawling_summary(target_date) Dict
+close() void
}
class DatabaseManager {
+get_daily_news(date) Dict[]
+get_daily_topics(date) Dict
+get_recent_topics(days) Dict[]
}
KeywordManager --> DatabaseManager : "查询数据"
```

**图表来源**
- [MindSpider/DeepSentimentCrawling/keyword_manager.py](file://MindSpider/DeepSentimentCrawling/keyword_manager.py#L29-L310)

#### 平台爬虫管理器 (PlatformCrawler)

平台爬虫管理器负责协调各平台的爬取任务：

```mermaid
sequenceDiagram
participant Manager as "PlatformCrawler"
participant DBConfig as "数据库配置"
participant BaseConfig as "基础配置"
participant MediaCrawler as "MediaCrawler"
participant Platform as "目标平台"
Manager->>DBConfig : configure_mediacrawler_db()
DBConfig-->>Manager : 配置完成
loop 遍历每个平台
Manager->>BaseConfig : create_base_config(platform, keywords)
BaseConfig-->>Manager : 配置完成
Manager->>MediaCrawler : subprocess.run(cmd)
MediaCrawler->>Platform : 执行爬取任务
Platform-->>MediaCrawler : 返回爬取结果
MediaCrawler-->>Manager : 返回统计信息
Manager->>Manager : 更新爬取统计
end
Manager-->>Manager : 生成总体统计报告
```

**图表来源**
- [MindSpider/DeepSentimentCrawling/platform_crawler.py](file://MindSpider/DeepSentimentCrawling/platform_crawler.py#L218-L459)

**章节来源**
- [MindSpider/DeepSentimentCrawling/main.py](file://MindSpider/DeepSentimentCrawling/main.py#L21-L189)
- [MindSpider/DeepSentimentCrawling/keyword_manager.py](file://MindSpider/DeepSentimentCrawling/keyword_manager.py#L29-L336)
- [MindSpider/DeepSentimentCrawling/platform_crawler.py](file://MindSpider/DeepSentimentCrawling/platform_crawler.py#L27-L491)

### 数据库管理工具

数据库管理工具提供了完整的数据库运维功能：

```mermaid
classDiagram
class DatabaseManager {
-Engine engine
+connect() void
+show_tables() void
+show_statistics() void
+show_recent_data(days) void
+cleanup_old_data(days, dry_run) void
+close() void
}
class CLICommands {
+--tables
+--stats
+--recent N
+--cleanup N
+--execute
}
DatabaseManager --> CLICommands : "命令行接口"
```

**图表来源**
- [MindSpider/schema/db_manager.py](file://MindSpider/schema/db_manager.py#L30-L295)

**章节来源**
- [MindSpider/schema/db_manager.py](file://MindSpider/schema/db_manager.py#L30-L299)

## 依赖关系分析

### 外部依赖管理

MindSpider采用分层依赖管理策略，确保各模块的独立性和可维护性：

```mermaid
graph TB
subgraph "核心依赖"
PYMYSQL[pymysql]
ASYNC_MY[asyncmy]
SQLALCHEMY[sqlalchemy]
OPENAI[openai]
HTTPX[httpx]
PLAYWRIGHT[playwright]
end
subgraph "模块特定依赖"
NUMPY[numpy]
PANDAS[pandas]
REGEX[regex]
TQDM[tqdm]
REDIS[redis]
FASTAPI[fastapi]
MATPLOTLIB[matplotlib]
end
subgraph "情感分析模型"
TRANSFORMERS[transformers]
PEFT[peft]
TORCH[torch]
end
PYMYSQL --> SQLALCHEMY
ASYNC_MY --> SQLALCHEMY
OPENAI --> BroadTopicExtraction
HTTPX --> BroadTopicExtraction
PLAYWRIGHT --> DeepSentimentCrawling
REDIS --> DeepSentimentCrawling
TRANSFORMERS --> SentimentAnalysisModel
PEFT --> SentimentAnalysisModel
TORCH --> SentimentAnalysisModel
```

**图表来源**
- [MindSpider/requirements.txt](file://MindSpider/requirements.txt#L1-L63)

### 模块间耦合关系

系统采用松耦合设计，通过明确的接口和配置实现模块间的解耦：

```mermaid
graph LR
subgraph "配置层"
CONFIG[config.py]
end
subgraph "数据访问层"
DB_MANAGER[database_manager.py]
SCHEMA_DB[db_manager.py]
end
subgraph "业务逻辑层"
BTE_MAIN[BroadTopicExtraction/main.py]
DSC_MAIN[DeepSentimentCrawling/main.py]
end
subgraph "爬虫执行层"
KEYWORD_MGR[keyword_manager.py]
PLATFORM_CRAWLER[platform_crawler.py]
MEDIA_CRAWLER[MediaCrawler]
end
CONFIG --> BTE_MAIN
CONFIG --> DSC_MAIN
DB_MANAGER --> BTE_MAIN
DB_MANAGER --> DSC_MAIN
SCHEMA_DB --> DSC_MAIN
KEYWORD_MGR --> DSC_MAIN
PLATFORM_CRAWLER --> DSC_MAIN
MEDIA_CRAWLER --> PLATFORM_CRAWLER
```

**图表来源**
- [MindSpider/config.py](file://MindSpider/config.py#L16-L35)
- [MindSpider/BroadTopicExtraction/main.py](file://MindSpider/BroadTopicExtraction/main.py#L29-L38)
- [MindSpider/DeepSentimentCrawling/main.py](file://MindSpider/DeepSentimentCrawling/main.py#L21-L28)

**章节来源**
- [MindSpider/requirements.txt](file://MindSpider/requirements.txt#L1-L63)
- [MindSpider/main.py](file://MindSpider/main.py#L183-L256)

## 性能考虑

### 数据库性能优化

系统在数据库层面采用了多项优化策略：

1. **索引优化**：为高频查询字段建立复合索引
2. **连接池管理**：使用异步连接池提高并发性能
3. **数据分区**：支持大数据量的分区表设计
4. **缓存策略**：集成Redis缓存热点数据

### 爬取性能优化

```mermaid
flowchart TD
Start[性能优化开始] --> DBOpt[数据库优化]
Start --> CrawlOpt[Crawl优化]
Start --> SysOpt[系统优化]
DBOpt --> Index[建立复合索引]
DBOpt --> Pool[连接池配置]
DBOpt --> Cleanup[定期清理策略]
CrawlOpt --> Delay[合理设置爬取间隔]
CrawlOpt --> Proxy[使用代理池]
CrawlOpt --> Concurrency[控制并发数]
SysOpt --> Redis[Redis缓存]
SysOpt --> Queue[异步任务队列]
SysOpt --> Monitor[系统监控]
Index --> End[性能提升]
Pool --> End
Cleanup --> End
Delay --> End
Proxy --> End
Concurrency --> End
Redis --> End
Queue --> End
Monitor --> End
```

### 并发处理策略

系统采用异步并发处理模式，支持高并发爬取：

- **异步数据库操作**：使用SQLAlchemy异步引擎
- **异步HTTP请求**：基于httpx实现异步网络请求
- **异步文件操作**：使用aiofiles处理文件I/O
- **进程隔离**：每个平台爬取任务在独立进程中执行

## 故障排除指南

### 常见问题及解决方案

#### 配置问题

| 问题类型 | 症状 | 解决方案 |
|---------|------|----------|
| 数据库连接失败 | 连接超时或认证失败 | 检查DB_HOST、DB_PORT、DB_USER、DB_PASSWORD配置 |
| API密钥错误 | AI话题提取失败 | 确认MINDSPIDER_API_KEY配置正确 |
| 爬虫登录失败 | 平台登录超时 | 关闭无头模式，手动扫码登录 |

#### 爬取问题

| 问题类型 | 症状 | 解决方案 |
|---------|------|----------|
| 数据为空 | 爬取结果为0 | 确认平台已登录，检查关键词有效性 |
| 请求频繁 | 被平台限制 | 调整爬取间隔，使用代理IP |
| 内存不足 | 程序崩溃 | 降低并发数，优化数据处理逻辑 |

#### 数据库问题

| 问题类型 | 症状 | 解决方案 |
|---------|------|----------|
| 表不存在 | 查询失败 | 运行数据库初始化脚本 |
| 字符集错误 | 中文乱码 | 检查DB_CHARSET配置为utf8mb4 |
| 连接池耗尽 | 请求超时 | 调整连接池大小，优化资源释放 |

**章节来源**
- [MindSpider/README.md](file://MindSpider/README.md#L435-L472)

### 调试工具

系统提供了完善的调试和监控工具：

1. **状态检查**：`python main.py --status` 查看系统状态
2. **数据库管理**：内置数据库管理工具，支持表结构查看和数据清理
3. **日志系统**：基于loguru的日志记录，支持详细调试信息
4. **性能监控**：实时监控系统资源使用情况

## 结论

MindSpider AI爬虫系统通过模块化设计和AI技术的深度融合，实现了从热点发现到深度分析的完整舆情监控解决方案。系统具有以下优势：

1. **架构清晰**：模块职责明确，便于维护和扩展
2. **技术先进**：集成AI话题提取和多平台爬取技术
3. **性能优秀**：采用异步并发和数据库优化策略
4. **易于使用**：提供完整的配置管理和部署指导

未来发展方向包括：
- 增强情感分析模型的准确性
- 扩展更多社交媒体平台支持
- 优化爬取策略以适应平台反爬虫机制
- 提升系统的可扩展性和分布式部署能力

## 附录

### 部署配置指南

#### 环境要求
- Python 3.9+
- MySQL 5.7+ 或 PostgreSQL
- Playwright浏览器驱动
- 至少8GB内存（推荐16GB+）

#### 安装步骤
1. 克隆项目并初始化子模块
2. 创建Python虚拟环境
3. 安装依赖包
4. 配置数据库和API密钥
5. 初始化数据库表结构
6. 首次运行需要各平台登录

#### 配置文件示例

```python
# .env文件配置示例
DB_HOST = "localhost"
DB_PORT = 3306
DB_USER = "mindspider"
DB_PASSWORD = "your_password"
DB_NAME = "mindspider"
DB_CHARSET = "utf8mb4"

MINDSPIDER_API_KEY = "sk-your-deepseek-key"
MINDSPIDER_BASE_URL = "https://api.deepseek.com"
MINDSPIDER_MODEL_NAME = "deepseek-chat"
```

### 使用示例

#### 基本使用
```bash
# 运行完整流程
python main.py --complete --test

# 仅运行话题提取
python main.py --broad-topic

# 仅运行深度爬取
python main.py --deep-sentiment --platforms xhs dy --test
```

#### 高级配置
```bash
# 指定日期和参数
python main.py --complete \
    --date 2024-01-15 \
    --keywords-count 150 \
    --max-keywords 30 \
    --max-notes 25 \
    --test
```

### 扩展开发指导

#### 添加新平台支持
1. 在`DeepSentimentCrawling/MediaCrawler/media_platform/`创建新平台目录
2. 实现平台的核心功能模块
3. 更新平台支持列表
4. 测试新平台的爬取功能

#### 自定义话题提取算法
1. 修改`topic_extractor.py`中的提示词模板
2. 调整关键词提取逻辑
3. 测试AI模型的准确性和稳定性
4. 优化处理流程和错误恢复机制

#### 数据库扩展
1. 更新`mindspider_tables.sql`添加新表结构
2. 在`models_sa.py`中定义数据模型
3. 实现相应的数据访问方法
4. 运行数据库初始化脚本