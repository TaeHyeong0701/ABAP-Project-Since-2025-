# ⚡ Quick Start Guide - ABAP Excel Upload

## 5분 안에 시작하기!

이 가이드를 따라하면 5분 안에 첫 Excel Upload 프로그램을 만들 수 있습니다.

---

## 📝 Step 1: 템플릿 복사 (30초)

1. `TEMPLATE_EXCEL_UPLOAD` 파일을 열기
2. 내용 전체 복사
3. SE38에서 새 프로그램 생성
   - 프로그램명: `Z_MY_FIRST_EXCEL_UPLOAD`
   - 설명: "나의 첫 Excel Upload"
4. 복사한 내용 붙여넣기

---

## ✏️ Step 2: 기본 정보 수정 (1분)

파일 상단의 주석 부분을 수정하세요:

```abap
*& Report Z_MY_FIRST_EXCEL_UPLOAD
*&---------------------------------------------------------------------*
*& Excel Upload Program Template
*& 작성일: 2025-01-15              ← 오늘 날짜로 변경
*& 작성자: 홍길동                   ← 본인 이름으로 변경
*& 목적: 테스트 데이터 업로드       ← 목적 입력
*& 대상테이블: ZTEST_TABLE         ← 테이블명 입력
*&---------------------------------------------------------------------*
```

---

## 🎯 Step 3: 필드 정의 (2분)

업로드할 데이터 필드를 정의하세요:

### 예시: 제품 정보 업로드

```abap
TYPES: BEGIN OF TY_EXCEL,
         PRODUCT_ID   TYPE CHAR10,    "A열 - 제품코드
         PRODUCT_NAME TYPE CHAR40,    "B열 - 제품명
         PRICE        TYPE P DECIMALS 2,  "C열 - 가격
         QUANTITY     TYPE I,         "D열 - 수량
         
         " 필수 필드 (처리 결과 메시지용)
         MSGTY        TYPE BAPI_MTYPE,
         MSGTX        TYPE BAPI_MSG,
       END OF TY_EXCEL.
```

**중요:** 컬럼 수를 확인하세요! (이 예시는 4개 컬럼)

---

## 🔄 Step 4: Excel 매핑 (1분)

`FORM UPLOAD_EXCEL` 내의 CASE 문을 수정하세요:

```abap
" 1. 컬럼 수 변경
CALL FUNCTION 'ALSM_EXCEL_TO_INTERNAL_TABLE'
  EXPORTING
    FILENAME    = P_FILE
    I_BEGIN_COL = 1
    I_BEGIN_ROW = 3
    I_END_COL   = 4        ← 실제 컬럼 수로 변경!
    I_END_ROW   = 100000
  TABLES
    INTERN      = LT_RAW.

" 2. 컬럼 매핑 추가
LOOP AT LT_RAW INTO LS_RAW.
  CASE LS_RAW-COL.
    WHEN 1. GS_EXCEL-PRODUCT_ID   = LS_RAW-VALUE.
    WHEN 2. GS_EXCEL-PRODUCT_NAME = LS_RAW-VALUE.
    WHEN 3. GS_EXCEL-PRICE        = LS_RAW-VALUE.
    WHEN 4. GS_EXCEL-QUANTITY     = LS_RAW-VALUE.
  ENDCASE.
```

---

## 📊 Step 5: ALV 필드 설정 (30초)

`FORM BUILD_FIELDCAT` 내에 필드를 추가하세요:

```abap
FORM BUILD_FIELDCAT.
  " 제품코드
  CLEAR GS_FCAT.
  GS_FCAT-FIELDNAME = 'PRODUCT_ID'.
  GS_FCAT-SELTEXT_L = '제품코드'.
  GS_FCAT-COL_POS   = 1.
  APPEND GS_FCAT TO GT_FCAT.

  " 제품명
  CLEAR GS_FCAT.
  GS_FCAT-FIELDNAME = 'PRODUCT_NAME'.
  GS_FCAT-SELTEXT_L = '제품명'.
  GS_FCAT-COL_POS   = 2.
  APPEND GS_FCAT TO GT_FCAT.

  " 가격
  CLEAR GS_FCAT.
  GS_FCAT-FIELDNAME = 'PRICE'.
  GS_FCAT-SELTEXT_L = '가격'.
  GS_FCAT-COL_POS   = 3.
  APPEND GS_FCAT TO GT_FCAT.

  " 수량
  CLEAR GS_FCAT.
  GS_FCAT-FIELDNAME = 'QUANTITY'.
  GS_FCAT-SELTEXT_L = '수량'.
  GS_FCAT-COL_POS   = 4.
  APPEND GS_FCAT TO GT_FCAT.

  " 메시지 타입 (자동 추가됨)
  CLEAR GS_FCAT.
  GS_FCAT-FIELDNAME = 'MSGTY'.
  GS_FCAT-SELTEXT_L = '메시지타입'.
  GS_FCAT-COL_POS   = 90.
  APPEND GS_FCAT TO GT_FCAT.

  " 메시지 (자동 추가됨)
  CLEAR GS_FCAT.
  GS_FCAT-FIELDNAME = 'MSGTX'.
  GS_FCAT-SELTEXT_L = '메시지'.
  GS_FCAT-COL_POS   = 91.
  APPEND GS_FCAT TO GT_FCAT.
ENDFORM.
```

