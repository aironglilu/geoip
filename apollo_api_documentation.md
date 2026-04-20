# Apollo.io API 接口文档

## 一、按条件批量搜索公司

通过该接口可基于多种筛选条件批量查询符合条件的公司数据，适用于构建目标企业名单、市场分析和销售线索发现等场景。

- **在线链接**：[https://api.apollo.io/api/v1/mixed_companies/search](https://api.apollo.io/api/v1/mixed_companies/search)  
- **请求方法**：`POST`
- **认证方式**：在请求头中携带 API 密钥  
  ```http
  Authorization: Bearer <your_api_key>
  ```

### 请求体（JSON）参数说明

支持多维度复合筛选，所有参数均为可选，可根据业务需求自由组合：

- `q_organization_domains_list[]`：指定域名数组进行精确匹配，最多支持 1000 个域名（如 `apollo.io`）
- `organization_num_employees_ranges[]`：员工规模范围筛选，格式为 `"min,max"`（如 `"250,1000"`）
- `organization_locations[]`：公司总部所在地筛选（如 `japan`、`united states`）
- `organization_not_locations[]`：排除特定地区的公司
- `revenue_range[min]` / `revenue_range[max]`：年营收范围（整数，单位美元）
- `currently_using_any_of_technology_uids[]`：当前使用的技术栈标识符（如 `salesforce`、`aws`）
- `q_organization_keyword_tags[]`：关键词标签过滤（如 `SaaS`、`AI`）
- `q_organization_name`：公司名称模糊搜索关键字
- `organization_ids[]`：通过 Apollo 内部 ID 精确匹配公司
- `latest_funding_amount_range[min]` / `max`：最近一轮融资金额范围
- `total_funding_range[min]` / `max`：累计融资总额范围
- `latest_funding_date_range[min]` / `max`：最近融资日期范围，格式为 `YYYY-MM-DD`
- `q_organization_job_titles[]`：当前正在招聘的职位标题关键词（如 `engineer`）
- `organization_job_locations[]`：招聘岗位所在地理位置
- `organization_num_jobs_range[min]` / `max`：当前开放职位数量区间
- `organization_job_posted_at_range[min]` / `max`：职位发布时间范围
- `page`：分页页码，从 1 开始（默认为 1）
- `per_page`：每页返回记录数，建议不超过 100 条以保证性能

### 请求示例

```json
{
  "q_organization_domains_list": ["apollo.io"],
  "organization_num_employees_ranges": ["250,1000"],
  "organization_locations": ["japan"],
  "page": 1,
  "per_page": 10
}
```

### 返回结果模板

响应包含分页信息和公司列表，每个公司对象包含基础字段：

```json
{
  "pagination": {
    "page": 1,
    "per_page": 10,
    "total_entries": 1184,
    "total_pages": 592
  },
  "organizations": [
    {
      "id": "string",
      "name": "string",
      "website_url": "string",
      "linkedin_url": "string",
      "founded_year": 0,
      "estimated_num_employees": 0
    }
  ]
}
```

## 二、通过域名获取单个公司完整信息

该接口用于根据公司域名获取其详细的组织信息，适用于企业背景调查、客户画像构建和数据验证等场景。通过传入精确的域名即可返回结构化的公司档案。

- **在线链接**：[https://api.apollo.io/api/v1/organizations/enrich](https://api.apollo.io/api/v1/organizations/enrich)  
- **请求方法**：`GET`
- **认证方式**：在请求头中携带 API 密钥  
  ```http
  Authorization: Bearer <your_api_key>
  ```

### 查询参数

仅需提供以下必填参数即可发起请求：

- `domain`（必填）：目标公司的主域名，格式为不带协议和子路径的标准域名（如 `apollo.io`），不包含 `www.` 或邮箱前缀（如 `@`）

### 请求示例

完整的请求 URL 示例：
```
https://api.apollo.io/api/v1/organizations/enrich?domain=apollo.io
```

### 返回结果模板

响应体包含一个完整的 `organization` 对象，字段涵盖公司基础信息、融资历史和技术栈等：

```json
{
  "organization": {
    "id": "string",
    "name": "string",
    "website_url": "string",
    "primary_domain": "string",
    "founded_year": 0,
    "estimated_num_employees": 0,
    "industry": "string",
    "location": "string",
    "linkedin_url": "string",
    "twitter_url": "string",
    "facebook_url": "string",
    "logo_url": "string",
    "total_funding": 0,
    "latest_funding_stage": "string",
    "funding_events": [
      {
        "date": "string",
        "amount": "string",
        "currency": "string"
      }
    ],
    "current_technologies": [
      {
        "name": "string",
        "category": "string"
      }
    ]
  }
}
```

## 三、批量域名富化公司信息

该接口支持通过一次性提交多个公司域名，批量获取其详细的组织档案信息。适用于企业客户数据清洗、CRM 补全、市场调研名单增强等高效率数据富化场景。相比单个查询，此接口能显著减少请求次数，提升集成性能。

- **在线链接**：[https://api.apollo.io/api/v1/organizations/bulk_enrich](https://api.apollo.io/api/v1/organizations/bulk_enrich)  
- **请求方法**：`POST`
- **认证方式**：在请求头中携带 API 密钥  
  ```http
  Authorization: Bearer <your_api_key>
  ```

### 请求体（JSON）参数说明

仅需提供一个域名数组即可发起批量请求，系统将并行处理并返回对应结果：

- `domains[]`：目标公司的主域名列表，格式为标准域名（如 `apollo.io`），不包含协议（`https://`）、子路径或邮箱前缀（`@`）。**最多支持 10 个域名**，超出将被截断或返回错误。

### 请求示例

```json
{
  "domains": ["apollo.io", "microsoft.com"]
}
```

### 返回结果模板

响应为一个 JSON 数组，每个元素对应一个公司的完整组织信息对象，字段结构与单个富化接口一致，包含基础信息、融资历史和技术栈等关键数据：

```json
[
  {
    "id": "string",
    "name": "string",
    "website_url": "string",
    "primary_domain": "string",
    "founded_year": 0,
    "estimated_num_employees": 0,
    "industry": "string",
    "location": "string",
    "linkedin_url": "string",
    "twitter_url": "string",
    "facebook_url": "string",
    "logo_url": "string",
    "total_funding": 0,
    "latest_funding_stage": "string",
    "funding_events": [
      {
        "date": "string",
        "amount": "string",
        "currency": "string"
      }
    ],
    "current_technologies": [
      {
        "name": "string",
        "category": "string"
      }
    ]
  }
]
```

## 四、官方API文档与开发者资源

为支持开发者高效集成与使用 Apollo.io API 服务，官方提供了完整的文档体系与技术支持资源。以下为关键链接汇总，涵盖接口说明、合作伙伴计划及产品功能详情，便于用户深入查阅与扩展应用。

- **[官方API文档](https://docs.apollo.io)**  
  提供最权威的 API 接口说明，包括认证机制、请求格式、响应结构、错误码详解及版本更新日志，是开发集成的核心参考指南。

- **[API Reseller 合作伙伴计划](https://www.apollo.io/partners/api-reseller)**  
  面向数据服务商、SaaS 平台和技术代理商开放的合作项目，支持第三方将 Apollo 的 B2B 数据能力整合至自有产品中，实现联合商业化。

- **[数据富化解决方案页面](https://www.apollo.io/solutions/b2b-data-enrichment)**  
  详细介绍企业数据清洗、CRM 补全、客户画像增强等场景下的技术实现路径与业务价值，适用于营销自动化与销售智能系统构建。

- **[B2B 数据库功能页](https://www.apollo.io/product/search)**  
  展示 Apollo 强大的企业搜索能力，支持基于行业、规模、技术栈、融资情况等多维度筛选目标客户，助力精准获客与市场分析。

- **[知识库（Knowledge Base）](https://knowledge.apollo.io/hc/en-us)**  
  官方帮助中心，收录常见问题解答、集成最佳实践、故障排查指南及使用技巧，为开发者和运营人员提供持续支持。