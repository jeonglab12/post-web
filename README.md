```mermaid
graph TD;
    subgraph "고객 접점"
        A[HTS/MTS/Web] -- 주문/조회 --> B{API Gateway}
    end

    subgraph "증권사 내부 시스템"
        B -- 주문 --> C[주문 관리 시스템 (OMS)]
        B -- 조회 --> G[계좌 관리 시스템]

        C -- 주문 라우팅 --> D{거래소 라우터}
        D -- 주문 --> E1[한국거래소 (KRX)]
        D -- 주문 --> E2[ATS (대체거래소)]

        E1 -- 체결/시세 --> F1[KRX 체결/시세 처리]
        E2 -- 체결/시세 --> F2[ATS 체결/시세 처리]

        F1 -- 체결/시세 통합 --> H[체결 관리 시스템 (EMS)]
        F2 -- 체결/시세 통합 --> H

        H -- 잔고/평가 업데이트 --> G[계좌 관리 시스템]
        H -- 시장감시 데이터 --> I[시장감시/리스크 관리]

        G -- 잔고/평가 조회 --> J[고객 정보 DB]
        H -- 체결 내역 저장 --> J

        I -- 이상 거래 감지 --> K[알림/보고 시스템]
    end

    subgraph "외부 시스템"
        E1 -- KRX 통신 --> L1[KRX 시스템]
        E2 -- ATS 통신 --> L2[ATS 시스템]
    end

    subgraph "공통 인프라"
        C -- DB 저장 --> DB[주문/체결 DB]
        H -- DB 저장 --> DB
        G -- DB 저장 --> DB
        DB -- DB 조회 --> G
        DB -- DB 조회 --> H
        DB -- DB 조회 --> I

        C -- 로깅 --> M[통합 로깅 시스템]
        H -- 로깅 --> M
        G -- 로깅 --> M
        I -- 로깅 --> M

        N[모니터링 시스템] -- 시스템 상태 --> M
        N -- 경고 --> K

    style C fill:#f9f,stroke:#333,stroke-width:2px;
    style D fill:#f9f,stroke:#333,stroke-width:2px;
    style H fill:#f9f,stroke:#333,stroke-width:2px;
    style G fill:#f9f,stroke:#333,stroke-width:2px;
    style DB fill:#f9f,stroke:#333,stroke-width:2px;

    classDef highlight fill:#ffc,stroke:#f00,stroke-width:2px;
    class C,D,H,G,DB highlight;

    note right of C: **장애 발생 가능 지점: 주문 라우팅 지연/실패, OMS 과부하**
    note right of D: **장애 발생 가능 지점: 잘못된 거래소 라우팅, 라우팅 로직 오류**
    note right of H: **장애 발생 가능 지점: 체결 정보 통합 지연/오류, 동기화 문제**
    note right of G: **장애 발생 가능 지점: 잔고 불일치, 평가금액 오류**
    note right of DB: **장애 발생 가능 지점: DB 부하, Lock, 데이터 정합성 문제**

