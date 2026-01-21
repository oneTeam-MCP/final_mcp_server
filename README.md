# 상명대학교 특화 MCP 서버 (SMUS)

상명대 환경에 특화된 **MCP(Model Context Protocol) 서버**입니다.  
백엔드는 **FastMCP**(Python)로 작성했고, 배포/호스팅은 **Smithery**를 사용했습니다.  
공지/학사일정/식단/시험 등 핵심 정보를 MCP 툴로 제공하며, 자연어로 조회·요약·일정 등록/삭제가 가능합니다.

- **공식 배포(내 서버):** https://smithery.ai/server/@hwruchan/smus  
- **레포지토리:** https://github.com/oneTeam-MCP/final_mcp_server

---

## 왜 FastMCP 인가

- **Pythonic & 빠른 개발**: 데코레이터(`@mcp.tool`, `@mcp.prompt`)만으로 MCP 툴/프롬프트를 노출  
- **MCP 표준 준수**: STDIO/HTTP(SSE) 전송을 포함한 MCP 스펙을 간단히 구현  
- **운영 친화**: Uvicorn/Starlette와 궁합이 좋아 로컬·컨테이너 배포가 수월

> 참고 문서  
> - FastMCP 공식 사이트: https://gofastmcp.com/  
> - FastMCP GitHub: https://github.com/jlowin/fastmcp  
> - MCP 공식 문서(빌드 가이드): https://modelcontextprotocol.io/docs/develop/build-server

---

## 왜 Smithery 인가

- **원클릭 배포**: MCP 서버를 URL/명령 기반으로 연결해 바로 사용  
- **클라이언트 연결 용이**: MCP URL 하나로 다양한 에이전트/클라이언트에서 사용 가능  
- **서버 카탈로그**: 공개 서버로 배포해 공유/발견이 쉬움

> 참고 문서  
> - Smithery: https://smithery.ai/  
> - Connect to MCPs: https://smithery.ai/docs/use/connect  
> - Deploy 가이드(예시): https://smithery.ai/docs/build/deployments/typescript

---

## 아키텍처 한눈에

[Frontend / MCP Client (예: Smithery, Chat UI)]
│ JSON-RPC over HTTP (streamable-http)
▼
[FastMCP Server (Python, Uvicorn/Starlette)]
├─ @prompt : default_prompt (KST 앵커링, 도구 호출 규칙)
├─ @tool : now_kr, query_smu_* (meals/notices/exam/schedule), add/delete schedule ...
└─ RDS(MySQL) : PyMySQL로 실데이터 조회/쓰기


- Transport: **HTTP(JSON-RPC, streamable-http)**
- Timezone: **KST(Asia/Seoul)** 기준 앵커링
- DB: **RDS MySQL** (환경변수로 접속 정보 관리)

---

## 핵심 기능(요약)

- **통합 조회**: 공지/학사일정/식단/시험을 한 인터페이스로  
- **자연어 상호작용**: “오늘 점심 뭐야?”, “중간고사 일정 찾아줘”  
- **일정 관리**: 개인 일정 등록/삭제 (`add_smu_schedule_structured`, `delete_smu_schedule_by_content`)  
- **안전장치 프롬프트**: 날짜 의존 질문 전 `now_kr` 호출, 띄어쓰기 변형 자동 고려

---

## 빠른 시작

```bash
git clone https://github.com/oneTeam-MCP/final_mcp_server.git
cd final_mcp_server
pip install -r requirements.txt

# 환경변수 (예시) — 운영은 .env / Secret Manager 권장
export DB_HOST=your-host.rds.amazonaws.com
export DB_USER=your-user
export DB_PASSWORD=your-password
export DB_NAME=oneteam_DB
export DB_PORT=3306
export PORT=8081

python server.py
# → http://0.0.0.0:8081 로 MCP HTTP 서버가 뜹니다.

Smithery에서 사용

위 서버를 Smithery에 등록하거나, 공개 배포본(https://smithery.ai/server/@hwruchan/smus)을
 바로 연결해 사용할 수 있습니다.
