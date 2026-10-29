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

测试发现来源层级问题：插入新高层节点可能从不属于该层的入口搜索，
进而让低层节点出现在高层邻接表。迁移版显式保留、与冻结原型比较并记录。
在建立正式性能基线前，应单独修复并验证图质量，不能把该修复当作调度收益。

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
CI 配置暂保存在子模块 `docs/ci.yml.example`，现有 GitHub 凭据缺少 workflow scope，尚未启用 Actions。

## 下一阶段

1. 为层级成员问题增加独立修复版本和质量验证。
2. 量化原型的哈希访问、候选排序、重复评分与内存开销。
3. 建立基础优化后的强基线，再实现固定 B/P/W。
4. 如果工作负载间配置偏好确有差异，再研究自适应和并发。

遵循根目录 workflow.md：先推子仓库，再提交主仓库指针。
实验固定提交，不用 update --remote 自动滚动基线。
