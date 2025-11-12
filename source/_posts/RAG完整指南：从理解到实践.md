title: RAG完整指南：从理解到实践
tags: []
categories: []
date: 2025-11-12 20:36:08
---
# RAG 完整指南：从理解到实践

我将为你系统性地讲解 RAG 的全部知识点，让你真正掌握它。

## 一、RAG 的本质理解

### 1.1 为什么需要 RAG？

传统大语言模型的局限：
- **知识截止日期**：模型只知道训练时的信息（比如我的知识截止到2025年1月）
- **无法访问私有数据**：不知道你公司的内部文档、数据库内容
- **容易产生幻觉**：在不确定时可能编造信息
- **更新成本高**：重新训练模型代价昂贵

**RAG 的解决方案**：让模型在回答时先"查资料"，就像学生考试时可以翻书一样。

### 1.2 RAG 工作流程图解

```
用户提问 → 问题向量化 → 检索相关文档 → 重排序 → 构建提示词 → LLM生成答案
   ↓           ↓              ↓            ↓         ↓           ↓
 "今年Q3     [0.2,        知识库中      选出最      "参考以下    基于真实
  销售额?"   0.8,...]     搜索匹配     相关的3条    文档..."     内容回答
```

---

## 二、RAG 核心技术组件

### 2.1 文档处理（Document Processing）

**步骤1：文档加载**
```python
# 支持的文档类型
- PDF 文件
- Word 文档
- 网页 HTML
- Markdown 文件
- 数据库记录
- API 返回数据
```

**步骤2：文本分块（Chunking）**
```
为什么要分块？
- 模型有输入长度限制
- 提高检索精确度
- 控制成本

常见分块策略：
1. 固定长度：每500字一块
2. 语义分块：按段落或章节
3. 重叠分块：块之间有50字重叠（避免信息断裂）
```

示例代码：
```python
from langchain.text_splitter import RecursiveCharacterTextSplitter

# 创建文本分割器
text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=500,        # 每块500字符
    chunk_overlap=50,      # 重叠50字符
    separators=["\n\n", "\n", "。", "!", "?", "；"]  # 分割优先级
)

# 分割文档
chunks = text_splitter.split_text(long_document)
```

### 2.2 向量化（Embedding）

**什么是 Embedding？**
把文本转换成数字向量，语义相似的文本向量距离近。

```
文本: "今天天气真好"  →  向量: [0.2, 0.8, -0.3, ..., 0.5]  (1536维)
文本: "今日天气不错"  →  向量: [0.21, 0.79, -0.31, ..., 0.48]  (很接近!)
```

**常用 Embedding 模型：**
- OpenAI: `text-embedding-3-small` (性价比高)
- OpenAI: `text-embedding-3-large` (精度更高)
- 开源: `bge-large-zh` (中文效果好)
- 多语言: `multilingual-e5-large`

代码示例：
```python
from openai import OpenAI

client = OpenAI()

# 文本向量化
response = client.embeddings.create(
    model="text-embedding-3-small",
    input="RAG是检索增强生成技术"
)

vector = response.data[0].embedding  # 获取向量
print(len(vector))  # 1536维
```

### 2.3 向量数据库（Vector Database）

**为什么需要向量数据库？**
- 存储海量向量（百万级以上）
- 快速相似度检索（毫秒级）
- 支持元数据过滤

**主流向量数据库：**

| 数据库 | 特点 | 适用场景 |
|--------|------|----------|
| **Pinecone** | 云服务、易用 | 快速原型、小团队 |
| **Weaviate** | 开源、功能全 | 企业级应用 |
| **Chroma** | 轻量、本地 | 开发测试 |
| **Milvus** | 高性能、可扩展 | 大规模生产环境 |
| **Qdrant** | Rust编写、快速 | 对性能要求高的场景 |

