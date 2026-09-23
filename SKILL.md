---
name: minwon-classifier
description: Korean-language, evidence-driven 민원 분류 스킬 — 신규 민원 접수 시 유사 이력 조회→근거 표 작성→증거 기반 다수결(≥60%)로 최종 분류 확정 및 SLA 기한 산출까지 수행. 결과 산출물에는 근거 표·검색쿼리/필터·결정 요약·예외/한계 메모를 반드시 포함. 예외/리트라이·품질 체크리스트를 내장.
---

---
name: minwon-classifier
description: Korean-language, evidence-driven 민원 분류 스킬 — 신규 민원 접수 시 유사 이력 조회→근거 표 작성→증거 기반 다수결(≥60%)로 최종 분류 확정 및 SLA 기한 산출까지 수행. 결과 산출물에는 근거 표·검색쿼리/필터·결정 요약·예외/한계 메모를 반드시 포함. 예외/리트라이·품질 체크리스트를 내장.
---

# minwon-classifier (개정판)

목적: 자유서술형 민원(국민신문고/제보/문의 포함)을 증거 기반으로 자동 분류하고, 소관부서를 지정하며, 긴급도와 처리기한(SLA)을 산출합니다. 모든 결과에는 근거 신호와 한국어 설명, 신뢰도(confidence), ‘근거 표(evidence_table)’가 포함됩니다.

개정 요약(중요 변경)
- 필수 절차 강제: (1) 준비/입력 정리 → (2) 유사 민원 조회(5~20건) → (3) 근거 사례 3~5건 선정 → (4) 근거 표 작성 → (5) 최종 분류(≥60% 다수결·상충 시 상위 카테고리 임시 분류+검토요청) → (6) SLA 산출 및 출처 기록 → (7) 산출물 스키마 고정 → (8) 품질 체크리스트 → (9) 예외 처리/리트라이.
- 산출물 스키마 확장: evidence_table, search, decision_summary, sla, limitations_or_exceptions 필드 추가(필수).
- 로그/리트라이: 유사 이력 조회와 근거 표 작성은 반드시 로그에 남기고 실패 시 최대 2회 리트라이, 예외 경로 명시.

--------------------------------
전체 SOP (증거 기반 분류 플로우)
--------------------------------
1) 준비/입력 정리
- 입력: 한국어 자유서술 민원 본문(필수), 제목/기관/분야 태그/메타(있으면 활용).
- 핵심 키워드 추출, 기관/분야 태그 보정, 예상 유형 후보(불편/신고/질의/건의) 초안 도출.
- 조회 필터 설정: 기간(기본 12~24개월), 유사도 기준(예: ≥0.70), 제외 조건(스팸/중복/다른 관할), 정렬(유사도·최신순).

2) 유사 민원 조회
- 이력DB/검색도구로 유사 이력 5~20건 확보(유사도 우선, 최신순 보조 정렬). 5건 미만이면 부족.
- 결과 부족/품질 저하 시 재검색(최대 2회): 기간 확장(최대 36개월), 동의어/태그 확장, 오타 교정, 불용어 제거.
- 각 시도별 검색 쿼리·필터·건수·정렬을 search.queries/filters/retry_count에 기록.

3) 근거 사례 선정(3~5건)
- 선정 기준: (a) 규정/가이드 정합성, (b) 최근성, (c) 맥락 일치도.
- 상충 시 우선순위: 상위 규정/최신성 우선, 맥락 완전 일치 가중.
- 선정된 각 사례는 evidence_table에 등재.

4) 근거 표 작성(필수)
- 필드: 이력ID(history_id), 접수일(received_date), 유형(type), 판단 요지(rationale_summary·1~2문장), 참고 링크/문서ID(reference), 선택 사유 또는 유사도(selection_reason_or_similarity).
- 링크 유효성 검사(HTTP 200/문서 존재) 후 기록. 불가 시 대체 식별자(DOC_ID)와 검증 실패 사유 기재.

