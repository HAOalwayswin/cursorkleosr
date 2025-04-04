# Project Configuration (LTM)

*This file contains the stable, long-term context for the project.*
*It should be updated infrequently, primarily when core goals, tech, or patterns change.*

---

## Core Goal

Create a full-stack web application to fetch, store, and visualize real-time commercial district (상권) trend data from the Seoul Open Data API.  
The dashboard will support searching by region and time period and display key metrics such as store count, average monthly sales, and floating population.

The goal is to provide urban commercial insight for public policy makers and business consultants.

---

## Tech Stack

- **Frontend**: React, CSS, Axios
- **Backend**: Django, Django REST Framework (DRF)
- **Database**: PostgreSQL
- **Task Queue / Scheduling**: Celery with Redis
- **Testing**: Pytest, React Testing Library
- **Deployment**: Docker, Nginx, Gunicorn

---

## Critical Patterns & Conventions

- **API Integration**:  
  - Source: Seoul Open API  
  - Dataset: [도시데이터 - 상권변화지표 (citydata_cmrcl)](https://data.seoul.go.kr/dataList/OA-22385/F/1/datasetView.do)  
  - API Key: `77576c4366676b77313033484a544654`
  - Sample Endpoint (encoded for "광화문·덕수궁" 상권):
    ```
    http://openapi.seoul.go.kr:8088/77576c4366676b77313033484a544654/json/citydata_cmrcl/1/5/%EA%B4%91%ED%99%94%EB%AC%B8%C2%B7%EB%8D%95%EC%88%98%EA%B6%81
    ```

- **ETL Flow**:  
  - Fetch external data (via Celery task or Django management command)
  - Parse and clean the data
  - Store in PostgreSQL (with timestamps for historical tracking)
  - Serve via DRF API (`/api/v1/commercial-areas`)
  - Render in React dashboard with filters (by region, year-month, category)

- **Data Visualization**:  
  - Use charts (e.g., bar, line) for time-based trends
  - Interactive filters (region selector, date range picker)
  - KPI tiles (총 점포수, 평균 매출, 유동 인구 등)

- **Component Design**:  
  - Atomic Design Pattern  
  - Use reusable data cards and charts  

---

## Key Constraints

- API rate limit must be respected  
- Real-time means data updated daily (or as frequently as possible)
- Models must be designed for efficient filtering and aggregation
- Must allow future integration with policy dashboards or ESG consulting tools

---

## 📡 External API Specification

### 🔗 API Endpoint (샘플)

GET http://openapi.seoul.go.kr:8088/77576c4366676b77313033484a544654/json/citydata_cmrcl/1/5/광화문·덕수궁

pgsql
복사
편집

> Note: Replace "광화문·덕수궁" with any commercial area (상권명), URL-encoded.

### 📥 Query Parameters

| Parameter | Type   | Description               |
|----------|--------|---------------------------|
| KEY      | string | API key                   |
| TYPE     | string | Response format (`json`)  |
| SERVICE  | string | `citydata_cmrcl`          |
| START_INDEX | int | Starting index of records |
| END_INDEX | int   | Ending index of records   |
| AREA_NM  | string | Commercial area name (URL-encoded) |

### 📤 Sample Response (Parsed)

```json
{
  "citydata_cmrcl": {
    "list_total_count": 1,
    "RESULT": {
      "CODE": "INFO-000",
      "MESSAGE": "정상 처리되었습니다"
    },
    "row": [
      {
        "AREA_NM": "광화문·덕수궁",
        "AREA_CD": "001",
        "AREA_GU": "종로구",
        "AREA_DONG": "세종로",
        "AREA_YN": "Y",
        "ADSTRD_CODE_SE": "1111010100",
        "REFINE_WGS84_LAT": "37.57164",
        "REFINE_WGS84_LOGT": "126.9769",
        "POP_CNT": 2435,
        "RES_CNT": 512,
        "WORK_POP_CNT": 1421,
        "VISIT_POP_CNT": 502,
        "SALES_CNT": 292,
        "AVG_SALE": 4531250,
        "CONGEST_LVL": "보통",
        "CONGEST_MSG": "약간 붐빔",
        "CONGEST_STTS": "2",
        "STD_YM": "202402"
      }
    ]
  }
}

📊 주요 필드 설명
필드명	설명
AREA_NM	상권명
AREA_GU	자치구
STD_YM	기준년월 (예: 202402)
POP_CNT	총 유동인구 수
RES_CNT	거주 인구 수
WORK_POP_CNT	직장 인구 수
VISIT_POP_CNT	방문 인구 수
SALES_CNT	점포 수
AVG_SALE	평균 매출 (원)
CONGEST_MSG	혼잡도 메시지

✅ Data Integration Checklist
 Connect to sample API endpoint

 Parse response fields (JSON)

 Normalize and store in PostgreSQL (via models.py)

 Create DRF endpoint: GET /api/v1/commercial-areas/

 Frontend: Build components to display population, sales, congestion, etc.

 Schedule periodic data updates with Celery