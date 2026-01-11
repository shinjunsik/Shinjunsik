# 신준식 (Shin Jun-sik)

## Introduction
안녕하세요, 신입 백엔드 개발자 신준식입니다.  
검색 도메인에서 성능(지연 최소화)과 안정성(실패 흡수)을 기준으로 트레이드오프를 고민하고,  
외부 의존성(Embedding / Rerank / LLM)이 실패하더라도 검색 결과가 비지 않도록 **fallback / retry / DLQ** 중심으로 안정화한 경험이 있습니다.

---

## Tech Stack
![Java](https://camo.githubusercontent.com/22381621c0d51d58b4e20e7508f09a880d8fe08f642844eb90e98e56447c12a9/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f4a6176612d4533344632363f7374796c653d666f722d7468652d6261646765266c6f676f3d4a617661266c6f676f436f6c6f723d7768697465)
![SpringBoot](https://camo.githubusercontent.com/cbaf74ed02a1b5fdf3200e3352fa85d8871f9afa2daa0499f96656f050504596/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f737072696e67253230626f6f742d3644423333463f7374796c653d666f722d7468652d6261646765266c6f676f3d737072696e67626f6f74266c6f676f436f6c6f723d7768697465)
![MySQL](https://camo.githubusercontent.com/72e9a5b8804402af8b88e058012160ee3c12aa6782325a51a87ec7870c99329d/687474703a2f2f696d672e736869656c64732e696f2f62616467652f4d7953514c2d3434373941313f7374796c653d666f722d7468652d6261646765266c6f676f3d4d7953514c266c6f676f436f6c6f723d7768697465)
![Elasticsearch](https://camo.githubusercontent.com/ac1893d25c2f39c0518d939fb9f98b6c71e5d1be676c140a06767b01fda49dd3/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f456c61737469637365617263682d3030353537312e7376673f7374796c653d666f722d7468652d6261646765266c6f676f3d656c6173746963736561726368266c6f676f436f6c6f723d7768697465)
![RabbitMQ](https://camo.githubusercontent.com/661f1fb62b66d00114052a4c749e4ab1a8aeef5701b0e90f46f4cd186c49d59f/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f7261626269746d712d4646363630303f7374796c653d666f722d7468652d6261646765266c6f676f3d7261626269746d71266c6f676f436f6c6f723d7768697465)
![Redis](https://camo.githubusercontent.com/372d959d6da83adf6e32f9b3d1dd5ac95c7997ceac1958a71bb2d7afd944d79e/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f52656469732d4443333832443f7374796c653d666f722d7468652d6261646765266c6f676f3d5265646973266c6f676f436f6c6f723d7768697465)
![Maven](https://camo.githubusercontent.com/54abcc74b4cd2b272caa349587deae8f8ad612ec0f99764db00fe3a3e57858c7/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f4170616368652532304d6176656e2d4337314133363f7374796c653d666f722d7468652d6261646765266c6f676f3d4170616368652532304d6176656e266c6f676f436f6c6f723d7768697465)

---

## Experience
### NHN Academy Java Backend 개발자 과정 12기
- 기간: 2025.07.28 ~ 2025.12.31  
자바 백엔드 과정을 수료하며 Spring 기반 백엔드 개발 역량을 학습했고,  
팀 프로젝트로 MSA 기반 인터넷 서점을 개발하며 협업/운영 관점의 경험을 쌓았습니다.

---

## Project
### 인터넷 서점 daiso-book (MSA 기반)
- 기간: 2025.11.11 ~ 2025.12.31
- 팀: 7인
- 소개: 일반 검색 + Gemini API 기반 AI 검색(추천 이유 제공)을 지원하는 인터넷 서점 프로젝트

#### 담당 역할: 검색 도메인(Search) 오너
- 검색/AI검색/색인 동기화(Worker)까지 **검색 도메인 End-to-End** 설계 및 구현
- 외부 의존성(Embedding/Rerank/LLM) 장애가 검색 공백으로 이어지지 않도록 **fallback/retry/DLQ** 중심으로 안정화
- Redis 캐시(검색 결과 / 할인정책)를 “성능”뿐 아니라 **정합성**까지 고려해 설계

---

## Highlights (핵심 구현)
- **Hybrid Search**: Elasticsearch BM25 + (조건부) Vector KNN 결합
- **AI Search Pipeline**: Embedding → ES 후보 생성 → Top-N Rerank → Top-M LLM 결과 생성
- **Worker Sync**: RabbitMQ upsert/delete + manual ack + retry(최대 3회) + DLQ 격리
- **Cache / Consistency**
  - AI 검색: Komoran 기반 키워드 추출 + 정렬(가나다순)로 “의미 중심” 캐시 키 구성
  - 할인정책: 5분 주기 갱신 + TTL 10분, 5분 사이 비활성화된 정책은 삭제 반영

---

## Key Decisions (트레이드오프 기반 의사결정)
> 수치 측정을 충분히 남기진 못했지만, **병목 지점/실패 시나리오**를 기준으로 개선 방향을 결정했습니다.  
> 자세한 내용은 Wiki의 Decision Log에 정리했습니다.

### 1) 일반 검색에서 리랭킹 제거
- **Problem**: 일반 검색에 하이브리드 + 리랭킹을 적용하니 응답 지연이 커져 사용자 체감이 나빠짐  
- **Decision**: 일반 검색은 리랭킹을 제거하고, 리랭킹은 AI 검색에만 적용  
- **Why/Result**: 검색 품질 체감은 크게 떨어지지 않으면서, “느려서 답답한” 케이스를 줄여 핵심 경로를 안정화

### 2) 캐시 전략 재정의: 일반 검색 제외, AI 검색만 캐싱
- **Problem**: 일반 검색은 리랭킹 제거 이후 충분히 빠른데, 캐시 도입의 이점이 제한적  
- **Decision**: 일반 검색 캐싱 제외 / AI 검색만 캐싱 적용  
- **Why/Result**: AI 검색은 rerank/LLM 호출로 지연이 커서 캐싱 가치가 높았고, “필요한 곳”에만 적용해 복잡도를 통제

### 3) AI 검색 캐시 키를 ‘의미 기반’으로 설계 (Komoran + 정렬)
- **Problem**: 단순 문자열 캐시는 문장/띄어쓰기/단어 순서가 달라지면 hit가 어려움  
- **Decision**: Komoran으로 핵심 키워드만 추출하고 가나다순으로 정렬해 캐시 키 생성  
- **Why/Result**: 같은 의미의 질문이 형태만 달라도 캐시 적중 가능성을 확보하면서, AI 검색 응답을 더 안정적으로 유지

### 4) 외부 API 실패 시 검색 결과 공백 방지(fallback)
- **Problem**: 외부 API 오류가 발생하면 결과 리스트가 비는 케이스가 발생  
- **Decision**: 각 단계에서 실패하면 “API 호출 직전 후보 리스트”를 fallback으로 사용  
- **Why/Result**: 품질이 일부 저 되더라도 최소 검색 결과는 항상 제공하도록 만들어 사용자 경험을 안정화

### 5) 리랭킹 입력 Top-N 제한(약 50 → 10)
- **Problem**: 하이브리드 후보를 그대로 리랭킹에 보내면 payload가 커져 지연이 커짐  
- **Decision**: 리랭킹 대상 도서를 Top 10권으로 제한  
- **Why/Result**: 체감 품질은 유지되면서, 길게 늘어지는 지연 케이스가 줄어 응답 안정성이 좋아짐

### 6) Gemini 호출 축소(Top 5 + 200자 제한)
- **Problem**: 많은 책에 추천이유를 생성하려다 프롬프트가 커져 429/timeout이 잦아짐  
- **Decision**: 상위 5권만 요청 + 응답 길이 약 200자 제한  
- **Why/Result**: 사용자 체감이 큰 구간에 집중하면서도 실패/지연 빈도를 줄여 AI 기능을 안정화

---

## Links
- [프로젝트 Repo](https://github.com/nhnacademy-be12-daiso)
- [Decision Log (Wiki)](https://github.com/nhnacademy-be12-daiso/booksearch-worker/wiki/Decision-Log)  
  - 검색 성능/안정성 개선 과정(트레이드오프, 적용 이유, 결과)을 기록

---

## Contact
- Email: jun03031@gmail.com
- GitHub: https://github.com/shinjunsik
