---
type: "参考"
title: "Testing and Change Verification"
openwiki_generated: true
verified:
  - by: openwiki/0.5.0
    at: 2026-09-10T03:05:58.506Z
sources:
  - id: openwiki-source-6388fc05a9158290812affb6
    resource: repo://internal/agentgateway/service_test.go
  - id: openwiki-source-8996d58d2067c1e0a3411962
    resource: repo://internal/bootstrap/cors_test.go
  - id: openwiki-source-b7940a732c5d5b1c1926c066
    resource: repo://internal/bootstrap/spa_traversal_test.go
  - id: openwiki-source-334ac0d09ca2932b4102e644
    resource: repo://internal/graphstore/file_memory_test.go
  - id: openwiki-source-c80aca65d5a881b68c9a8409
    resource: repo://internal/graphstore/memory_test.go
  - id: openwiki-source-9399b815ab7411fa5ae5842a
    resource: repo://internal/query/timeout_test.go
  - id: openwiki-source-be3eec976952ae86c4b83240
    resource: repo://internal/umodel/import_confine_test.go
  - id: openwiki-source-dfa7a934e5ae60a00daa5085
    resource: repo://internal/workspace/service_test.go
  - id: openwiki-source-012f2c78e3b1446dfc35803f
    resource: repo://Makefile
  - id: openwiki-source-2434b2c7ed42cd4ae41c945c
    resource: repo://tests/contract/graphstore_contract_test.go
  - id: openwiki-source-f9844d3421f56d94f3a97c1a
    resource: repo://tests/contract/public_contract_test.go
  - id: openwiki-source-c6ff61348d8fef66d1c1f2bd
    resource: repo://tests/e2e/business_flow_test.go
  - id: openwiki-source-dc66c8a713672905b465d08f
    resource: repo://tests/e2e/cli_business_flow_test.go
  - id: openwiki-source-8d9ddd660a45814864967824
    resource: repo://tests/e2e/mcp_business_flow_test.go
  - id: openwiki-source-5b830c2d983a802fcc0bc94e
    resource: repo://tests/golden/query_golden_test.go
  - id: openwiki-source-d213ffb921aba27761468a9d
    resource: repo://tests/integration/capability_gate_test.go
  - id: openwiki-source-9f23b120b22b67dbd93f1ace
    resource: repo://tests/integration/http_flow_test.go
  - id: openwiki-source-7de0c852333fe3be5b9d4281
    resource: repo://tests/integration/quickstart_health_test.go
  - id: openwiki-source-cc18f7c982b54a9643a61d74
    resource: repo://tests/README.md
  - id: openwiki-source-aff14dcab73429d5391387e3
    resource: repo://tools/guards/architecture_guard.py
  - id: openwiki-source-3eb877b4c195e14ee0c36ecf
    resource: repo://tools/verify/check_manifest.py
  - id: openwiki-source-4136f0664cb1897bbb9234b7
    resource: repo://tools/verify/verify_go.sh
  - id: openwiki-source-1ff6f05d8b4d10fe5265f281
    resource: repo://tools/verify/verify_java.sh
  - id: openwiki-source-5f5d41b7f217a62333118bc0
    resource: repo://tools/verify/verify_python.sh
  - id: openwiki-source-d9a095b0e10881a51c9f81ae
    resource: repo://web/e2e/query-capability.spec.ts
  - id: openwiki-source-01fd15c10619cde2573c91ea
    resource: repo://web/e2e/umodel-explorer.spec.ts
  - id: openwiki-source-9953c8a4b5a75bef445cf794
    resource: repo://web/playwright.config.ts
generated: { by: "openwiki/0.5.0", at: "2026-09-10T03:05:58.506Z" }
---


# Testing and Change Verification

这页不是测试文件清单，而是**改动行为 → 应验证的边界 → 最小聚焦命令 → 必跑门禁**的地图。测试目录按责任分层：`tests/unit/` 放模块单测，`tests/contract/` 覆盖公共契约、Storage interface、MCP 和 SDK，`tests/integration/` 覆盖跨模块路径，`tests/e2e/` 覆盖 Quickstart 端到端，`tests/golden/` 固化查询语义、explain 和文档示例，`tests/architecture/` 对依赖方向及禁用 API 做架构守卫。