5) 최종 분류 결정(증거 다수결)
- 선정 근거의 공통 유형(카테고리)이 60% 이상이면 해당 유형으로 확정.
- 일치율이 낮거나 상충 시: 상위 카테고리로 임시 분류하고 검토 요청/에스컬레이션 수행(decision_summary 및 limitations_or_exceptions에 명시).
- 분류 사유를 evidence_table 행과 연결해 2~3문장으로 요약(decision_summary).

6) 처리 기한 판단(SLA)
- 확정 유형/부서에 해당하는 SLA 매핑을 참조해 기한(deadline)을 산출.
- SLA 출처(정책 문서/가이드 링크)를 sla.source_link로 기록.

7) 결과 생성(스키마 고정)
- 아래 ‘출력 스키마(JSON)’에 정확히 맞춰 산출. 여분 키 금지, 필수 키 누락 금지.

8) 품질 체크리스트(출력 전 자동 점검)
- 유사 이력 조회 수행·로그 기록됨?
- evidence_table 존재·필수 필드 충족·건수(가능 시 3~5건)·링크 유효?
- 결정-근거 정합성(다수결 규칙 적용)·decision_summary 근거 연결 OK?
- SLA deadline·source_link 기재?
- signals·rationale·confidence·fallback 규칙 충족, PII 미노출?
- 예외/제한 사항 기록(limitations_or_exceptions)?

9) 예외 처리/폴백
- 유사 사례 없음: 키워드/동의어 확장 등 2회 재검색 후 ‘유사 이력 없음’으로 evidence_table에 명시, 전문가 검토 요청(fallback=true 가능).
- 접근 실패(네트워크/권한): 즉시 1회 재시도→대체 소스 조회→실패 사유를 limitations_or_exceptions에 기록.
- 근거 상충: 정책팀 확인 태스크 생성(내부 티켓 참조), 상위 카테고리 임시 분류 및 에스컬레이션 명시.

----------------------
세부 규칙(변경·유지)
----------------------
A) 개인정보(PII) 최소화(유지)
- 주민등록번호/전화번호/정확 주소/계좌 등은 출력 금지. rationale/signals에는 “개인정보 포함” 등 일반화 표기만.

B) 신호 추출·점수화(유지·보완)
- 카테고리 신호: 불편/신고/질의/건의 관련 키워드·행동표현.
- 부서 신호: 키워드 사전(아래 표) 매칭(동의어 포함) 가중.
- 긴급도 신호·가중치(내부 5-티어):
  • 생명·신체 위험/화재·가스·붕괴·침수·폭행: +0.50
  • 대규모 전면 중단(전력/수도/가스/교통): +0.35 / 부분 중단: +0.25
  • 법정기한 임박 D≤1: +0.30 / D≤3: +0.20 / D≤7: +0.10
  • 취약계층 직접 위험: +0.10
  • 장소·시간(심야·통학로·병원 인접 등): +0.05~0.10
  • 반복 제기/재발: +0.05
  • 감쇄: “현장 조치 완료” −0.20(잔여 위험 있으면 미감쇄), “추가정보요청만” −0.10
- 티어 기준: Critical≥0.90, High≥0.75, Medium≥0.55, Low≥0.35, Minimal<0.35 → 출력 매핑: 긴급/보통/낮음.

C) 결정·타이브레이커(유지)
- 카테고리: ‘위법/신고/처벌/단속’ 우세→신고, ‘문의/알고 싶다/방법’→질의, ‘제안/개선’→건의, 그 외 이용상 문제→불편. ‘신고’ 신호 동시 존재 시 신고 우선.
- 긴급도: 즉시 위험/법정기한 임박 시 상향. ‘조치 완료’ 신호 확인 시 한 단계 하향(단 잔여 위험 존재 시 하향 금지).
- 부서: 안전·재난 급박성 신호 있으면 안전재난과 1순위. 다건 이슈는 공공위해도 높은 주제 우선. 모호하면 “소관부서 검토 필요”.

