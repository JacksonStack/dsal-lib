# Rust HNSW 调度研究

本路线从 `JacksonStack/rustv` 提取现有自写算法，研究邻居扩展的缓存和调度。
它与 `spec.md` 中的多语言教学自实现路线并行，当前不引入 C++ hnswlib。

## 仓库与来源

- 独立私有仓库：<https://github.com/JacksonStack/hnsw-scheduling-rust>
- 子模块：`hnsw/scheduling-rust`
- 来源：`JacksonStack/rustv` 提交 `a8879aea13aa7c75a40a7da11eb7ef1ba8441e45`
- 来源文件哈希和提取范围见子模块 `docs/source-manifest.json`、`docs/provenance.md`。
- 来源项目没有修改。未找到来源许可声明，不擅自附加开源许可，crate 禁止发布。

## 当前里程碑

已提取 HnswIndex、FlatIndex、FP32 平方 L2、配置和必要类型。
保留 ID 哈希分层、最近邻截断、HashMap/HashSet、结果 Vec 排序、负距离得分、
删除再插入更新以及最终重复评分。去掉 tenant、Collection、snapshot、WAL、HTTP。
增加独立 API 输入校验、efSearch 调整、原型差分测试、图结构检查和 CSV benchmark。
未提取持久化，未实现 B/P/W、显式 SIMD、预取和自适应。

来源层级问题已单独修复：层搜索只接受目标层内的入口，避免插入新高层节点时将旧低层入口错误提升。
提取标签保留原缺陷，冻结原型与修复版通过明确区分的结构及同图查询测试验证。
48 组小型质量审计中有 18 组 recall 下降，最大 13.75 个百分点，因此尚不能视为强性能基线。
详见子模块 `docs/layer-membership-fix.md` 及完整 CSV。结构修复与耗时变化不属于调度收益。

## 使用

```bash
git submodule update --init --recursive hnsw/scheduling-rust
cd hnsw/scheduling-rust
cargo test --offline
cargo run --release --example basic
cargo bench --bench baseline -- 1000 32 100 32 10 3 42
```

benchmark 参数依次为 N、维度、查询数、efSearch、k、重复次数、种子。
CSV 中 commit 通过 `HNSW_RESEARCH_COMMIT` 注入；未提供时为 unrecorded。
查询计时排除构建、ground truth、recall、预热和结果销毁。
小型随机数据只验证程序流程，不构成正式性能结论。
CI 配置位于子模块 `.github/workflows/ci.yml`，覆盖格式、Clippy、测试、示例和小型 benchmark。
初始行为提取提交由 `rustv-derived-baseline` 标签固定。算法修复与调度优化继续独立提交。

## 本机 SSH

JacksonStack 的本机 SSH 别名为 `github.com-per`，对应 `~/.ssh/js-mac`。
主仓库 origin 和子模块本地 URL 使用该别名，共享 `.gitmodules` 仍保留 HTTPS，
不让其他克隆环境依赖本机别名。其他机器执行 submodule sync 可能恢复共享 URL，这是正常行为。

## 下一阶段

1. 调查修复后聚类数据 recall 下降：检查底层有向可达性和按簇召回，再评估插入/选边策略。
2. 质量边界明确后量化哈希访问、候选排序、重复评分与内存开销。
3. 建立基础优化后的强基线，再实现固定 B/P/W。
4. 如果工作负载间配置偏好确有差异，再研究自适应和并发。

遵循根目录 workflow.md：先推子仓库，再提交主仓库指针。
实验固定提交，不用 update --remote 自动滚动基线。