## 先选验证层级

| 改动或风险 | 首选验证 | 重点边界与命令 |
|---|---|---|
| 解析器、planner、renderer、查询服务、搜索路由 | `internal/query` 单测；必要时 `tests/golden` | `go test ./internal/query -run 'Test(Parse|Planner|Execute|.*Search|.*Topo|.*Format|.*Mode)'`；语义或 explain 改动再跑 `go test ./tests/golden -run TestQueryGoldenResults` |
| GraphStore、实体/关系方法语义、TTL、上下文 | `internal/graphstore`、`internal/entitystore` | `go test ./internal/graphstore ./internal/entitystore`；并跑 `go test -race ./internal/graphstore ./internal/entitystore` |
| UModel schema 校验、导入、删除、路径安全 | `internal/umodel`、schema registry/validator | `go test ./internal/umodel ./internal/umodel/schemaspec`；路径或不可信输入改动必须包含 import confinement 和 symlink 测试 |
| provider 注册、能力或 fallback | `tests/contract`、`tests/integration`、provider 专项测试 | `go test ./tests/contract -run 'Test(GraphStore|DefaultApp|GraphStoreProvider)'`；`make test-capability` |
| HTTP handler、workspace、持久化、Quickstart、CORS、SPA | `internal/bootstrap`、`internal/workspace`、`tests/integration` | `go test ./internal/bootstrap ./internal/workspace ./tests/integration`；UI 入口或跨模块流程变化再跑 `make test-quickstart-health` |
| Agent discovery、tool/resource policy、MCP | `internal/agentgateway`、`tests/contract`、MCP E2E | `go test ./internal/agentgateway ./tests/contract -run 'MCP|Agent|Public'`；`go test ./tests/e2e -run TestMCP` |
| `umctl` 命令或公共 REST surface | public contract、CLI E2E、HTTP integration | `go test ./tests/contract ./tests/integration -run 'Contract|HTTP'`；`go test ./tests/e2e -run TestCLI` |
| Web UI、查询交互、explorer、提交预览 | `web/e2e` Playwright | `make test-ui`；服务运行后 `make test-ui-e2e`，或 `cd web && npx playwright test --reporter=list` |
| examples、Quickstart 数据或 manifest | Quickstart health、sample/integration E2E、validator | `make example-validate`、`make check-manifest`、`make test-quickstart-health` |
| schema、嵌入 schema、文档或 generated SDK | schema/manifest/docs gate 与 `make verify` | `make schemas-embed-check docs-schema-check verify`；生成源改变时先 `make expand`，再检查 diff 并提交生成物 |

## Query 与存储：先测语义，再测 provider 边界

### Query planner、执行和 golden 语义

Query Service 的测试应覆盖完整链路，而非只断言 parser 能接受字符串：source 选择（`.umodel`、`.entity`、`.topo`、`.runbook_set`）、`with/where/project/sort/limit` 管道、ID 和 `topk` 过滤、受控 `graph-call`/Cypher、search mode、pushdown/fallback 以及返回的 explain。解析器还必须对引号内含有 `<=`、`>=` 等操作符的值保持 quote-aware 行为。首选：

```bash
go test ./internal/query -run 'Test(Parse|Planner|Execute|.*Search|.*Topo|TestParsePredicateQuoteAware)'
go test ./tests/golden -run TestQueryGoldenResults
```

`tests/golden/query_golden_test.go` 将 `.umodel`、`.entity` 和 `.topo` 的列、行、分页、provider、pushdown/fallback、operator、depth 和 `time_range_applied` 一并比较；因此排序、默认 limit、explain 字段或 renderer 选择的任何改变，都应更新意图明确的 golden，而不是只扩大普通单测。查询管道的跨模块行为可用：

```bash
go test ./tests/integration -run 'TestHTTPQuickFlow|TestHTTPUModelImportThenQuery|TestHTTPSampleImportThenQuery|TestCapabilityGate'
```

### timeout 与 cancellation 必须分开

