# ABAP Project - Excel Upload Programs

## 개요
이 Repository는 ABAP Excel Upload 프로그램의 표준 템플릿과 예제를 제공합니다.

## Excel Upload 프로그램 표준 형식

### 기본 구조
모든 Excel upload 프로그램은 Z_SIMPLE_EXCEL_UPLOAD의 형식을 따릅니다.

### 표준 템플릿 구조

```abap
*&---------------------------------------------------------------------*
*& Report Z_[PROGRAM_NAME]
*&---------------------------------------------------------------------*
*& Excel Upload Program
*& 작성일: [DATE]
*& 작성자: [AUTHOR]
*& 목적: [PURPOSE]
*&---------------------------------------------------------------------*
REPORT Z_[PROGRAM_NAME].

TYPE-POOLS: TRUXS, SLIS.

"=========================
" TYPE 정의
"=========================
TYPES: BEGIN OF TY_EXCEL,
         [필드 정의]
         MSGTY TYPE BAPI_MTYPE,
         MSGTX TYPE BAPI_MSG,
       END OF TY_EXCEL.

DATA: GT_EXCEL TYPE STANDARD TABLE OF TY_EXCEL,
      GS_EXCEL TYPE TY_EXCEL.

" ALV 관련 변수
DATA: GR_ALV       TYPE REF TO CL_GUI_ALV_GRID,
      GR_CONTAINER TYPE REF TO CL_GUI_CUSTOM_CONTAINER.

DATA: GT_FCAT   TYPE LVC_T_FCAT,
      GS_FCAT   TYPE LVC_S_FCAT,
      GS_LAYOUT TYPE LVC_S_LAYO.

PARAMETERS: P_FILE TYPE RLGRAP-FILENAME OBLIGATORY.

"=========================
" SELECTION SCREEN
"=========================
AT SELECTION-SCREEN ON VALUE-REQUEST FOR P_FILE.
  CALL FUNCTION 'F4_FILENAME'
    IMPORTING
      FILE_NAME = P_FILE.

"=========================
" MAIN LOGIC
"=========================
START-OF-SELECTION.
  PERFORM UPLOAD_EXCEL.

"=========================
" EXCEL 업로드 처리
"=========================
FORM UPLOAD_EXCEL.
  DATA: LT_RAW TYPE TABLE OF ALSMEX_TABLINE,
        LS_RAW TYPE ALSMEX_TABLINE.

  CALL FUNCTION 'ALSM_EXCEL_TO_INTERNAL_TABLE'
    EXPORTING
      FILENAME    = P_FILE
      I_BEGIN_COL = 1
      I_BEGIN_ROW = 3
      I_END_COL   = [컬럼수]
      I_END_ROW   = 100000
    TABLES
      INTERN      = LT_RAW.

  LOOP AT LT_RAW INTO LS_RAW.
    CASE LS_RAW-COL.
      WHEN 1. GS_EXCEL-[필드1] = LS_RAW-VALUE.
      WHEN 2. GS_EXCEL-[필드2] = LS_RAW-VALUE.
      " ... 추가 필드 매핑
    ENDCASE.
    AT END OF ROW.
      APPEND GS_EXCEL TO GT_EXCEL.
      CLEAR GS_EXCEL.
    ENDAT.
  ENDLOOP.

  " 데이터 처리 로직
  PERFORM PROCESS_DATA.
ENDFORM.

"=========================
" 데이터 처리
"=========================
FORM PROCESS_DATA.
  " 비즈니스 로직 구현
ENDFORM.
```

## 주요 구성 요소

### 1. TYPE-POOLS
```abap
TYPE-POOLS: TRUXS, SLIS.
```
- **TRUXS**: Excel 변환 관련 타입
- **SLIS**: ALV 리스트 출력 관련 타입

### 2. TYPE 정의
```abap
TYPES: BEGIN OF TY_EXCEL,
         " 비즈니스 필드들
         MSGTY TYPE BAPI_MTYPE,  " 메시지 타입
         MSGTX TYPE BAPI_MSG,    " 메시지 텍스트
       END OF TY_EXCEL.
```
- Excel 데이터 구조 정의
- 항상 MSGTY, MSGTX 필드 포함 (처리 결과 메시지용)

### 3. 파일 선택 파라미터
```abap
PARAMETERS: P_FILE TYPE RLGRAP-FILENAME OBLIGATORY.
```

### 4. F4 도움말
```abap
AT SELECTION-SCREEN ON VALUE-REQUEST FOR P_FILE.
  CALL FUNCTION 'F4_FILENAME'
    IMPORTING
      FILE_NAME = P_FILE.
```

### 5. Excel 업로드 함수
```abap
CALL FUNCTION 'ALSM_EXCEL_TO_INTERNAL_TABLE'
  EXPORTING
    FILENAME    = P_FILE
    I_BEGIN_COL = 1
    I_BEGIN_ROW = 3      " 보통 3행부터 시작 (헤더 제외)
    I_END_COL   = [컬럼수]
    I_END_ROW   = 100000
  TABLES
    INTERN      = LT_RAW.
```

## Excel 파일 형식 가이드

### Excel 파일 레이아웃
- **1행**: 제목
- **2행**: 컬럼 헤더
- **3행~**: 실제 데이터

### 예제 (Z_SIMPLE_EXCEL_UPLOAD)

| A | B | C | D | E | F | G | H | I |
|---|---|---|---|---|---|---|---|---|
| **자재코드** | **자재유형** | **자재내역** | **자재그룹** | **기본단위** | **플랜트** | **MRP 컨트롤러** | **로트 키** | **고정로트수량** |
| MATNR | MTART | MAKTX | MATKL | MEINS | WERKS | MRP_CTRLER | LOTSIZEKEY | FIXED_LOT |

## 프로그램 목록

### Z_SIMPLE_EXCEL_UPLOAD
- **목적**: 자재 마스터 데이터 Excel 업로드
- **대상 테이블**: ZMDAT9012
- **컬럼 수**: 9개
- **주요 필드**: 자재코드, 자재유형, 자재내역, 자재그룹, 기본단위, 플랜트, MRP 컨트롤러, 로트 키, 고정로트수량

## 개발 가이드라인

### 명명 규칙
- 프로그램명: `Z_[모듈]_EXCEL_UPLOAD`
- 타입명: `TY_EXCEL`
- 전역변수: `GT_` (Global Table), `GS_` (Global Structure)
- 로컬변수: `LT_` (Local Table), `LS_` (Local Structure)

### 코딩 표준
1. 섹션별 주석으로 코드 구분
2. 들여쓰기 2칸 사용
3. 한글 주석 권장 (필드 설명)
4. FORM 서브루틴 활용

### 에러 처리
1. MSGTY, MSGTX 필드로 처리 결과 추적
2. ALV에서 결과 확인 가능하도록 구현
3. 필수 입력값 검증

## 사용 방법

1. 프로그램 실행 (SE38 또는 트랜잭션 코드)
2. Excel 파일 선택 (F4 도움말 사용 가능)
3. 실행 (F8)
4. 결과 확인

## 참고사항

- Excel 파일은 `.xls` 또는 `.xlsx` 형식 지원
- 최대 100,000행까지 처리 가능
- 대용량 데이터는 분할 업로드 권장
- 업로드 전 데이터 백업 필수

## 버전 이력

- 2025-01: 초기 템플릿 생성 (Z_SIMPLE_EXCEL_UPLOAD)

## 문의

프로젝트 관련 문의사항은 Issue를 통해 등록해주세요.