D) 신뢰도·폴백(유지)
- overall confidence = min(카테고리 점수, 긴급도 확신도, 부서 점수) − 품질감점(최대 0.1).
- 기준: 0.9↑ 매우 확실, 0.7~0.89 보통, 0.5~0.69 낮음, 0.5 미만 불충분.
- fallback=true: confidence<0.5, 주요 신호 충돌, 스팸/욕설/반어법 의심, 입력 극단적 축약.

E) 소관부서 매핑(예시, 환경별 교체)
- 교통과: 도로, 보행, 신호/신호등, 주정차/불법주차, 버스, 택시, 자전거, 과속, 횡단보도, 포트홀
- 청소행정과: 쓰레기, 무단투기, 재활용, 음식물, 분리수거, 청소, 폐기물, 적치물
- 공원녹지과: 공원, 가로수/수목, 화단, 잔디, 놀이터, 산책로, 녹지, 조경
- 환경위생과: 소음, 악취, 진동, 위생, 방역, 해충, 배출, 미세먼지, 유흥업소, 식품위생
- 상하수도과: 상수도, 하수도, 배수, 누수, 단수, 수질, 맨홀, 정화조, 빗물
- 건축과: 건축허가, 불법건축물, 증축, 용도변경, 건축물대장, 공사소음, 옥상 증축
- 복지정책과: 복지, 생활지원, 기초수급, 장애, 돌봄, 노인, 보육, 바우처
- 세무과: 세금, 과태료, 과세, 납부, 체납, 고지서, 감면, 환급
- 안전재난과: 안전, 재난, 재해, 화재, 가스, 붕괴, 침수, 낙석, 도로함몰, 폭행, 전기/정전
- 민원여권과: 민원접수, 일반문의, 여권, 제증명, 상담, 안내, 담당자
메모: 배포 전 현행 조직도/권한대장과 동기화.

--------------------
출력 스키마(JSON)
--------------------
반드시 아래 JSON 구조로만 출력합니다(키·값 고정, 여분 키 금지).

- category: "불편" | "신고" | "질의" | "건의"
- department: string (최적 부서명, 모호 시 "소관부서 검토 필요")
- urgency: "긴급" | "보통" | "낮음"
- confidence: number 0..1
- signals: { "category": string[], "department": string[], "urgency": string[] }
- rationale: 2~4문장 한국어 요약(PII 금지)
- fallback: boolean (정보 부족·충돌 시 true)
- evidence_table: [ { "history_id": string, "received_date": "YYYY-MM-DD", "type": string, "rationale_summary": string, "reference": string, "selection_reason_or_similarity": string } ]
- search: { "queries": string[], "filters": { "period": string, "similarity_threshold": number, "excludes": string[], "sort": string }, "retrieved_count": number, "retry_count": number }
- decision_summary: string (2~3문장, evidence_table 참조ID 연결)
- sla: { "policy_name": string, "deadline": string, "source_link": string }
- limitations_or_exceptions: string

JSON 템플릿:
{
  "category": "불편",
  "department": "소관부서 검토 필요",
  "urgency": "보통",
  "confidence": 0.0,
  "signals": {
    "category": [],
    "department": [],
    "urgency": []
  },
  "rationale": "",
  "fallback": false,
  "evidence_table": [
    {
      "history_id": "",
      "received_date": "",
      "type": "",
      "rationale_summary": "",
      "reference": "",
      "selection_reason_or_similarity": ""
    }
  ],
  "search": {
    "queries": [],
    "filters": { "period": "", "similarity_threshold": 0.7, "excludes": [], "sort": "similarity_desc,date_desc" },
    "retrieved_count": 0,
    "retry_count": 0
  },
  "decision_summary": "",
  "sla": { "policy_name": "", "deadline": "", "source_link": "" },
  "limitations_or_exceptions": ""
}