过期 deadline 是可归因的 provider timeout，映射为 `CodeTimeout`；父 context 被取消（例如客户端断开）不能伪装成 timeout 或可重试的 provider 故障。服务层和 memory store 都要验证：

```bash
go test ./internal/query -run 'Test(ParseQueryTimeout|AsQueryTimeout|ExecuteDeadlineExceededReturnsTimeout|ExecuteCancelledContextIsNotTimeout)'
go test ./internal/graphstore -run TestQueryRespectsContextCancellation
```

新增 provider 或改执行器时，至少复用同一组 deadline/cancel 边界，并确认错误码在 HTTP、CLI 或 Agent envelope 中没有被重新包装成另一类错误。

### provider capability gate 与 Cypher

Query planner 依赖 provider capabilities 决定 pushdown、application fallback、limit/depth 和受支持的 graph call；不能只在 memory provider 上验证“能返回行”。GraphStore contract 会同时 exercise memory 和 file-memory，并检查 workspace/schema、UModel snapshot、实体和关系查询、时间范围、未来数据为空、删除结果以及 health/capabilities：

```bash
go test ./tests/contract -run 'TestMemoryGraphStoreConformance|TestFileMemoryGraphStoreConformance|TestGraphStoreProviderRegistry'
go test ./internal/query -run 'TestPlannerRejectsLimitDepthAndUnsupportedCapabilities|TestExplainShowsTopoPushdowns|TestExplainShowsCapabilityFallback'
go test ./internal/graphstore -run 'TestMemoryCypher|Test.*MaxLimit|TestQueryRespectsContextCancellation'
make test-capability
```

`make test-capability` 是能力门禁，不是样例烟测：它导入 `multi-domain-quickstart`，检查 UModel/entity/topology 查询、pipeline、ID/query/topk、direct/neighbor/Cypher、跨域关系和 explain 元数据。改 provider registry、默认 provider、capability 字段或 Cypher read-only 约束时，应把它和 `go test ./tests/contract` 一起作为回归门槛。

## UModel 导入、schema 和持久化安全

### 导入 confinement 与 schema validation

API 来源的 `Import` 路径必须限制在配置的 import root 内；绝对路径、`..` 越界、root 内指向外部的目录 symlink，以及目录扫描时指向外部的嵌套文件 symlink 都必须在读取文件前被拒绝并返回 `INVALID_ARGUMENT`。只有明确的 trusted sample 入口可绕过 confinement。用以下测试锁住安全边界：

```bash
go test ./internal/umodel -run 'TestImport(ConfinesToRoot|RejectsSymlinkEscape|RejectsNestedSymlinkFileEscape)'
go test ./internal/umodel -run 'TestValidate|TestImportFile|TestImportDirectory|TestImportRejectsInvalidSources'
```

schema 变更还要验证 envelope version、kind registry、必填嵌套字段、类型/枚举/模式和未知 kind；`entity_set_link` 的 endpoint 以及 `entity_set` 的 `fields[]` 是典型回归边界：

```bash
go test ./internal/umodel/schemaspec -run 'Test(DefaultRegistry|RegistryAccepts|EntitySetLink|EntitySetSchema)'
```

如果改的是 expanded schema、`schemas/manifest.yaml` 或生成规则，额外执行：

```bash
make check-manifest
make expand
make schemas-embed-check
make docs-schema-check
make example-validate
```

其中 `check-manifest` 对比 manifest 声明的 kind 与 `schemas/core/` 文件；`make expand` 会展开并验证 schema、生成 Go/Python/Java SDK、格式化 Go SDK，并同步 embedded schema；`docs-schema-check` 以 check 模式确认 schema HTML/reference 未漂移；`example-validate` 批量验证 `examples`。

### 写入方法、可见性和持久化恢复

实体/关系 `Create` 的重复键应报告 `ALREADY_EXISTS`，`Update` 应保留 `__first_observed_time__`，`Expire` 应从 current 查询隐藏但在合适的 historical range 以 deleted 记录可见；删除和 TTL 不应误伤另一类数据。优先运行：

```bash
go test ./internal/entitystore ./internal/graphstore -run 'TestWrite|TestExpire|TestRunTTL|TestMemoryStore(EntityMethod|RelationMethod)|TestFileMemoryStore'
go test ./tests/contract -run 'Test(MemoryGraphStoreConformance|FileMemoryGraphStoreConformance)'
```