---

## 📄 Step 6: Excel 파일 준비 (30초)

Excel 파일을 다음 형식으로 작성하세요:

### 파일 예시: `test_products.xls`

```
제품 정보 업로드 테스트

| 제품코드 | 제품명    | 가격   | 수량 |
|----------|-----------|--------|------|
| P001     | 노트북    | 1500000| 10   |
| P002     | 마우스    | 25000  | 50   |
| P003     | 키보드    | 80000  | 30   |
```

**중요:**
- 1행: 제목 (자유롭게)
- 2행: 컬럼 헤더 (한글)
- 3행~: 실제 데이터

---

## ▶️ Step 7: 프로그램 실행 (30초)

1. SE38에서 프로그램 활성화 (Ctrl+F3)
2. 실행 (F8)
3. 파일 선택 필드에서 F4 클릭
4. `test_products.xls` 파일 선택
5. 실행 (F8)
6. ALV 화면에서 결과 확인!

---

## 🎉 축하합니다!

첫 Excel Upload 프로그램을 완성했습니다!

---

## 🔍 다음 단계

### 초급
- ✅ `FORM VALIDATE_DATA` 수정하여 데이터 검증 추가
- ✅ `FORM PROCESS_DATA` 수정하여 데이터베이스 저장 구현

### 중급
- ✅ **EXAMPLES.md**의 "예제 2: 데이터 검증 포함" 학습
- ✅ 에러 처리 로직 추가
- ✅ 진행률 표시 기능 추가

### 고급
- ✅ **EXAMPLES.md**의 "예제 3: BAPI 사용" 학습
- ✅ **EXAMPLES.md**의 "예제 4: 대용량 데이터 처리" 학습
- ✅ 성능 최적화 적용

---

## 📚 더 자세한 학습

- **README_KR.md** - 프로젝트 전체 개요
- **EXCEL_UPLOAD_GUIDE.md** - 상세한 개발 가이드
- **EXAMPLES.md** - 5가지 실전 예제
- **Z_SIMPLE_EXCEL_UPLOAD** - 참조 예제

---

## 💡 팁

### 빠른 개발을 위한 팁

1. **매크로 활용**
```abap
DEFINE ADD_FIELD.
  CLEAR GS_FCAT.
  GS_FCAT-FIELDNAME = &1.
  GS_FCAT-SELTEXT_L = &2.
  GS_FCAT-COL_POS   = &3.
  APPEND GS_FCAT TO GT_FCAT.
END-OF-DEFINITION.

" 사용
ADD_FIELD 'PRODUCT_ID' '제품코드' 1.
ADD_FIELD 'PRODUCT_NAME' '제품명' 2.
```

2. **주석 활용**
```abap
" TODO: 나중에 구현
" FIXME: 버그 수정 필요
" NOTE: 중요한 메모
```

3. **디버깅**
```abap
" 디버깅 포인트 설정: /h 명령어 또는 BREAK-POINT
BREAK-POINT.
```

---

## ❓ 문제 발생시

### 1. 컴파일 에러
- 필드명 오타 확인
- TYPE 선언 확인
- 변수명 일치 확인

### 2. Excel 읽기 실패
- 파일 경로 확인
- 파일 형식 확인 (.xls 권장)
- 파일이 열려있지 않은지 확인

### 3. ALV 표시 안됨
- GT_EXCEL에 데이터가 있는지 확인
- GT_FCAT이 올바르게 구성되었는지 확인

더 자세한 문제 해결은 **EXCEL_UPLOAD_GUIDE.md**의 "문제해결" 섹션을 참조하세요.

---

## 🎯 체크리스트

개발 완료 전 확인하세요:

- [ ] 프로그램명이 Z_로 시작하는가?
- [ ] TY_EXCEL에 MSGTY, MSGTX가 포함되어 있는가?
- [ ] I_END_COL이 실제 컬럼 수와 일치하는가?
- [ ] 모든 컬럼이 CASE 문에 매핑되어 있는가?
- [ ] Field Catalog에 모든 필드가 포함되어 있는가?
- [ ] Excel 파일 형식이 올바른가? (1행:제목, 2행:헤더, 3행~:데이터)
- [ ] 프로그램이 정상적으로 활성화되는가?
- [ ] 테스트 실행이 성공하는가?

---

**Good Luck! 🚀**
