# chatbot
# mcp-server.py
from mcp.server.fastmcp import MCPServer
from mcp.server.tool import Tool, ToolResult
import os

# ========== 配置 ==========
CODEBASE_PATH = "./codebase"
KEYWORDS = {
    "代码", "实现", "函数", "方法", "类", "逻辑", "哪里",
    "定义", "怎么写", "bug", "报错", "修复", "源码", "原理",
    "readme", "入口", "调用", "报错"
}

# Prompt 模板
PROMPT_TEMPLATE = """
你是一个代码助手，请结合以下项目代码回答问题。

# 项目代码摘要：
{code_summary}

# 用户问题：
{question}

# 请据此作答。
""".strip()

# ========== MCP Server 实例 ==========
mcp_server = MCPServer(
    name="code-prompt-server",
    description="仅在用户询问代码相关问题时被调用",
)

# ========== 工具定义 ==========
@mcp_server.tool(
    prompt_triggers=list(KEYWORDS)  # 👈 关键：告诉 Cline 哪些词会触发调用
)
def generate_code_aware_prompt(question: str) -> ToolResult:
    """为代码相关问题生成增强 Prompt"""
    if not question.strip():
        return ToolResult(error="问题不能为空")

    # 扫描代码仓（略去细节，见前文）
    code_summary = []
    if os.path.exists(CODEBASE_PATH):
        for root, _, files in os.walk(CODEBASE_PATH):
            for file in files:
                if file.endswith((".py", ".js", ".ts", ".go")):
                    filepath = os.path.join(root, file)
                    rel_path = os.path.relpath(filepath, CODEBASE_PATH)
                    try:
                        with open(filepath, "r", encoding="utf-8") as f:
                            content = f.read().strip()[:1000]  # 截断大文件
                        code_summary.append(f"=== {rel_path} ===\n{content}")
                    except:
                        pass

    final_prompt = PROMPT_TEMPLATE.format(
        code_summary="\n\n".join(code_summary) if code_summary else "（无代码）",
        question=question,
    )
    return ToolResult(content=final_prompt)