file-memory 的恢复测试不仅检查重启后 workspace/UModel/entity/relation 仍可查询，也检查 legacy single-file 状态迁移及删除后的再次 reopen；workspace 测试覆盖 file-memory 和 ladybug 数据目录恢复，以及 deleted workspace 不被重新激活：

```bash
go test ./internal/bootstrap ./internal/workspace ./internal/graphstore -run 'Test.*(Persists|PersistsAcrossReopen|Migrates|Recovers|Tombstone|DoesNotRevive)'
```

provider 改动还需明确区分：默认 `NewApp` 使用 `local.ladybug`，显式 `memory`/`file.memory` 可注册选择，未知 provider 必须失败；若本机配置了 Ladybug，再按项目约定运行 `UMODEL_TEST_LADYBUG=1 make test-ladybug`。

## HTTP、公共契约、Agent/MCP 与 CLI

### REST/OpenAPI 契约

公共 API 只能通过 Query Service 提供 domain reads；workspace metadata API 不应扩展 runtime lifecycle 操作。契约测试会读取 `api/openapi/openapi.yaml`，检查 workspace、UModel import/validate/write、entity/relation write、query execute/explain、agent discover/tools/resources 等路径和 response schema，同时拒绝旧的 domain read API：

```bash
go test ./tests/contract -run 'TestOpenAPIContainsPublicContracts|TestMCPToolSchemaDefaults'
make guard
```

错误验证要看**状态码 + 稳定 error code**，例如 query parse error、validation failure 和默认关闭的 write tool，而不是仅断言 HTTP 失败：

```bash
go test ./tests/integration -run 'TestHTTPErrorContractsAndWriteToolDefault'
```

### Agent discovery、resource 隔离和 MCP

AgentGateway 的只读 query tools 默认启用；写工具（如 `entity_write`、`umodel_import`、`entity_expire`）默认关闭，并带有显式 enable 元数据。discovery/resource 必须返回结构化、read-only 的入口和 next actions，resource 不能泄漏运行时 rows；开启写能力后，tool 仍应调用 service layer，而不是绕过服务。聚焦运行：

```bash
go test ./internal/agentgateway -run 'Test(Tools|Execute|Discover|ReadResources|UModelValidate|WriteTools)'
go test ./tests/e2e -run 'TestMCPBusinessFlowCoversDiscoveryResourcesAndToolPolicy|TestMCPQueryToolDiscoversAndLoadsEntityLinkedSkill'
```

MCP E2E 通过 `go run ./cmd/umodel-mcp` 逐行发送 JSON-RPC，覆盖 `initialize`、`tools/list`、`tools/call`、`resources/list`、`resources/read`、query examples、`structuredContent`/`text/toon`、linked skill/knowledge 加载和 `TOOL_DISABLED`。协议 schema 变化应同时检查 `api/mcp/tools.schema.json` 的 transport、method、input/output schema、capability metadata 和 `read_only` 字段。

### CLI 只走 public contracts

`umctl` E2E 会构建真实 CLI 并通过 HTTP server 完成 workspace create/get/update/list/delete、UModel validate/put/import/export、entity/topo write/query、agent discover/tool、expire/delete，以及默认关闭写 tool 的错误。命令或 API surface 改动不要只运行内部 Go service 单测：

```bash
go test ./tests/e2e -run TestCLIBusinessFlowUsesOnlyPublicContracts
go test ./tests/integration -run 'TestHTTPQuickFlow|TestHTTPErrorContractsAndWriteToolDefault'
```

## Quickstart、业务流与 UI 边界

### capability、quickstart health 和业务 E2E

Quickstart 的健康检查把 `sample-data/manifest.json` 与 entities/relations 实际数据对齐，并验证 domain、seed entity ID、entity type 对应的 entity_set YAML、跨域 relation 和唯一 ID；服务层则验证 workspace metadata、UModel/entity/topology 导入及重复加载安全：

