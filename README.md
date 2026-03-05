# RAG_fang
Demonstrate some simple practical operations of RAG projects
# RAG 项目说明

本项目实现了一条面向公司年报的 RAG 流水线：从 PDF 解析、表格序列化、页面规整、文本分块、向量库/ BM25 索引构建，到问题检索与答案生成。核心流程集中在 [pipeline.py](file:///e:/LLM%20Development/RAG/RAG_practice/RAG-Challenge-2-main/src/pipeline.py) 与 [questions_processing.py](file:///e:/LLM%20Development/RAG/RAG_practice/RAG-Challenge-2-main/src/questions_processing.py)。

**核心流程**
- PDF 解析：基于 Docling 进行 OCR 与表格结构化，输出结构化 JSON
- 报告规整：将复杂 JSON 规整成每页 Markdown 风格文本
- 分块与索引：按页切分成 chunk，构建 FAISS 向量库与 BM25 索引
- 检索与问答：向量检索/混合重排 + 结构化答案输出
- 多公司比较：将比较问题拆解为单公司问题并汇总

**模块说明**
- [pdf_parsing.py](file:///e:/LLM%20Development/RAG/RAG_practice/RAG-Challenge-2-main/src/pdf_parsing.py)：Docling PDF 解析与结构化输出，生成 metainfo/content/tables/pictures
- [tables_serialization.py](file:///e:/LLM%20Development/RAG/RAG_practice/RAG-Challenge-2-main/src/tables_serialization.py)：表格序列化（LLM 生成上下文无关的信息块）
- [parsed_reports_merging.py](file:///e:/LLM%20Development/RAG/RAG_practice/RAG-Challenge-2-main/src/parsed_reports_merging.py)：页面文本规整、表格/列表/脚注合并、Markdown 导出
- [text_splitter.py](file:///e:/LLM%20Development/RAG/RAG_practice/RAG-Challenge-2-main/src/text_splitter.py)：基于 tiktoken 的按页分块，可插入序列化表格块
- [ingestion.py](file:///e:/LLM%20Development/RAG/RAG_practice/RAG-Challenge-2-main/src/ingestion.py)：FAISS 向量库与 BM25 索引构建
- [retrieval.py](file:///e:/LLM%20Development/RAG/RAG_practice/RAG-Challenge-2-main/src/retrieval.py)：向量检索、BM25 检索与父页面检索
- [reranking.py](file:///e:/LLM%20Development/RAG/RAG_practice/RAG-Challenge-2-main/src/reranking.py)：LLM 重排器（可与向量分数融合）
- [api_requests.py](file:///e:/LLM%20Development/RAG/RAG_practice/RAG-Challenge-2-main/src/api_requests.py)：OpenAI/IBM/Gemini/DashScope 请求封装与结构化输出解析
- [api_request_parallel_processor.py](file:///e:/LLM%20Development/RAG/RAG_practice/RAG-Challenge-2-main/src/api_request_parallel_processor.py)：并发 API 请求处理与限流
- [questions_processing.py](file:///e:/LLM%20Development/RAG/RAG_practice/RAG-Challenge-2-main/src/questions_processing.py)：问题处理、检索、结构化答案、提交格式化
- [pipeline.py](file:///e:/LLM%20Development/RAG/RAG_practice/RAG-Challenge-2-main/src/pipeline.py)：流水线编排、配置预设与示例执行
- [prompts.py](file:///e:/LLM%20Development/RAG/RAG_practice/RAG-Challenge-2-main/src/prompts.py)：结构化输出与重排提示词

**Pipeline 核心阶段**
- parse_pdf_reports：解析 PDF 为结构化 JSON
- serialize_tables：表格序列化
- merge_reports：规整报告为每页文本
- export_reports_to_markdown：导出 Markdown
- chunk_reports：按页分块并可插入表格块
- create_vector_dbs / create_bm25_db：构建检索索引
- process_questions：问题检索与回答

**运行配置（RunConfig）**
- use_serialized_tables：是否使用序列化表格
- parent_document_retrieval：是否返回父页面
- use_vector_dbs / use_bm25_db：检索索引开关
- llm_reranking：是否启用 LLM 重排
- top_n_retrieval / llm_reranking_sample_size：检索与重排规模
- api_provider / answering_model：LLM 与嵌入服务选择
- full_context：是否直接使用全文

**CLI 用法（main.py）**
```bash
python main.py --help

python main.py download-models
python main.py parse-pdfs --parallel --chunk-size 2 --max-workers 10
python main.py serialize-tables --max-workers 10
python main.py process-reports --config ser_tab
python main.py process-questions --config max_nst_o3m
```

**直接运行 pipeline.py**
```bash
python .\src\pipeline.py
```

**环境变量**
- OPENAI_API_KEY
- DASHSCOPE_API_KEY
- GEMINI_API_KEY
- IBM_API_KEY
- JINA_API_KEY

**产出数据格式（简要）**
- 解析报告 JSON：metainfo、content（按页）、tables（带 markdown/html/serialized）、pictures
- 规整报告 JSON：content.pages（每页文本）
- 分块报告 JSON：content.chunks（chunk 文本、页码、token 长度）

**说明**
- 表格序列化为可检索的上下文无关信息块，可用于提升跨页/表格理解能力
- 比较类问题会被拆解为单公司问题，再汇总生成最终比较答案