-------------
예시(output)
-------------
입력: "횡단보도 앞 불법주정차 차량 때문에 보행자가 도로로 내려가 통행합니다. 신속히 조치 바랍니다."
출력:
{
  "category": "신고",
  "department": "교통과",
  "urgency": "긴급",
  "confidence": 0.88,
  "signals": {
    "category": ["불법주정차", "조치 요청", "신고 성격"],
    "department": ["횡단보도/주정차→교통과"],
    "urgency": ["보행 안전 위험", "즉시 조치 필요"]
  },
  "rationale": "불법주정차로 보행 안전 위험이 커 신고로 분류합니다. 주정차·보행 안전 관련은 교통과 소관입니다. 즉시 위험 요소로 긴급으로 판단합니다.",
  "fallback": false,
  "evidence_table": [
    {"history_id": "T-2025-0412", "received_date": "2025-04-12", "type": "신고", "rationale_summary": "횡단보도 앞 불법주정차 단속 요청, 보행 위험 강조", "reference": "https://.../cases/T-2025-0412", "selection_reason_or_similarity": "유사도 0.83"},
    {"history_id": "T-2026-0108", "received_date": "2026-01-08", "type": "신고", "rationale_summary": "학교 앞 불법주정차로 통학로 위험, 즉시 조치 필요", "reference": "https://.../cases/T-2026-0108", "selection_reason_or_similarity": "유사도 0.79"},
    {"history_id": "T-2024-1119", "received_date": "2024-11-19", "type": "신고", "rationale_summary": "보행로 인접 불법주정차 반복 발생, 단속 요청", "reference": "DOC:CASE-1119", "selection_reason_or_similarity": "규정 정합성 높음"}
  ],
  "search": {
    "queries": ["불법주정차 보행 위험", "횡단보도 앞 불법주차 단속"],
    "filters": {"period": "2024-01-01..2026-09-23", "similarity_threshold": 0.75, "excludes": ["타 지자체"], "sort": "similarity_desc,date_desc"},
    "retrieved_count": 14,
    "retry_count": 1
  },
  "decision_summary": "근거표 T-2025-0412/T-2026-0108/T-2024-1119 중 3/3가 신고로 일치(≥60%). 보행 안전 위험 신호로 긴급 판단, 교통과 라우팅.",
  "sla": {"policy_name": "불법주정차 민원 처리 기준", "deadline": "D+3 영업일", "source_link": "https://.../policy/parking-enforcement"},
  "limitations_or_exceptions": "사진/위치 좌표 미첨부 — 현장 확인 시 추가 자료 요청 필요"
}

----------------
도구 연동 가이드
----------------
- 이력DB 조회 래퍼(history_db.search): input {query: string[], filters: {period, similarity_threshold, excludes, sort}} → output {items:[{id, date, type, summary, url, sim}]}
- 링크 검증기(link_validator.check): input {url} → output {ok: boolean, status: number}
- SLA 레퍼런스 로더(sla_loader.lookup): input {category, department} → output {policy_name, deadline, source_link}
메모: 실제 함수/엔드포인트 명은 운영 환경에 맞게 매핑. 조회/검증/로더 실패 시 최대 2회 리트라이, 실패 사유를 limitations_or_exceptions에 기록.

-------------
테스트·모니터링
-------------
- 유사 이력 확보율: 입력당 ≥5건(가능 시) 확보 비율
- 근거 표 충족률: 필수 필드 100% 충족, 링크 유효율 ≥98%
- 다수결 일치율: 60% 규칙 적용 정확도 모니터링
- SLA 기재율: deadline+source_link 포함 ≥99%
- 분류 품질: 카테고리 macro F1 ≥0.85, ‘신고’ 정밀도 ≥0.85, ‘긴급’ 재현율 ≥0.9

사용 팁
- 항상 근거 중심으로 설명하고, 모호하면 폴백과 에스컬레이션을 명시하십시오.
- 부서 명칭은 운영기관 환경에 맞춰 수시로 갱신하십시오.