代码示例（使用 Chroma）：
```python
import chromadb

# 初始化客户端
client = chromadb.Client()
collection = client.create_collection("my_docs")

# 添加文档
collection.add(
    documents=["RAG用于检索增强", "向量数据库存储embedding"],
    metadatas=[{"source": "doc1"}, {"source": "doc2"}],
    ids=["id1", "id2"]
)

# 检索相似文档
results = collection.query(
    query_texts=["什么是RAG?"],
    n_results=2
)
print(results['documents'])
```

### 2.4 检索策略（Retrieval Strategies）

**1. 密集检索（Dense Retrieval）**
- 使用向量相似度
- 理解语义含义
- 适合模糊查询

**2. 稀疏检索（Sparse Retrieval）**
- 传统关键词匹配（BM25）
- 精确匹配能力强
- 适合专有名词查询

**3. 混合检索（Hybrid）**
```python
# 结合两种方法
dense_results = vector_search(query)      # 70%权重
sparse_results = keyword_search(query)    # 30%权重
final_results = merge_and_rerank(dense_results, sparse_results)
```

**4. 重排序（Reranking）**
```python
# 先粗筛（检索100条）
candidates = retrieve_top_100(query)

# 再精排（使用更强模型选出前3条）
from sentence_transformers import CrossEncoder
reranker = CrossEncoder('cross-encoder/ms-marco-MiniLM-L-12-v2')

scores = reranker.predict([(query, doc) for doc in candidates])
top_3 = get_top_k(candidates, scores, k=3)
```

---

## 三、RAG 完整实现示例

### 3.1 基础 RAG 系统

```python
from langchain.document_loaders import PyPDFLoader
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain.embeddings import OpenAIEmbeddings
from langchain.vectorstores import Chroma
from langchain.chat_models import ChatOpenAI
from langchain.chains import RetrievalQA

# 步骤1: 加载文档
loader = PyPDFLoader("company_handbook.pdf")
documents = loader.load()

# 步骤2: 分块
text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=1000,
    chunk_overlap=100
)
chunks = text_splitter.split_documents(documents)

# 步骤3: 向量化并存储
embeddings = OpenAIEmbeddings()
vectorstore = Chroma.from_documents(chunks, embeddings)

# 步骤4: 创建检索器
retriever = vectorstore.as_retriever(
    search_type="similarity",
    search_kwargs={"k": 3}  # 返回最相关的3个文档块
)

# 步骤5: 创建问答链
llm = ChatOpenAI(model="gpt-4", temperature=0)
qa_chain = RetrievalQA.from_chain_type(
    llm=llm,
    retriever=retriever,
    return_source_documents=True  # 返回来源文档
)

# 步骤6: 提问
query = "公司的年假政策是什么？"
result = qa_chain({"query": query})

print("答案:", result['result'])
print("来源:", result['source_documents'])
```

### 3.2 高级 RAG 优化技术

**1. 查询改写（Query Rewriting）**
```python
# 用户问题可能表达不清晰，先让LLM改写
def rewrite_query(original_query):
    prompt = f"""
    将以下用户问题改写得更清晰、更适合检索：
    原问题: {original_query}
    改写后:
    """
    return llm.predict(prompt)

# 使用
user_question = "那个东西怎么用啊"  # 模糊问题
better_query = rewrite_query(user_question)  # "如何使用该产品的操作步骤"
```

**2. 假设性文档嵌入（HyDE）**
```python
# 先让LLM生成一个假设的答案，再用这个答案去检索
def hyde_retrieval(query):
    # 生成假设答案
    hypothetical_doc = llm.predict(f"请回答: {query}")
    
    # 用假设答案的向量去检索（而不是用问题）
    results = vectorstore.similarity_search(hypothetical_doc, k=5)
    return results
```

**3. 元数据过滤**
```python
# 只检索特定类型的文档
retriever = vectorstore.as_retriever(
    search_kwargs={
        "k": 5,
        "filter": {
            "source": "financial_reports",  # 只查财务报告
            "year": 2024                     # 只查2024年的
        }
    }
)
```

---

