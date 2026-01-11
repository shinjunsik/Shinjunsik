# 신준식 (Shin junsik)

## Introduction
안녕하세요.
저는 신입 백엔드 개발자
신준식 입니다.

---
## Tech Stack
![Java](https://camo.githubusercontent.com/22381621c0d51d58b4e20e7508f09a880d8fe08f642844eb90e98e56447c12a9/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f4a6176612d4533344632363f7374796c653d666f722d7468652d6261646765266c6f676f3d4a617661266c6f676f436f6c6f723d7768697465) ![SpringBoot](https://camo.githubusercontent.com/cbaf74ed02a1b5fdf3200e3352fa85d8871f9afa2daa0499f96656f050504596/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f737072696e67253230626f6f742d3644423333463f7374796c653d666f722d7468652d6261646765266c6f676f3d737072696e67626f6f74266c6f676f436f6c6f723d7768697465) ![Mysql](https://camo.githubusercontent.com/72e9a5b8804402af8b88e058012160ee3c12aa6782325a51a87ec7870c99329d/687474703a2f2f696d672e736869656c64732e696f2f62616467652f4d7953514c2d3434373941313f7374796c653d666f722d7468652d6261646765266c6f676f3d4d7953514c266c6f676f436f6c6f723d7768697465) ![Elasticsearch](https://camo.githubusercontent.com/ac1893d25c2f39c0518d939fb9f98b6c71e5d1be676c140a06767b01fda49dd3/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f456c61737469637365617263682d3030353537312e7376673f7374796c653d666f722d7468652d6261646765266c6f676f3d656c6173746963736561726368266c6f676f436f6c6f723d7768697465) ![RabbitMQ](https://camo.githubusercontent.com/661f1fb62b66d00114052a4c749e4ab1a8aeef5701b0e90f46f4cd186c49d59f/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f7261626269746d712d4646363630303f7374796c653d666f722d7468652d6261646765266c6f676f3d7261626269746d71266c6f676f436f6c6f723d7768697465) ![Redis](https://camo.githubusercontent.com/372d959d6da83adf6e32f9b3d1dd5ac95c7997ceac1958a71bb2d7afd944d79e/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f52656469732d4443333832443f7374796c653d666f722d7468652d6261646765266c6f676f3d5265646973266c6f676f436f6c6f723d7768697465) ![Maven](https://camo.githubusercontent.com/54abcc74b4cd2b272caa349587deae8f8ad612ec0f99764db00fe3a3e57858c7/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f4170616368652532304d6176656e2d4337314133363f7374796c653d666f722d7468652d6261646765266c6f676f3d4170616368652532304d6176656e266c6f676f436f6c6f723d7768697465) 

---

## Experience
### NHN Academy Java Backend 개발자 과정 12기
2025.07.28 ~ 2025.12.31
</br>
자바 백엔드 과정 경험을 통해 자바 백엔드에 대한 이해를 높이고 인터넷 서점 도서 검색 개발을 통해 직무 경험을 쌓았습니다.

---
## Projects
### 인터넷 서점 'daiso-book'
기간: 2025.11.11 ~ 2025.12.31
</br>
NHN Academy에서 진행한 7인 팀 프로젝트 입니다.
</br>
MSA 구조를 기반으로 인터넷 서점을 만들었습니다.
</br>
일반검색과 Gemini API를 활용해 도서를 추천해주는 AI 검색이 있습니다.
</br> </br>
**담당 역할**: **검색 도메인**을 담당하였습니다
- 하이브리드 검색(BM25 + Vector) 검색 구현
- AI 검색 파이프라인(Embedding + Rerank + LLM) 설계 및 안정화
- RabbitMQ 기반 upsert/delete + retry\DLQ 운영 설계 및 구현
- Redis 캐시 (검색 캐시 / 할인 정책 캐시) 전략 수립

---
### 검색 관련 핵심 기능
1) 검색
   - Elasticsearch 기반 키워드 검색(BM25)
   - 임베딩이 유효한 경우에만 Vector KNN을 결합한 **하이브리드 검색**
   - 일반 검색과 AI 검색을 분리하여 **핵심 경로(빠른 검색)**을 안정적으로 유지
2) AI 검색
   - 파이프라인: Embedding -> ES 후보 생성 -> Top-N Rerank -> Top-M LLM 결과 생성
   - 외부 API 실패/지연을 고려해 **단계별 fallback**으로 결과가 비지 않게 설계
3) Worker
   - MySQL 변경을 RabbitMQ 이벤트로 받아 ElasticSearch 문서를 **upsert/delete**로 동기화
   - Manual Ack + Retry (최대 3회) + DLQ 격리로 운영 리스크 감소
4) Cache / Consistency
   - (AI 검색) Komoran 기반 **의미 중심 캐시 키** 생성: 키워드 추출 + 정렬(가나다 순) + Redis 저장
   - (할인정책) 5분 주기 갱신 + TTL 10분, 5분 사이 비활성화 된 정책은 삭제 반영
  
---

### Link
- 프로젝트 링크: [https://github.com/nhnacademy-be12-daiso](https://github.com/nhnacademy-be12-daiso)
- Wiki (Decision Log) : [https://github.com/nhnacademy-be12-daiso/booksearch-worker/wiki/Decision-Log](https://github.com/nhnacademy-be12-daiso/booksearch-worker/wiki/Decision-Log)
  - 검색 성능/안정성 개선 과정(트레이드오프, 적용 이유, 결과)을 기록
 

---
## Contact
- Phone: 010 - 6483 - 0735
- Email: jun03031@gmail.com
- Github: https://github.com/shinjunsik
