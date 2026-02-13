# ABAP Excel Upload 프로그램 개발 가이드

## 목차
1. [시작하기](#시작하기)
2. [템플릿 사용법](#템플릿-사용법)
3. [단계별 개발 가이드](#단계별-개발-가이드)
4. [주요 함수 설명](#주요-함수-설명)
5. [Best Practices](#best-practices)
6. [문제해결](#문제해결)

---

## 시작하기

### 사전 요구사항
- SAP 시스템 접근 권한
- SE38 또는 SE80 트랜잭션 접근 권한
- 대상 테이블에 대한 읽기/쓰기 권한

### 파일 구조
```
ABAP-Project-Since-2025-/
├── README.md                    # 프로젝트 개요
├── EXCEL_UPLOAD_GUIDE.md       # 이 파일
├── TEMPLATE_EXCEL_UPLOAD       # 템플릿 프로그램
└── Z_SIMPLE_EXCEL_UPLOAD       # 참조 예제
```

---

## 템플릿 사용법

### 1. 템플릿 복사
1. `TEMPLATE_EXCEL_UPLOAD` 파일을 새 프로그램으로 복사
2. 프로그램명을 `Z_[모듈명]_EXCEL_UPLOAD`로 변경

### 2. TODO 항목 수정
템플릿에서 `[대괄호]`로 표시된 항목들을 실제 값으로 변경:

```abap
*& Report Z_[PROGRAM_NAME]_EXCEL_UPLOAD
*& 작성일: [YYYY-MM-DD]
*& 작성자: [작성자명]
*& 목적: [프로그램 목적 설명]
*& 대상테이블: [테이블명]
```

### 3. 필드 정의
`TY_EXCEL` 구조체에 필요한 필드 추가:

```abap
TYPES: BEGIN OF TY_EXCEL,
         FIELD1 TYPE MATNR,     "A열 - 자재코드
         FIELD2 TYPE MTART,     "B열 - 자재유형
         FIELD3 TYPE MAKTX,     "C열 - 자재내역
         " ... 추가 필드
         MSGTY  TYPE BAPI_MTYPE,
         MSGTX  TYPE BAPI_MSG,
       END OF TY_EXCEL.
```

---

## 단계별 개발 가이드

### Step 1: 요구사항 분석
- [ ] 업로드할 데이터 항목 확인
- [ ] 대상 테이블 구조 파악
- [ ] Excel 파일 형식 정의
- [ ] 필수 검증 규칙 확인

### Step 2: TYPE 정의
```abap
TYPES: BEGIN OF TY_EXCEL,
         " 비즈니스 필드 (Excel 컬럼과 매핑)
         MATNR      TYPE MATNR,      "자재코드
         MTART      TYPE MTART,      "자재유형
         MAKTX      TYPE MAKTX,      "자재내역
         
         " 필수 시스템 필드
         MSGTY      TYPE BAPI_MTYPE, "메시지 타입
         MSGTX      TYPE BAPI_MSG,   "메시지 텍스트
       END OF TY_EXCEL.
```

**주의사항:**
- 필드명은 가능한 표준 데이터 요소 사용
- 주석으로 컬럼 위치와 설명 명시
- MSGTY, MSGTX는 필수 포함

### Step 3: Excel 데이터 매핑
```abap
LOOP AT LT_RAW INTO LS_RAW.
  CASE LS_RAW-COL.
    WHEN 1. GS_EXCEL-MATNR = LS_RAW-VALUE.
    WHEN 2. GS_EXCEL-MTART = LS_RAW-VALUE.
    WHEN 3. GS_EXCEL-MAKTX = LS_RAW-VALUE.
    " ... 추가 컬럼 매핑
  ENDCASE.
  
  AT END OF ROW.
    PERFORM VALIDATE_DATA.
    APPEND GS_EXCEL TO GT_EXCEL.
    CLEAR GS_EXCEL.
  ENDAT.
ENDLOOP.
```

**매핑 규칙:**
- Excel 컬럼 순서에 맞춰 WHEN 절 작성
- 숫자 필드는 형변환 필요시 CONVERSION 사용
- 날짜 필드는 형식 변환 주의

### Step 4: 데이터 검증
```abap
FORM VALIDATE_DATA.
  " 필수 필드 검증
  IF GS_EXCEL-MATNR IS INITIAL.
    GS_EXCEL-MSGTY = 'E'.
    GS_EXCEL-MSGTX = '자재코드는 필수입력 항목입니다.'.
    RETURN.
  ENDIF.

  " 형식 검증
  IF NOT GS_EXCEL-MATNR CO '0123456789'.
    GS_EXCEL-MSGTY = 'E'.
    GS_EXCEL-MSGTX = '자재코드는 숫자만 입력 가능합니다.'.
    RETURN.
  ENDIF.

  " 데이터베이스 검증
  SELECT SINGLE MATNR FROM MARA
    INTO @DATA(LV_MATNR)
    WHERE MATNR = @GS_EXCEL-MATNR.
  
  IF SY-SUBRC <> 0.
    GS_EXCEL-MSGTY = 'W'.
    GS_EXCEL-MSGTX = '존재하지 않는 자재코드입니다.'.
    RETURN.
  ENDIF.

  " 검증 통과
  GS_EXCEL-MSGTY = 'S'.
  GS_EXCEL-MSGTX = '검증 완료'.
ENDFORM.
```

**검증 레벨:**
- **E (Error)**: 치명적 오류, 처리 불가
- **W (Warning)**: 경고, 처리 가능하나 확인 필요
- **S (Success)**: 정상
- **I (Information)**: 정보성 메시지

### Step 5: 데이터 처리

#### 방법 1: 직접 테이블 업데이트
```abap
FORM PROCESS_DATA.
  DATA: LT_TARGET TYPE TABLE OF ZMDAT9012,
        LS_TARGET LIKE LINE OF LT_TARGET.

  " 에러가 없는 데이터만 처리
  LOOP AT GT_EXCEL INTO GS_EXCEL WHERE MSGTY <> 'E'.
    MOVE-CORRESPONDING GS_EXCEL TO LS_TARGET.
    APPEND LS_TARGET TO LT_TARGET.
  ENDLOOP.

  " 데이터베이스 업데이트
  IF LT_TARGET IS NOT INITIAL.
    MODIFY ZMDAT9012 FROM TABLE LT_TARGET.
    
    IF SY-SUBRC = 0.
      COMMIT WORK.
      MESSAGE |{ LINES( LT_TARGET ) }건이 처리되었습니다.| TYPE 'S'.
    ELSE.
      ROLLBACK WORK.
      MESSAGE '데이터 저장 중 오류가 발생했습니다.' TYPE 'E'.
    ENDIF.
  ENDIF.
ENDFORM.
```

#### 방법 2: BAPI 사용
```abap
FORM PROCESS_DATA.
  DATA: LS_RETURN TYPE BAPIRET2.

  LOOP AT GT_EXCEL INTO GS_EXCEL WHERE MSGTY <> 'E'.
    " BAPI 호출
    CALL FUNCTION 'BAPI_MATERIAL_SAVEDATA'
      EXPORTING
        HEADDATA = ...
      IMPORTING
        RETURN   = LS_RETURN.

    " 결과 처리
    IF LS_RETURN-TYPE = 'E' OR LS_RETURN-TYPE = 'A'.
      GS_EXCEL-MSGTY = LS_RETURN-TYPE.
      GS_EXCEL-MSGTX = LS_RETURN-MESSAGE.
    ELSE.
      GS_EXCEL-MSGTY = 'S'.
      GS_EXCEL-MSGTX = '처리 성공'.
      
      " 변경사항 확정
      CALL FUNCTION 'BAPI_TRANSACTION_COMMIT'
        EXPORTING
          WAIT = 'X'.
    ENDIF.

    MODIFY GT_EXCEL FROM GS_EXCEL.
  ENDLOOP.
ENDFORM.
```

### Step 6: ALV 출력
```abap
FORM BUILD_FIELDCAT.
  " 매크로를 사용한 간편한 Field Catalog 구성
  DEFINE ADD_FIELD.
    CLEAR GS_FCAT.
    GS_FCAT-FIELDNAME = &1.
    GS_FCAT-SELTEXT_L = &2.
    GS_FCAT-COL_POS   = &3.
    APPEND GS_FCAT TO GT_FCAT.
  END-OF-DEFINITION.

  ADD_FIELD 'MATNR' '자재코드' 1.
  ADD_FIELD 'MTART' '자재유형' 2.
  ADD_FIELD 'MAKTX' '자재내역' 3.
  ADD_FIELD 'MSGTY' '메시지타입' 90.
  ADD_FIELD 'MSGTX' '메시지' 91.
ENDFORM.
```

---

## 주요 함수 설명

### 1. ALSM_EXCEL_TO_INTERNAL_TABLE
Excel 파일을 내부 테이블로 변환

**Parameters:**
- `FILENAME`: Excel 파일 경로
- `I_BEGIN_COL`: 시작 컬럼 (1부터)
- `I_BEGIN_ROW`: 시작 행 (보통 3, 헤더 제외)
- `I_END_COL`: 종료 컬럼
- `I_END_ROW`: 종료 행 (100000 권장)
- `INTERN`: 결과를 받을 내부 테이블

**반환 구조 (ALSMEX_TABLINE):**
- `ROW`: 행 번호
- `COL`: 컬럼 번호
- `VALUE`: 셀 값

### 2. F4_FILENAME
파일 선택 대화상자

**Parameters:**
- `FILE_NAME`: 선택된 파일 경로 (반환)

### 3. REUSE_ALV_GRID_DISPLAY
ALV Grid 출력

**Parameters:**
- `I_CALLBACK_PROGRAM`: 현재 프로그램명 (SY-REPID)
- `IS_LAYOUT`: Layout 구조체
- `IT_FIELDCAT`: Field Catalog 테이블
- `T_OUTTAB`: 출력할 데이터 테이블

---

## Best Practices

### 1. 성능 최적화
```abap
" ❌ 나쁜 예: 반복문에서 SELECT
LOOP AT GT_EXCEL INTO GS_EXCEL.
  SELECT SINGLE * FROM MARA WHERE MATNR = GS_EXCEL-MATNR.
  ...
ENDLOOP.

" ✅ 좋은 예: 한번에 SELECT
SELECT MATNR, MTART
  FROM MARA
  INTO TABLE @DATA(LT_MARA)
  FOR ALL ENTRIES IN @GT_EXCEL
  WHERE MATNR = @GT_EXCEL-MATNR.
```

### 2. 에러 처리
```abap
" 항상 SY-SUBRC 체크
CALL FUNCTION 'SOME_FUNCTION'
  EXCEPTIONS
    ERROR = 1
    OTHERS = 2.

IF SY-SUBRC <> 0.
  " 에러 처리
  MESSAGE ID SY-MSGID TYPE SY-MSGTY NUMBER SY-MSGNO
    WITH SY-MSGV1 SY-MSGV2 SY-MSGV3 SY-MSGV4.
ENDIF.
```

### 3. 트랜잭션 관리
```abap
" 데이터 일관성 유지
MODIFY ZTABLE FROM TABLE LT_DATA.

IF SY-SUBRC = 0.
  COMMIT WORK AND WAIT.
ELSE.
  ROLLBACK WORK.
ENDIF.
```

### 4. 로깅
```abap
" 처리 로그 기록
DATA: LV_CNT_SUCCESS TYPE I,
      LV_CNT_ERROR   TYPE I.

LOOP AT GT_EXCEL INTO GS_EXCEL.
  IF GS_EXCEL-MSGTY = 'S'.
    ADD 1 TO LV_CNT_SUCCESS.
  ELSEIF GS_EXCEL-MSGTY = 'E'.
    ADD 1 TO LV_CNT_ERROR.
  ENDIF.
ENDLOOP.

MESSAGE |성공: { LV_CNT_SUCCESS }건, 실패: { LV_CNT_ERROR }건| TYPE 'S'.
```

---

## 문제해결

### Q1: Excel 파일을 읽을 수 없습니다
**증상:** ALSM_EXCEL_TO_INTERNAL_TABLE 실행 시 에러

**해결방법:**
- 파일 경로 확인 (절대 경로 사용)
- 파일 접근 권한 확인
- Excel 파일이 열려있는지 확인 (닫고 재실행)
- `.xls` 형식 사용 권장 (`.xlsx`는 문제 발생 가능)

### Q2: 한글이 깨집니다
**증상:** Excel에서 한글 데이터가 깨져서 표시

**해결방법:**
```abap
" 인코딩 변환
DATA: LV_TEXT TYPE STRING.

LV_TEXT = LS_RAW-VALUE.
CALL FUNCTION 'SCP_REPLACE_STRANGE_CHARS'
  CHANGING
    INTEXT = LV_TEXT.

GS_EXCEL-FIELD = LV_TEXT.
```

### Q3: 날짜 형식이 맞지 않습니다
**증상:** Excel의 날짜가 제대로 변환되지 않음

**해결방법:**
```abap
" Excel 날짜를 ABAP 날짜로 변환
DATA: LV_DATE_EXTERNAL TYPE CHAR10,
      LV_DATE_INTERNAL TYPE DATUM.

LV_DATE_EXTERNAL = LS_RAW-VALUE.  " '2025-01-15' 형식

CALL FUNCTION 'CONVERT_DATE_TO_INTERNAL'
  EXPORTING
    DATE_EXTERNAL = LV_DATE_EXTERNAL
  IMPORTING
    DATE_INTERNAL = LV_DATE_INTERNAL.

GS_EXCEL-DATE = LV_DATE_INTERNAL.
```

### Q4: 대용량 데이터 처리가 느립니다
**증상:** 수천 건 이상 처리시 성능 저하

**해결방법:**
```abap
" 배치 단위로 처리
DATA: LV_BATCH_SIZE TYPE I VALUE 1000.

LOOP AT GT_EXCEL INTO GS_EXCEL.
  APPEND GS_EXCEL TO LT_BATCH.
  
  IF LINES( LT_BATCH ) >= LV_BATCH_SIZE.
    PERFORM PROCESS_BATCH USING LT_BATCH.
    CLEAR LT_BATCH.
  ENDIF.
ENDLOOP.

" 남은 데이터 처리
IF LT_BATCH IS NOT INITIAL.
  PERFORM PROCESS_BATCH USING LT_BATCH.
ENDIF.
```

### Q5: ALV가 표시되지 않습니다
**증상:** REUSE_ALV_GRID_DISPLAY 호출 후 아무것도 표시되지 않음

**해결방법:**
- GT_EXCEL에 데이터가 있는지 확인
- GT_FCAT이 올바르게 구성되었는지 확인
- SY-SUBRC 체크로 에러 확인

```abap
IF GT_EXCEL IS INITIAL.
  MESSAGE '표시할 데이터가 없습니다.' TYPE 'S' DISPLAY LIKE 'W'.
  RETURN.
ENDIF.

" ALV 호출 전 데이터 확인
DESCRIBE TABLE GT_EXCEL LINES DATA(LV_LINES).
MESSAGE |{ LV_LINES }건의 데이터를 표시합니다.| TYPE 'S'.
```

---

## Excel 파일 준비 가이드

### 표준 Excel 템플릿 형식

```
행1: [프로그램 제목 또는 설명]
행2: [컬럼 헤더]
행3~: [실제 데이터]
```

**예시:**
```
자재 마스터 데이터 업로드 템플릿

| 자재코드 | 자재유형 | 자재내역 | 자재그룹 | 기본단위 |
|----------|----------|----------|----------|----------|
| 100001   | ROH      | 원자재A  | 0001     | EA       |
| 100002   | HALB     | 반제품B  | 0002     | KG       |
```

### 주의사항
- 셀에 수식 사용 금지 (값만 입력)
- 병합된 셀 사용 금지
- 빈 행 금지 (데이터 중간에 빈 행 있으면 안됨)
- 특수문자 주의 (탭, 개행 등)

---

## 참고 자료

### 관련 트랜잭션
- **SE38**: ABAP 에디터
- **SE80**: Object Navigator
- **SE11**: ABAP Dictionary
- **SE16**: Data Browser

### 유용한 함수
- `TEXT_CONVERT_XLS_TO_SAP`: Excel 변환 (대안)
- `GUI_UPLOAD`: 파일 업로드
- `CONVERSION_EXIT_*`: 형식 변환
- `BAPI_TRANSACTION_COMMIT`: 변경사항 확정
- `BAPI_TRANSACTION_ROLLBACK`: 변경사항 취소

### 추가 학습 자료
- SAP Help Portal: ALV Grid Control
- SAP Help Portal: BAPI Programming
- SCN (SAP Community Network): Excel Upload

---

## 변경 이력

| 날짜 | 작성자 | 내용 |
|------|--------|------|
| 2025-01 | - | 초기 가이드 작성 |

---

## 라이선스

이 프로젝트는 학습 및 참고 목적으로 제공됩니다.