## 四、RAG 应用场景

### 4.1 企业知识库问答

**场景**：员工查询公司政策、流程、技术文档
```
知识源: 员工手册、技术文档、历史邮件
用户: "如何申请远程办公？"
RAG: 检索HR政策 → 生成详细步骤和注意事项
```

### 4.2 客户服务机器人

**场景**：7x24小时回答客户问题
```
知识源: 产品说明书、FAQ、客服记录
用户: "这款手机支持5G吗？"
RAG: 检索产品参数 → 给出准确答案并推荐套餐
```

### 4.3 法律/医疗辅助

**场景**：辅助专业人员决策
```
知识源: 法律条文、判例、医学文献
律师: "类似案件的判决先例有哪些？"
RAG: 检索相关判例 → 总结关键点和判决依据
```

### 4.4 代码助手

**场景**：帮助开发者理解和使用代码库
```
知识源: 代码仓库、API文档、技术博客
开发者: "如何使用我们的支付API？"
RAG: 检索API文档和示例代码 → 生成完整使用教程
```

### 4.5 个人知识管理

**场景**：管理笔记、书籍、文章
```
知识源: Notion笔记、PDF书籍、收藏的文章
你: "我之前看的关于时间管理的方法是什么？"
RAG: 检索相关笔记 → 总结要点
```

---

## 五、RAG vs 微调（Fine-tuning）

| 对比维度 | RAG | 微调 |
|---------|-----|------|
| **知识更新** | 实时更新，添加文档即可 | 需要重新训练 |
| **成本** | 低（只需存储和检索） | 高（需要GPU训练） |
| **准确性** | 基于真实文档，可追溯 | 知识融入模型，不可追溯 |
| **适用场景** | 知识频繁变化、需要引用来源 | 改变模型风格、行为模式 |
| **响应速度** | 稍慢（需要检索） | 快（直接生成） |

**最佳实践**：两者结合使用
- 微调：调整模型的回答风格、格式
- RAG：提供最新的事实知识

---

## 六、RAG 常见问题和解决方案

### 6.1 检索不准确

**问题**：检索到的文档不相关

**解决方案**：
```python
# 1. 改进分块策略
- 使用语义分块而非固定长度
- 增加上下文重叠

# 2. 使用更好的 Embedding 模型
embeddings = OpenAIEmbeddings(model="text-embedding-3-large")

# 3. 混合检索
from langchain.retrievers import EnsembleRetriever
ensemble = EnsembleRetriever(
    retrievers=[vector_retriever, bm25_retriever],
    weights=[0.7, 0.3]
)

# 4. 添加重排序
from langchain.retrievers import ContextualCompressionRetriever
from langchain.retrievers.document_compressors import CohereRerank

compressor = CohereRerank()
compression_retriever = ContextualCompressionRetriever(
    base_retriever=retriever,
    base_compressor=compressor
)
```

### 6.2 答案产生幻觉

**问题**：模型编造不在文档中的信息

**解决方案**：
```python
# 在提示词中明确要求
prompt_template = """
基于以下文档回答问题。如果文档中没有相关信息，请明确说"文档中未提供此信息"。

文档内容:
{context}

问题: {question}

回答:
"""

# 使用温度参数降低创造性
llm = ChatOpenAI(temperature=0)  # 0表示最确定性的输出
```

### 6.3 性能优化

**问题**：检索和生成速度慢

**解决方案**：
```python
# 1. 缓存常见查询
from langchain.cache import InMemoryCache
langchain.llm_cache = InMemoryCache()

# 2. 批量处理
queries = ["问题1", "问题2", "问题3"]
embeddings = model.embed_documents(queries)  # 一次性处理

# 3. 异步检索
import asyncio

async def async_retrieve(query):
    return await vectorstore.asimilarity_search(query)

results = await asyncio.gather(*[async_retrieve(q) for q in queries])

# 4. 使用更快的向量数据库
# Qdrant (Rust) > Milvus > Chroma
```

---