```bash
make test-capability
make test-quickstart-health
go test ./internal/bootstrap -run 'TestLoadQuickStart|TestRunbookSetSearchIsConfigured'
go test ./tests/e2e -run 'TestRESTBusinessFlowCoversCoreScenarios|TestQuickstart'
```

`TestRESTBusinessFlowCoversCoreScenarios` 是业务流回归：创建和 tombstone workspace，validate/put/import UModel，current/historical entity 和 relation 生命周期，topology explain、非法 query、Agent discovery/resource 隔离与 query tool。任何涉及导入、写入、生命周期或错误契约的改动都应跑它。

### SPA traversal、CORS 与 Playwright

HTTP 集成测试覆盖配置 UI 目录后的 `/`、SPA 深层路由和静态 asset；bootstrap 单测还确保 `/../`、编码 traversal 和嵌套 traversal 不会泄露 UI 目录外文件。CORS 测试覆盖 loopback origin 的 preflight 与普通 API response：

```bash
go test ./internal/bootstrap ./tests/integration -run 'Test(SPA|CORS|HTTPServesSPA)'
```

真实浏览器层用 `web/playwright.config.ts` 的 Chromium project（base URL 为 `http://localhost:5173`）：

```bash
make test-ui
make test-ui-e2e
# 或：cd web && npx playwright test --reporter=list
```

`web/e2e/query-capability.spec.ts` 验证 workspace landing、UModel/entity/topology query、explain provider、explorer graph、提交预览成功/失败和 API Debugger；`web/e2e/quickstart-demo.spec.ts` 验证 demo workspace、各 domain/entity/topology/cross-domain 数据及 Agent discovery；`web/e2e/umodel-explorer.spec.ts` 验证 explorer 使用 `.umodel | sort name | limit 2000`，不突破 request limit contract。UI 改动若涉及 deep link、query 结果、Monaco/diff drawer、分页或错误显示，应至少选择相应 spec，而不只做 TypeScript build。

## 生成物与最终本地门禁

### generated SDK 与 schema 资产

生成 SDK 的验证脚本按语言执行：Go 检查 module、`go build ./...`、`go vet ./...` 和测试；Python 检查 `umodel` 导入并在存在时运行 SDK tests；Java 在存在 Maven/Java 与 `generated/java/pom.xml` 时执行 compile/test，缺少可选工具时脚本会跳过并返回成功。统一入口是：

```bash
make verify
# 需要单独定位时：
make verify-go
make verify-python
make verify-java
```

不要手改 generated SDK 来修复 schema 变更：修改 schema 后运行 `make expand`，审阅生成 diff，再运行 `make verify`、`make schemas-embed-check`、`make docs-schema-check` 和 `make example-validate`。

### 必跑 gates

提交前至少运行：

```bash
make guard
make test-service
make test-capability
make test-quickstart-health
make verify
make check-manifest
make example-validate
make schemas-embed-check
make docs-schema-check
```

`make test-service` 使用 `go test -race ./...`，是并发、context、共享 store 和 handler 改动的必跑门禁；`make test` 组合 `guard`、race-enabled service tests 和 `verify`。若要复现 CI 的完整本地序列，直接运行：

```bash
make ci
```

`make ci` 还会执行 `build-service`、capability/quickstart health、generated SDK verification、manifest、examples、embedded schema 与 schema docs checks，并在全部通过后输出 `Local CI passed.`。工具缺失导致 SDK 验证被跳过时，仍应记录跳过原因；若改动触及被跳过的语言生成物，应在具备对应工具的环境补跑，而不能把“未执行”当作验证通过。

## 失败诊断顺序

1. **先看边界错误**：解析/验证失败看稳定 code 和输入；timeout 与 cancellation 分开看；provider capability failure 看 explain 的 pushdown/fallback。
2. **再看状态 owner**：current/historical visibility、tombstone、reopen 后数据和 workspace metadata 是否由正确 store/service 持有。
3. **再看公共 surface**：OpenAPI/MCP schema、Agent discovery、CLI 输出和 UI 是否仍与同一契约一致。
4. **最后跑全门禁**：先用最小 `go test -run` 缩小范围，再用 `make test-service` 和 `make ci` 确认没有 race、架构、生成物或示例漂移。
